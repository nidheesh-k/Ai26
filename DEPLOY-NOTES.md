# AI46 website: deploy notes for ai46.ai

## What this is
A single static page exported from Claude Design. No build step, no server code, no database.
Upload everything in this folder to the web root, keeping the `fonts/` folder:

```
index.html                    the site (the export, plus link-preview and icon tags in the outer <head>)
og.png                        1200x630 link-preview image
favicon.ico / favicon.svg     browser icons
apple-touch-icon.png          iPhone home-screen icon
fonts/DepartureMono-Regular.woff2   required: the page loads it from /fonts/ (it is not inside the export)
fonts/DepartureMono-LICENSE.txt     SIL Open Font License
```

## Hosting and DNS
- Any static host works: Vercel, Netlify, Cloudflare Pages, S3 + CloudFront, nginx. HTTPS required.
- Canonical address is `https://ai46.ai/`. Redirect `www.ai46.ai` to it with a 301.
- Change only the records for the website (apex A/ALIAS, `www` CNAME).
  Do not touch MX, SPF, DKIM or DMARC records: the contact email fj@ai46.ai depends on them.
- Caching: serve `index.html` with `Cache-Control: no-cache` so updates show at once. Images and fonts can be cached long.

## Third-party requests the page makes at runtime
These must be reachable from visitors' browsers:
- `unpkg.com`: three.js 0.180.0 and the Lenis snap plugin 1.3.4 (`esm.sh` is used as a fallback)
- `cdn.jsdelivr.net`: the hand model (`@webxr-input-profiles/assets@1.0.20`, generic-hand left/right .glb)
- `formsubmit.co`: contact-form submissions (POST `https://formsubmit.co/ajax/fj@ai46.ai`)

React and the Archivo fonts are embedded; the Google Fonts preconnect hints are harmless.

## Only if you add a Content-Security-Policy
The page unpacks itself into `blob:` URLs and its loader uses `new Function`, so a strict CSP will break it.
Starting point, then check the browser console after enabling:

```
default-src 'self';
script-src 'self' 'unsafe-inline' 'unsafe-eval' blob: https://unpkg.com https://esm.sh;
connect-src 'self' blob: data: https://unpkg.com https://esm.sh https://cdn.jsdelivr.net https://formsubmit.co;
style-src 'self' 'unsafe-inline' blob:;
font-src 'self' data: blob:;
img-src 'self' data: blob:;
worker-src 'self' blob:;
```

## Contact form: one-time activation
1. fj@ai46.ai must already receive mail.
2. From the live site (https://ai46.ai), send one test message through "Contact us".
3. FormSubmit emails fj@ai46.ai an activation link. Click it.
4. Until that click, the form shows "The message could not be sent. Please write to fj@ai46.ai directly." This is expected.

## What was tested
In Chrome at 1440x900 and 390x844, with every runtime file served: all six chapters render,
no script errors, no missing files once `fonts/` is in place, and the form sends a correctly formed request.
Not tested: Safari/iOS and real GPUs. Please check on an iPhone, an Android phone and a Mac before announcing.

## Optional, later
- Self-host three.js, Lenis and the hand model to remove the CDN dependency. This needs the unbundled source
  (Claude Design: hand off to Claude Code), not this exported file.
- Privacy policy: the page sends visitor IP addresses to unpkg/jsDelivr/esm.sh and form contents to FormSubmit.
- Analytics: none is included.

## Credits (keep)
Hand model: WebXR Input Profiles "generic-hand", MIT License, Copyright (c) 2019 Amazon.
three.js and Lenis: MIT. Archivo and Departure Mono: SIL Open Font License. Notices are also in the page source.
