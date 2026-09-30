# glanceneuro.org

The GLANCE landing page. Flat HTML — **no build step, no dependencies**. What is
in this repo is exactly what gets served.

```
public/           <- everything in here is the website, and nothing else is
  index.html      the page (all CSS inline, so one file is the whole design)
  logo.png        light-mode logo, copied from glance-neuro/resources/
  logo-darkmode.png
  board.jpg       photo of the assembled board (1600 px, EXIF stripped)
  board.webp      same photo, ~35% smaller, for browsers that take it
  favicon.svg
  robots.txt
  sitemap.xml
wrangler.jsonc    only needed for the Workers deploy path; see below
README.md         this file
```

Site files live in `public/` so that this README and `wrangler.jsonc` are not
served as part of the site.

## The board photo

`public/board.jpg` is 1600 px wide, re-encoded from the 2475 px original. That
took it from 1.7 MB to 302 KB, and `board.webp` is 197 KB for browsers that
accept it — the `<picture>` element serves whichever fits.

**The original carried GPS coordinates in its EXIF.** Re-encoding dropped all
metadata, so the published file does not say where the photo was taken. Worth
remembering for any phone photo that goes on the site.

The uncompressed original is kept out of the repo. To replace the photo, drop a
new one in and re-run the resize; if its aspect ratio differs, update
`width`/`height` on the `<img>` in `index.html` so the page does not reflow as it
loads.

## Deploying on Cloudflare

Cloudflare now offers two ways to host a static site, and its repo-import flow
defaults to the first. Either works; this repo is set up for both.

### Workers (what the dashboard gives you by default)

Recognisable by a **Deploy command** field pre-filled with `npx wrangler deploy`.
Leave it as it is, leave **Build command** empty, and let `wrangler.jsonc` do the
rest — it points at `public/` and declares no Worker code, which is what makes it
a plain static site.

### Pages (the older, simpler flow)

**Workers & Pages → Create → Pages → Connect to Git**, then:

- **Framework preset:** `None`
- **Build command:** *empty* (if the form insists on something, `exit 0`)
- **Build output directory:** `public`

There is no deploy command in this flow — deployment is the part Cloudflare does
for you.

### Either way

Deploy, and you get a `*.pages.dev` or `*.workers.dev` URL to check immediately.
Then **Custom domains** → add `glanceneuro.org` and `www.glanceneuro.org`. Since
the domain is already in your Cloudflare account, the DNS records are created for
you: no A records to copy, no TLS to configure.

Every push to `main` redeploys. Pull requests get their own preview URL.

## The other two domains

Pick **one** canonical domain and redirect the rest to it. Three domains serving
the same page splits the ranking signals and reads as duplicate content.

These files assume **`glanceneuro.org`** is canonical. If you prefer `.com`,
change the `<link rel="canonical">` and `og:url` in `index.html`, the URL in
`sitemap.xml` and `robots.txt`, and reverse the redirects below.

For `glanceneuro.com` and `glance-neuro.org`, in each zone: **Rules → Redirect
Rules → Create**, with a wildcard 301 to the canonical host so deep links survive:

| field | value |
|---|---|
| When incoming requests match | `Hostname` `equals` `glanceneuro.com` |
| Then | Dynamic redirect, status **301** |
| Expression | `concat("https://glanceneuro.org", http.request.uri.path)` |

A redirect is right *here* — consolidating domains you own onto one. It is the
wrong tool for pointing a domain at a GitHub page: a 301 tells Google to index
the destination instead, so the domain itself would never appear in results.

## Getting it indexed

Deploying is not indexing. In order of how much each actually matters:

1. **Inbound links.** The single biggest factor, and the one that needs a human.
   Link `glanceneuro.org` from `kemerelab.com`, from the Rice neuroengineering
   page, from the **About → Website** field of all three GitHub repos, from the
   org profile README, and from any paper or preprint. A domain nothing links to
   is a domain Google has no reason to trust.
2. **Google Search Console.** Add the domain, verify by DNS TXT (one record, and
   Cloudflare makes it a two-click job), submit `sitemap.xml`, then use **URL
   Inspection → Request indexing**. That is the difference between days and
   weeks.
3. **Check Cloudflare is not blocking crawlers.** Bot Fight Mode, and especially
   "Under Attack" mode, can serve Googlebot a challenge page instead of the site.
   Verify with Search Console's URL Inspection → *Test live URL* after deploying.

## Editing

It is one HTML file with the CSS inline. Open it in a browser from disk to
preview — there is nothing to run.

Content is drawn from `glance-neuro/README.md`; if the specs there change, they
should change here too. The page deliberately claims only what is verifiable: the
channel count, the sample rate, one datagram per sample, three streams. End-to-end
closed-loop latency is **not** quoted, because it depends on the host and on the
Open Ephys audio buffer size rather than on this hardware — see
`glance-neuro-plugin/docs/latency.md`.
