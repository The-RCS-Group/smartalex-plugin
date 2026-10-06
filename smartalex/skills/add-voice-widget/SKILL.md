---
name: add-voice-widget
description: Add the SmartAlex voice and chat widget to the website in this codebase, so visitors can talk to the business's AI agent. Use when the user says "add SmartAlex to my site", "install the widget", "add a voice assistant to this website", or "embed the SmartAlex chat".
---

# Add the SmartAlex widget to this site

The widget is one script tag. The job is to put it in the right place for this framework, once,
so it loads on every page.

## 1. Get the code for this account

Call `smartalex_get_widget_install_code`. It returns:

- `install_code`: the script tag for this account (a `src` plus a `data-tenant-id`)
- `snippets`: ready variants for plain HTML, React (react-helmet), WordPress and Shopify
- `instructions`: the short install guide

Use the code exactly as returned. Do not change the `src` or `data-tenant-id`, and do not invent a
snippet from memory. Only use the React snippet if the project already uses `react-helmet` or
`react-helmet-async`; do not add a dependency just for this.

## 2. Find where it belongs

Look at the project and pick the one place that renders on every page:

| Project | Where it goes |
|---|---|
| Plain HTML or a static site | Every page's `<head>`, or the shared layout/partial if there is one |
| Vite, Create React App | `index.html`, inside `<head>` |
| Next.js, App Router | `app/layout.tsx`, using `next/script` with `strategy="afterInteractive"` |
| Next.js, Pages Router | `pages/_document.tsx` or `pages/_app.tsx`, using `next/script` |
| Remix, Astro, SvelteKit, Nuxt | The root layout or document component |
| WordPress | The `snippets` WordPress variant, in the child theme's `functions.php` |
| Shopify | The `snippets` Shopify variant, in `layout/theme.liquid` |

For `next/script`, carry the `src` and `data-tenant-id` from the returned tag onto the `Script`
component unchanged.

Before adding it, search the codebase for an existing SmartAlex script. If one is there, update it
rather than adding a second.

## 3. Add it and check

1. Add the tag in the chosen place. Do not add it to individual pages as well.
2. If the site sets a Content Security Policy, allow the widget's script origin (the host in the
   returned `src`) in `script-src`, plus `connect-src` and `frame-src` if the policy restricts them.
3. Run the site locally if you can and confirm the widget button appears and the browser console
   shows no errors from it.

## 4. Tell the user

One short summary: the file you changed, and that the widget appears in the bottom-right corner of
every page once the site is deployed. If they want to change what the agent says, use the
`build-an-agent` skill.
