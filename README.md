# glanceneuro.org

The GLANCE landing page. Flat HTML — **no build step, no dependencies**. What is
in this repo is exactly what gets served.

```
index.html        the page (all CSS inline, so one file is the whole design)
logo.png          light-mode logo, copied from glance-neuro/resources/
logo-darkmode.png dark-mode logo
board.jpg         photo of the assembled board  <-- ADD THIS, see below
favicon.svg
robots.txt        points at the sitemap
sitemap.xml       one URL; add more if the site grows
```

## Add the board photo

`index.html` references **`board.jpg`** and the page will show a broken image
until it exists. Save the photo of the MicroZed seated on the carrier into this
directory under that name.

Two things worth doing to it first:

- **Resize to about 1600 px wide and save as JPEG at ~80% quality.** A phone
  photo is often 3–8 MB; that is the difference between a page that loads
  instantly and one that does not, and Google measures it.
- **Add the real dimensions to the `<img>` tag** — `width="1600" height="1100"`
  or whatever it actually is. Without them the page reflows when the photo
  loads. Everything works without this; it is just a little jumpy.

If you would rather use a different filename or a `.webp`, change the `src` in
`index.html` to match.

## Deploying on Cloudflare Pages

1. Push this repo to `glanceneuro/glance-neuro-website` on GitHub.
2. Cloudflare dashboard → **Workers & Pages** → **Create** → **Pages** →
   **Connect to Git**, and pick the repo.
3. Build settings — the important part:
   - **Framework preset:** `None`
   - **Build command:** *leave empty*
   - **Build output directory:** `/`
4. Deploy. You get a `*.pages.dev` URL immediately.
5. **Custom domains** → add `glanceneuro.org` and `www.glanceneuro.org`. Because
   the domain is already in your Cloudflare account, the DNS records are created
   for you; there are no A records to copy and no TLS to configure.

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
