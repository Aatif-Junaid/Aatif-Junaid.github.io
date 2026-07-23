# aatifmulla.me

Personal site for Aatif Junaid Mulla. Static HTML and CSS, hosted on GitHub Pages. No framework, no build step.

## Files
- `index.html`: all the content and copy for the home page
- `case-studies.html`: the five go-to-market case studies
- `styles.css`: all the styling for both pages
- `resume.pdf`: the file both Resume buttons link to
- `og-image.jpg`: the preview card shown when the site is shared
- `favicon.svg`, `favicon.ico`, `favicon-16x16.png`, `favicon-32x32.png`, `apple-touch-icon.png`: browser and device icons
- `sitemap.xml`: the page list for search engines
- `assets/`: the headshot and the company and school logos
- `CNAME`: tells GitHub Pages the custom domain is aatifmulla.me (do not delete)

## Edit the copy
Open `index.html` or `case-studies.html` and edit the text between the tags. Save, refresh the browser.

## Cache busting
Both pages load the stylesheet as `styles.css?v=NN`. Bump `NN` in both files on every CSS change, or returning visitors keep the old styles. Do the same for `resume.pdf?v=...` on both Resume buttons whenever the PDF is replaced.

## Preview locally
From this folder, run:

    python -m http.server 8000

Then open http://localhost:8000 in your browser.

## Deploy
Pushing to the `main` branch updates the live site automatically through GitHub Pages. It goes live in roughly one to three minutes.
