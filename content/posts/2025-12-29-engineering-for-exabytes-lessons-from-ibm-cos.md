---
title: "Engineering for Exabytes: Lessons from IBM COS"
date: 2025-12-29
isStarred: false
description: "How do you achieve 15-nines durability? A deep dive into IBM's Cloud Object Storage architecture: SecureSlice, ZSS, and why math beats brute force replication."
tags:
  - system design
  - storage
  - architecture
  - engineering
  - ibm
---

## Beyond Standard Storage
After [my last post](/posts/2025-12-28-open-source-is-not-enough-the-post-minio-era) on the MinIO shift, I started going down the rabbit hole. We often take object storage for granted, but when you step back and ask, *"How do you actually manage data at an Exabyte scale?"*, the answers get fascinating.

How do you design a system that reaches 15-nines of durability without bankrupting the company on hardware?

I recently found an IBM Redpaper on their Cloud Object Storage (COS) that peels back the layers. It explains the engineering choices required to operate at that magnitude.

---
## The "Secret Sauce": IDA and SecureSlice
![IBM Cloud Object Storage SecureSlice AONT Encryption](/images/ibm-cloud-object-storage-secure-slice-aont-encryption.png)
Most systems treat encryption and durability as separate steps. IBM COS merges them into a single process called **SecureSlice**.

Instead of simple replication, they use an Information Dispersal Algorithm (IDA). But the real genius is how they handle data segments (~4 MB) before they even hit the disk. The process effectively removes the need for a complex external Key Management System (KMS) for every read/write.

Here is the 8-step lifecycle of a data segment:

1.  **Integrity Check:** A checksum is appended to the data to allow tamper detection later.
2.  **Key Gen:** A random encryption key is generated for this specific segment.
3.  **Encryption:** The data (plus checksum) is encrypted with that key.
4.  **Hashing:** A hash of the *encrypted* data is calculated.
5.  **The Magic XOR:** The encryption key is combined with the hash via an exclusive-OR operation: `X = HASH ⊕ KEY`.
6.  **AONT Packaging:** This value `X` is appended to the data, forming an **All-Or-Nothing Transform (AONT)** package.
7.  **Erasure Coding:** The package is encoded using Reed–Solomon, producing *n* fragments where any *k* are needed to reconstruct.
8.  **Dispersal:** These fragments are scattered across different storage nodes and physical locations.

**Why this matters:** because the key is mathematically embedded in the data via the hash, you don't need a central vault to store billions of keys. Partial data exposure reveals nothing because you need *all* relevant slices to reverse the AONT and recover the key.

---
## The Hardware Layer: Bypassing the Filesystem
![PSS and ZSS architecture comparison](/images/pss-and-zss-architecture-comparison.png)
You can't achieve exabyte scale if your software is fighting the OS kernel. IBM realized this and transitioned their storage engine from **PSS** (Packed Slice Storage) to **ZSS** (Zone Slice Storage).

### The Legacy: PSS
PSS was designed to optimize for small objects by grouping them into larger "bin files." This reduced metadata overhead (inode usage) and improved IOPS, but it still relied on the underlying filesystem.

### The Evolution: ZSS
**ZSS (Zone Slice Storage)** is the game changer. It completely bypasses the traditional Linux `ext4` filesystem to talk directly to the raw hardware using `LIBAIO` and `LIBZBC`.

* **SMR Support:** It writes sequentially to large "zones," perfectly aligning with the physics of Shingled Magnetic Recording (SMR) drives.
* **Flash Efficiency:** It minimizes write amplification, significantly extending the life of SSDs.
* **Performance:** By eliminating kernel-space context switching and filesystem metadata lookups, ZSS delivers up to **300% higher throughput** for large objects compared to PSS.

> **The Shift:** PSS treated disks like a generic filesystem; ZSS treats them as raw, zone-aware block devices.

---
## The Economics: Math vs. Brute Force
Finally, let's talk about the cost of reliability.

The industry default for high durability (like HDFS or early MinIO) is **3x Replication**. To store 1 TB of data, you buy 3 TB of disk.
* **Expansion Factor:** 3.0

IBM COS uses IDA (erasure coding). In a standard geo-dispersed setup, the expansion factor is roughly **1.6x**.
* **Expansion Factor:** 1.6

When you are operating at the Exabyte scale, this math defines your business model. To store **1 Exabyte** of usable data:
* **Replication** needs **3 Exabytes** of raw hardware.
* **IBM COS** needs **1.6 Exabytes** of raw hardware.

That is a saving of **1.4 Exabytes** of hard drives, power, cooling, and rack space.

And the kicker? You actually get *higher* durability. In a 3-site setup, 3x replication leaves you vulnerable if one site goes down and a drive fails in another. With IDA, you can lose an entire data center *plus* several nodes in the remaining sites and still experience zero data loss.

---
## Conclusion
Getting into these details helps us understand what is actually happening "behind the curtain." When we understand the trade-offs the data layer makes—like favoring CPU (for IDA math) over raw disk space (replication)—we can design our own software architectures with much more confidence.

For those who want to go deep, the full Redpaper is worth a read.[^1]

---
[^1]: [IBM Cloud Object Storage
Concepts and Architecture](https://www.redbooks.ibm.com/redpieces/pdfs/redp5537.pdf)