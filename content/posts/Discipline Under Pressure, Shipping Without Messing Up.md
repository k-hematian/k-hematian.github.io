---
title: "Discipline Under Pressure: Shipping Without Messing Up"
date: 2025-08-20
isStarred: true
tags:
  - leadership
  - software engineering
  - technical debt
  - startup
---
## The Reality of Urgency in Startups
It’s crucial for every senior software engineer, technical leader, or anyone accountable for delivering technical solutions to know how to handle urgent tasks wisely. Let’s call this person a *leader* — and today, that’s you.

In some industries, or at certain stages of a company’s lifecycle, urgent requirements hit technical teams constantly. I currently work as a technical lead at a startup ([Done](https://www.linkedin.com/company/donetech/)), where the rules of the game seem to change every other week.

Here, *urgent* really does mean urgent. If the technical team fails to deliver, there might not even be a company left to reach that ideal state of technical excellence.

---
## Trade-Offs Under Pressure
This is where trade-offs become serious. As a leader, you have to resist the urge to ship messy solutions and call it “technical debt.” That path leads to burnout, systems full of cruft, and demotivated engineers.

So, how do you handle urgent requirements wisely? The answer is: make the most of it. Chaos is a ladder.

---
## The Campaign Challenge
Recently at [DONE](https://www.linkedin.com/company/donetech/), we faced a challenge affecting 1M+ TVs using our platform. A new app version needed to reach every device, but our statistics showed many hadn’t updated — leaving them vulnerable.

The business proposed a bold (and expensive) idea: offer free subscriptions between three popular VOD platforms for users who updated their TVs. Great in theory, but brilliant ideas don’t implement themselves.

Product and business teams were in a rush, phones ringing nonstop, demanding speed and “magic” while carrying hidden assumptions. Some examples:

- Free subscriptions? Without a credit card? Instant activation? A paradise for fraudsters — we needed safeguards.
- Massive marketing campaigns and multiple call centers ready? That meant sudden spikes in traffic and load.
- Giving something away for free when it was always paid before? That introduced new concepts and entities into our system.
- An urgent launch? That guaranteed unknowns waiting right after release, with little capacity left for debugging — so reliability was non-negotiable.


---
## Priorities and Strategy
As the technical lead, my first job was to remove noise so the team could focus on engineering, not firefighting.  

We defined four priorities:
1. **No fraud**  
2. **Short time to market**  
3. **Minimal technical debt** (Only *Prudent Technical Debt*[^1])  
4. **Maximum reuse**  

We analyzed solutions — from building a full campaign manager microservice (expensive) to using zero-cost purchases (cheap). After balancing priorities, we concluded:
1. Adding a new microservice during unknowns was risky.  
2. Two new VOD integrations had to be implemented with obsessive reliability — any fault would become an operational black hole draining energy immediately after release.  
3. The “gift” concept needed to exist in the system anyway, deserving a solid, reusable implementation for future OKRs.  
4. Campaign logic was temporary and should be isolated, with accurate eligibility checks and clean code for quick tweaks.

---
## Discipline Over Chaos
You might wonder: how can words like *obsessive* and *solid* fit a solution designed under urgent pressure? As Robert C. Martin (Uncle Bob) reminds us: *“A mess is not technical debt.”* In these situations, discipline matters more than ever.[^2]

We deliberately avoided entering the campaign domain to create a standardized, reusable solution. But when touching volatile or business-critical parts of the system, adding cruft isn’t “technical debt” — it’s simply a mess, and:

> “A mess is *always* a loss.”

---
## Execution and Delivery
We broke the project into components, defined clear contracts, and started development — after walking the product team through all limitations and getting their confirmation.

After four days of ~14-hour shifts, we launched the feature bug-free, with a robust whitelist mechanism. The next day, the business requested a blacklist mechanism. Thanks to our earlier isolation strategy, we delivered it in less than a day. Everything related to the campaign was neatly contained.

---
## Lessons Learned
We avoided the trap of insisting on building a full campaign manager microservice, which could have burned out the team and failed the business. Instead, we:
- Isolated technical debt and documented it for later cleanup.  
- Delivered part of a quarterly initiative along the way.  

The solutions weren’t rocket science — the key was *how* we approached them. Future engineers won’t curse us for:
- A bloated “campaign management” microservice doing everything under the sun.  
- A purchase database littered with meaningless zero-value purchases.  
- A giant, unmanageable feature branch spawning hundreds of hotfixes.  

✅ We avoided all of that.  
⚡ We delivered fast.  
💡 And most importantly, we kept the system healthy for the future.

---
[^1]: [Technical Debt Quadrant from Martin Fowler](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)  
[^2]: [A mess is not a Technical Debt from Uncle Bob](https://blog.cleancoder.com/uncle-bob/2010/12/05/MessIsNotTechnicalDebt.html)