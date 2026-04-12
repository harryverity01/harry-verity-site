# Harry Verity — Personal Website

A clean, editorial personal website for AI consulting, journalism, and content creation. Built for GitHub Pages with Decap CMS for blog management.

## Quick Start

1. Push this folder to a new GitHub repository
2. Enable GitHub Pages in Settings → Pages → Source: `main` / root
3. Set up Decap CMS authentication (see DEPLOY-GUIDE.md)
4. Update the YouTube channel ID and Formspree form ID

## Structure

```
├── index.html              # Main single-page site
├── admin/
│   ├── index.html          # Decap CMS admin panel
│   └── config.yml          # CMS configuration
├── blog/                   # Individual blog post HTML files
├── _posts/                 # Blog post markdown source files
├── blog-posts.json         # Blog index data (update when adding posts)
├── assets/
│   ├── images/
│   │   ├── hv-logo.svg
│   │   ├── hero-presenting.jpg
│   │   └── group-photo.jpg
│   └── uploads/            # CMS media uploads go here
└── DEPLOY-GUIDE.md         # Step-by-step deployment instructions
```

## Two Things to Update Before Going Live

1. **YouTube Channel ID** — In `index.html`, search for `UCxxxxxxxxxxxxxxxxxx` and replace with your real channel ID
2. **Contact Form** — In `index.html`, search for `YOUR_FORM_ID` and replace with your Formspree form ID (free at formspree.io)
