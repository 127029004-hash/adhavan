# Make Your Datz — Deployment Guide

Static site + Decap CMS (Netlify CMS) admin panel + Cloudinary media + Netlify Identity login.

## Folder layout

```
site/
├── index.html              # public website (gallery + videos load dynamically)
├── admin/
│   ├── index.html          # /admin login + dashboard
│   └── config.yml          # CMS schema  ← EDIT cloud_name + branch
├── content/
│   ├── gallery.json        # written by CMS — photo gallery items
│   └── videos.json         # written by CMS — video items
├── netlify.toml            # Netlify config (no build step)
└── README-DEPLOY.md
```

The website fetches `/content/gallery.json` and `/content/videos.json` on
page load. The CMS edits those files via Git Gateway → commits to your repo
→ Netlify auto-deploys → website updates.

## One-time setup

### 1. Create a Cloudinary account (free tier is fine)
- Sign up at https://cloudinary.com
- Dashboard → copy **Cloud name** and **API Key**
- Open `site/admin/config.yml`, replace `YOUR_CLOUDINARY_CLOUD_NAME` and
  `YOUR_CLOUDINARY_API_KEY` with those values.

### 2. Push this `site/` folder to a new GitHub repo
- Create a repo on GitHub (e.g. `myd-photography`)
- Push the contents of the `site/` folder to the repo root (NOT inside a
  `site/` subfolder — the repo root should contain `index.html`, `admin/`,
  `content/`, `netlify.toml`).

### 3. Deploy to Netlify
- https://app.netlify.com → **Add new site → Import an existing project**
- Connect GitHub, pick the repo
- Build command: *(leave empty)*
- Publish directory: `.`
- Click **Deploy site**

### 4. Enable Netlify Identity (login system)
- In your Netlify site dashboard → **Site configuration → Identity → Enable Identity**
- **Registration preferences** → set to **Invite only** (so random people
  can't sign up as admins)
- **Services → Git Gateway → Enable Git Gateway** (this is what lets the
  CMS write back to your GitHub repo)

### 5. Invite yourself as the admin user
- Netlify dashboard → **Identity → Invite users** → enter your email
- Check your email, click the invite link, set a password
- You're done — go to `https://YOUR-SITE.netlify.app/admin/` and log in.

### 6. (Optional) Custom domain
- Netlify → **Domain management → Add custom domain**

## Using the admin panel

Open `https://YOUR-SITE.netlify.app/admin/`

You'll see two collections:

- **Photo Gallery** — Add/edit/delete/reorder photos. Each photo has:
  - Image (uploaded to Cloudinary)
  - Category: Wedding & Pre-Wedding / Child / Portrait
  - Display order (lower = first)
- **Videos** — Add/edit/delete/reorder videos. First 4 by order appear in
  the homepage featured grid; all appear in the Videos overlay page.

Drag items in the list widget to reorder. Click **Publish**. Netlify
auto-deploys within ~30 seconds and the website updates.

## How the dynamic gallery works

- `site/index.html` has an empty `PF_ITEMS = []` array and empty video grid
  containers (`#vgFeaturedGrid`, `#vgpGrid`).
- A loader script (injected before `</body>`) fetches
  `/content/gallery.json` and `/content/videos.json`, fills the arrays,
  and calls the existing `buildPfCatGrid()` render function.
- All existing CSS, animations, lightbox, filters (All / Wedding /
  Child / Portrait), responsiveness — **unchanged**.

## Notes

- The CMS maps the CMS category `child` to the internal filter key `baby`
  used by the existing render code — the **All / Wedding & Pre-Wedding /
  Child / Portrait** filter buttons keep working exactly as before.
- Items without a `Display order` value are pushed to the end.
- The `uploads/` folder mentioned in `config.yml` is a fallback that only
  triggers if Cloudinary is misconfigured.
- A handful of non-gallery decorative images (hero logo, profile photo)
  are still inline in `index.html` as base64. They are NOT part of the
  gallery and were left untouched to preserve the design. If you want
  those moved to the CMS too, ask and I'll add a third collection.
