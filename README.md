# Hex byte reverser

A single-file, branded HTML tool that converts a decimal number to uppercase
hex and flips the byte order. Built for the Initio Learning Trust IT knowledge
base — hosted on GitHub Pages, embedded via iframe in a Halo ITSM KB article.

No build step, no dependencies, no external network requests.

## Files

\`\`\`
/
├── index.html              Main tool (self-contained)
├── assets/
│   └── logo.svg            Initio lockup — replace placeholder with the licensed file
├── embed-example.html      Demo of the auto-resize iframe pattern
└── README.md               This file
\`\`\`

## Local preview

Open `index.html` directly in a browser, or serve the folder:

\`\`\`sh
python3 -m http.server 8080
# then visit http://localhost:8080/
\`\`\`

To try the auto-resize pattern, open `http://localhost:8080/embed-example.html`
and type a long number into the tool — the host iframe should grow/shrink as
content reflows.

## Replacing the placeholder logo

`assets/logo.svg` currently holds a placeholder lockup. Drop the official
Initio Learning Trust SVG into the same path (`assets/logo.svg`) and commit.
Do not resize, recolour, or reposition elements of the supplied logo.

## Embedding in Halo ITSM

In the KB article rich text editor, click the HTML (source) button and paste:

\`\`\`html
<iframe
  src="https://USERNAME.github.io/REPO_NAME/"
  width="100%"
  height="420"
  style="border: 0; display: block; max-width: 680px; margin: 0 auto;"
  title="Hex byte reverser">
</iframe>
\`\`\`

Replace `USERNAME` and `REPO_NAME` with the GitHub Pages values. If Halo's
sanitiser strips the `style` attribute, fall back to:

\`\`\`html
<iframe src="https://USERNAME.github.io/REPO_NAME/" width="100%" height="420" frameborder="0"></iframe>
\`\`\`

### About the iframe height

Halo's rich text editor very likely strips `<script>` tags, so the
parent-side resize listener won't run and the iframe will stay at its static
`height="420"`. The tool is laid out to fit comfortably inside 420 px at a
640 px width.

The auto-resize messaging is still included in `index.html`. It works in any
host that allows inline scripts (internal portals, SharePoint, Confluence) and
is a no-op when scripts are blocked.

## GitHub Pages deployment

1. Push this repo to GitHub (public).
2. Repo → **Settings** → **Pages** → **Build and deployment**.
3. Source: **Deploy from a branch**. Branch: `main`. Folder: `/ (root)`. Save.
4. Wait about a minute, then visit `https://USERNAME.github.io/REPO_NAME/`.
5. Sanity check: verify the page loads, the logo shows (case-sensitive path
   `assets/logo.svg`), and the tool responds as you type.
6. Paste the embed snippet above into the Halo KB article.

## Behaviour summary

Two modes, switched via the segmented control at the top of the tool:

**Decimal → Reversed hex**
- Parses input with `BigInt()` — no precision loss on very large numbers.
- Empty input → em-dashes, no error.
- Negative input → "Please enter a non-negative number".
- Non-numeric input → "Not a valid whole number".
- Hex is uppercase and left-padded with a single `0` to an even length.
- Byte order is reversed by splitting into 2-char chunks and reversing the
  array.

**Reversed hex → Decimal**
- Accepts hex with or without whitespace; case-insensitive.
- Empty input → em-dashes, no error.
- Non-hex characters → "Not a valid hex string".
- Odd character count → "Hex must have an even number of characters".
- Un-reverses the bytes and parses the result via `BigInt('0x' + hex)`.

Both modes update live on every keystroke. Copy buttons use
`navigator.clipboard.writeText()` with a `document.execCommand('copy')`
fallback. The mode switch preserves each direction's last-used input.

## Acceptance checks

- Decimal → Reversed hex: `2196032614` → `82E4CC66` → reversed `66CCE482`.
- Reversed hex → Decimal: `66CCE482` → `82E4CC66` → `2196032614`.
- Very large numbers (e.g. `99999999999999999999`) round-trip both directions
  without loss of precision.
- Hex input accepts spaced and lowercase forms (`66 cc e4 82`).
- Dark mode (OS-level or DevTools emulation) renders legibly.
- Keyboard: Tab cycles through mode buttons → input → copy buttons. Focus
  rings visible.
- No console errors, no external requests.
