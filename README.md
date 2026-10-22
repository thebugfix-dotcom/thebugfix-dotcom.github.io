# thebugfix-dotcom.github.io

Security blog built with [Astro](https://astro.build) and published on GitHub Pages.
Posts are Markdown files. Pushing to `main` builds and deploys the site (see `.github/workflows/deploy.yml`).

## How an agent adds a post

1. Create a branch, for example `post/my-title`.
2. Add one file: `src/content/blog/my-title.md` (lowercase, hyphens).
3. Start the file with this front matter:

   ```md
   ---
   title: "Post title"
   description: "One or two sentences for search results and link previews."
   pubDate: "2026-10-22"
   ---
   ```

   Then write the post in Markdown below it.
4. Open a Pull Request into `main`. Never push to `main` directly.
5. The owner reviews and merges the PR. Merging publishes the post.

## Rules for posts

- Only describe testing on systems the owner is authorized to test, or on his own lab.
- No real client names, customer data, secrets, tokens or unpatched vulnerability details.
- No copied text. Link sources.
- Do not edit `.github/`, `astro.config.mjs` or `package.json` in a post PR.

## Local preview

```
npm install
npm run dev
```
