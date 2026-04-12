# Deploying Your Site to GitHub Pages with CMS

This guide walks you through everything from pushing the code to having a live website with a working blog CMS.

---

## Step 1: Create a GitHub Repository

1. Go to [github.com/new](https://github.com/new)
2. Name it `harry-verity-site` (or whatever you like — if you name it `YOUR_USERNAME.github.io` it becomes your main GitHub Pages site)
3. Set it to **Public**
4. Don't add a README (we already have one)
5. Click **Create repository**

## Step 2: Push the Site Files

Open Terminal on your Mac, navigate to this folder, and run:

```bash
cd /path/to/harry-verity-site
git init
git add .
git commit -m "Initial site"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/harry-verity-site.git
git push -u origin main
```

Replace `YOUR_USERNAME` with your actual GitHub username.

## Step 3: Enable GitHub Pages

1. Go to your repo on GitHub
2. Click **Settings** → **Pages** (in the left sidebar)
3. Under "Source", select **Deploy from a branch**
4. Choose branch: **main**, folder: **/ (root)**
5. Click **Save**

Your site will be live at: `https://YOUR_USERNAME.github.io/harry-verity-site/`

(If you named the repo `YOUR_USERNAME.github.io`, it'll just be `https://YOUR_USERNAME.github.io/`)

## Step 4: Custom Domain (Optional but Recommended)

If you own a domain like `harryverity.com`:

1. In **Settings → Pages**, enter your domain under "Custom domain"
2. At your domain registrar (Namecheap, Google Domains, etc.), add these DNS records:

   **A Records** (point to GitHub):
   ```
   185.199.108.153
   185.199.109.153
   185.199.110.153
   185.199.111.153
   ```

   **CNAME Record** (for www):
   ```
   www → YOUR_USERNAME.github.io
   ```

3. Back in GitHub Pages settings, tick **Enforce HTTPS**
4. Create a file called `CNAME` in your repo root containing just your domain:
   ```
   harryverity.com
   ```

DNS can take up to 24 hours to propagate, but usually works within an hour.

---

## Step 5: Set Up the Contact Form

The site uses [Formspree](https://formspree.io) for the contact form (free for up to 50 submissions/month):

1. Go to [formspree.io](https://formspree.io) and sign up
2. Create a new form — it'll give you an endpoint like `https://formspree.io/f/xyzabc123`
3. In `index.html`, find `YOUR_FORM_ID` and replace it with your form ID (e.g., `xyzabc123`)
4. Commit and push the change

## Step 6: Find Your YouTube Channel ID

The meetup section auto-pulls your latest videos. You need your channel ID:

1. Go to [youtube.com/@harryverity](https://youtube.com/@harryverity)
2. Click your profile icon → **Settings** → **Advanced settings**
3. Your Channel ID starts with `UC` and is 24 characters long
4. Alternatively, use a tool like [commentpicker.com/youtube-channel-id.php](https://commentpicker.com/youtube-channel-id.php) — paste your channel URL and it'll find it
5. In `index.html`, find `UCxxxxxxxxxxxxxxxxxx` and replace it with your real channel ID
6. Commit and push

---

## Step 7: Set Up the Blog CMS (Decap CMS)

This gives you a visual editor at `yoursite.com/admin/` where you can write and publish blog posts without touching code.

### Option A: GitHub OAuth App (Recommended for GitHub Pages)

1. Go to [github.com/settings/developers](https://github.com/settings/developers)
2. Click **New OAuth App**
3. Fill in:
   - **Application name:** Harry Verity Site CMS
   - **Homepage URL:** `https://YOUR_USERNAME.github.io/harry-verity-site/` (or your custom domain)
   - **Authorization callback URL:** `https://api.netlify.com/auth/done`
4. Click **Register application**
5. Note your **Client ID** and generate a **Client Secret**

Now connect it via Netlify (free — you only need the auth service, not hosting):

1. Go to [app.netlify.com](https://app.netlify.com) and sign up with GitHub
2. Click **Add new site** → **Import an existing project** → select your repo
3. You don't need to deploy here — just connect the repo
4. Go to **Site settings** → **Identity** → **Enable Identity**
5. Under **Identity** → **Services** → **Git Gateway**, click **Enable Git Gateway**
6. Under **Identity** → **Registration**, set to **Invite only**
7. Invite yourself (your email) under **Identity** → **Invite users**

### Option B: Simpler Alternative — Use `github` Backend Directly

If you don't want to set up Netlify, you can use the GitHub backend with a personal access token:

1. Edit `admin/config.yml` and change the backend to:
   ```yaml
   backend:
     name: github
     repo: YOUR_USERNAME/harry-verity-site
     branch: main
   ```
2. When you visit `/admin/`, it'll redirect you to GitHub to authorize

### Using the CMS

1. Visit `https://yoursite.com/admin/`
2. Log in with your credentials
3. Click **Blog Posts** → **New Blog Post**
4. Write your post using the visual editor
5. Click **Publish**

The CMS will commit the new post as a markdown file to your `_posts/` folder.

**Important:** Since this is a static site (not Jekyll), after publishing via CMS you'll also need to:
- Add the post to `blog-posts.json` (so the homepage shows it)
- Create the HTML version in `blog/` folder

For a fully automated workflow, consider adding a GitHub Action (see below).

---

## Step 8 (Optional): Automate Blog Builds with GitHub Actions

Create `.github/workflows/build-blog.yml` in your repo:

```yaml
name: Build Blog Posts
on:
  push:
    paths:
      - '_posts/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install dependencies
        run: npm install -g marked

      - name: Build blog posts
        run: |
          node -e "
          const fs = require('fs');
          const { marked } = require('marked');
          const posts = [];

          fs.readdirSync('_posts').filter(f => f.endsWith('.md')).forEach(file => {
            const content = fs.readFileSync('_posts/' + file, 'utf8');
            const frontMatter = content.match(/^---\n([\s\S]*?)\n---/);
            if (!frontMatter) return;

            const meta = {};
            frontMatter[1].split('\n').forEach(line => {
              const [key, ...val] = line.split(':');
              if (key && val.length) meta[key.trim()] = val.join(':').trim().replace(/^\"|\"$/g, '');
            });

            const body = content.replace(/^---\n[\s\S]*?\n---\n/, '');
            const html = marked(body);
            const slug = file.replace('.md', '');

            posts.push({
              title: meta.title || slug,
              date: meta.date || slug.substring(0, 10),
              slug: slug,
              excerpt: meta.excerpt || ''
            });

            // Generate blog HTML page
            const template = \`<!DOCTYPE html>
<html lang=\"en\"><head>
<meta charset=\"UTF-8\"><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\">
<title>\${meta.title} — Harry Verity</title>
<link href=\"https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;500&family=Inter:wght@300;400;500&display=swap\" rel=\"stylesheet\">
<style>*{box-sizing:border-box;margin:0;padding:0}body{font-family:'Inter',sans-serif;font-size:17px;line-height:1.8;color:#1A1A1A;background:#FAFAF8}.c{max-width:680px;margin:0 auto;padding:120px 24px 80px}.back{display:inline-block;font-size:14px;color:#888;margin-bottom:40px;text-decoration:none}.back:hover{color:#1A1A1A}.meta{font-size:13px;color:#888;text-transform:uppercase;letter-spacing:.05em;margin-bottom:12px}h1{font-family:'Playfair Display',serif;font-size:clamp(28px,4vw,40px);font-weight:500;line-height:1.2;margin-bottom:32px}h2{font-family:'Playfair Display',serif;font-size:24px;margin:40px 0 16px}p{color:#4A4A4A;margin-bottom:20px}a{color:#E41E2B}hr{border:none;border-top:1px solid rgba(0,0,0,.08);margin:40px 0}ul,ol{margin:0 0 20px 24px;color:#4A4A4A}</style>
</head><body><div class=\"c\">
<a href=\"/\" class=\"back\">← Back to home</a>
<p class=\"meta\">\${new Date(meta.date).toLocaleDateString('en-GB',{day:'numeric',month:'long',year:'numeric'})}</p>
<h1>\${meta.title}</h1>
\${html}
</div></body></html>\`;

            fs.writeFileSync('blog/' + slug + '.html', template);
          });

          // Sort by date descending
          posts.sort((a, b) => b.date.localeCompare(a.date));
          fs.writeFileSync('blog-posts.json', JSON.stringify(posts, null, 2));
          "

      - name: Commit changes
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add blog/ blog-posts.json
          git diff --staged --quiet || git commit -m "Auto-build blog posts"
          git push
```

This means: every time a new markdown post is added to `_posts/` (via the CMS or manually), it automatically generates the HTML and updates the blog index.

---

## Summary of What to Replace

| Find this in `index.html` | Replace with |
|---|---|
| `UCxxxxxxxxxxxxxxxxxx` | Your YouTube Channel ID |
| `YOUR_FORM_ID` | Your Formspree form ID |

| Find this in `admin/config.yml` | Replace with |
|---|---|
| `YOUR_GITHUB_USERNAME/harry-verity-site` | Your actual GitHub username/repo |
| `YOUR_GITHUB_USERNAME.github.io` | Your actual GitHub Pages domain |

---

## You're Done!

Your site should now be live with:
- A beautiful editorial homepage
- Working contact form
- Auto-updating YouTube feed
- Interactive workflow audit quiz
- A CMS at `/admin/` for writing blog posts
- Automated blog builds via GitHub Actions

If you want to make any design tweaks, everything is in `index.html` — the CSS is at the top, the content is in the HTML, and the JavaScript is at the bottom.
