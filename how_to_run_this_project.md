
# How to Run This Project

This is a Hugo-based blog project that syncs content from an Obsidian vault. Here's how to run it:

## Prerequisites

Make sure you have the following installed:
- [Hugo](https://gohugo.io/installation/)
- Git
- Python 3
- rsync

## Running the Project

### 1. Sync Content from Obsidian (Optional)

If you want to sync content from your Obsidian vault:

```bash
./sync.sh
```

This script will:
- Copy posts from your Obsidian vault to the Hugo content folder
- Process image links in Markdown files
- Copy images to the appropriate location

### 2. Run the Hugo Server for Local Development

To start the local development server with live reload:

```bash
hugo server
```

This will start a local server, typically at http://localhost:1313/

### 3. Build for Production

To build the site for production:

```bash
hugo
```

This will generate the static site in the `public` directory, which you can then deploy to your web server.

## Additional Notes

- The site is configured in `hugo.toml`
- The theme is imported from "github.com/hugo-sid/hugo-blog-awesome"
- Content is stored in the `content/posts` directory