# FigCheck

A free tool that checks scientific figures against journal submission requirements before you hit submit. Covers 30 major journals across the Nature Portfolio, Cell Press, and AAAS/Science families, plus eLife, JCI, PNAS, NEJM, Lancet, JAMA, BMJ, EMBO, Rockefeller UP, and AACR titles.

Built as a single static HTML file. No build step, no backend, no analytics, no tracking. Files never leave the user's browser — all parsing happens client-side.

## A note on how this was built

This project was built with significant AI assistance (Claude, Anthropic) for the frontend design, the TIFF/PNG/JPEG header parsing in JavaScript, and the structure of the journal specifications database. I'm a biologist (PhD candidate at UCSF, immunology and cancer), not a frontend developer — AI assistance was what made it possible to ship something polished in a weekend.

The product decisions are mine: the pain point (figure submission requirements being scattered and inconsistent), the scope (30 journals across general / oncology / immunology / cell bio / clinical), the journal list, the curated specs, the framing of free vs. Pro tier features, and the deliberate choice to keep all file parsing client-side for privacy. The journal specifications themselves are sourced from publisher author guidelines and should be verified against the journal's current submission portal before relying on them — both because publishers change requirements occasionally, and because any database curated by an AI-assisted process can contain errors.

If you find a spec that's wrong, please open an issue.

## Local preview

Open `index.html` in a browser. That's it. Or run a local server:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Before deploying — set up the waitlist endpoint

The waitlist form posts to Formspree (free tier: 50 submissions/month, no credit card required).

1. Sign up at https://formspree.io
2. Create a new form, copy the endpoint URL (looks like `https://formspree.io/f/abc123xyz`)
3. Open `index.html`, search for `YOUR_FORM_ID`, replace with your real Formspree ID
4. Save

Without this step, the form will still show a success message (graceful fallback), but submissions won't actually reach you.

## Deploy to Vercel (recommended — free, custom domain, ~5 min)

Fastest path, no terminal required:

1. Push this folder to a GitHub repo
2. Go to https://vercel.com → "Add New Project" → import the repo
3. Click "Deploy" — defaults are correct for a static site
4. You get a `figcheck-yourname.vercel.app` URL immediately

To add a custom domain:
- Buy a domain (Namecheap, Porkbun — `.com` is ~$10/yr, `.tools` or `.app` are nice for this)
- In Vercel project settings → Domains → add your domain
- Vercel shows DNS records to add at your registrar
- HTTPS is automatic

## Alternative: deploy via Vercel CLI

```bash
npm i -g vercel
cd figcheck
vercel
# follow prompts, accept defaults
```

## What it actually checks

Currently (free tier):
- File format vs journal-accepted formats
- File size vs journal max
- DPI (halftone + line art thresholds) for raster files
- Print dimensions in mm (computed from pixel size + DPI)
- Color mode (RGB / CMYK / Grayscale) — extracted from TIFF photometric tag
- Detects vector files (PDF/EPS/AI) and skips raster checks

Pro tier (waitlist):
- Embedded font detection in PDF/EPS
- Line weight enforcement at print scale
- Batch checking across multiple journals
- Auto-fix tools (DPI re-sampling, color mode conversion)

## Adding journals

Edit the `JOURNALS` array in `index.html`. Each entry needs:

```js
{
  id: 'unique-slug',
  name: 'Journal Name',
  publisher: 'Publisher Name',
  category: 'general' | 'oncology' | 'immunology' | 'cellbio' | 'clinical',
  if: '12.3',
  minDPI: { halftone: 300, line: 600, combo: 500 },
  maxWidthMM: { single: 85, full: 175 },
  maxHeightMM: 230,
  formats: ['tiff', 'eps', 'pdf'],
  colorMode: ['rgb', 'cmyk'],
  minFontPt: 7,
  maxFileSizeMB: 20,
  notes: 'One-line editor note shown after a check.'
}
```

Source the specs from each publisher's "instructions for authors" page. Re-check quarterly — they change.

## Posting checklist (when you're ready to launch)

- [ ] Replace Formspree ID
- [ ] Deploy to Vercel
- [ ] (Optional) buy a domain like `figcheck.tools` or `figurecheck.io`
- [ ] Test the checker with a real figure file from a recent submission
- [ ] Test the waitlist form end-to-end (submit, confirm email arrives)
- [ ] Post on Twitter/X with #AcademicTwitter, #PhDLife
- [ ] Post on r/labrats, r/AskAcademia, r/PhD
- [ ] Post in relevant Slack/Discord communities (lab groups, society channels)
- [ ] Email any colleagues currently submitting manuscripts

## License & disclaimers

Not affiliated with any publisher. Specs sourced from publicly available author guidelines and may go stale — always verify against the journal's current submission portal. The directory is meant as a first-pass sanity check, not a substitute for reading the actual instructions for authors.
