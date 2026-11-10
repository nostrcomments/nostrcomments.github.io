# nostrcomments.github.io

The website for [NostrComments](https://github.com/briskness-byte/NostrComments): the landing page,
the thread reader and the embed for site owners.

**This is an organisation site** — the repository `nostrcomments.github.io` in the `nostrcomments`
organisation — so it is served from the root of its own hostname. Every favicon lookup on the web
uses the hostname alone (awesome-nostr builds its icons from `https://<hostname>/favicon.ico`), which
a project page under another site would not have.

It moved here from `briskness-byte.github.io` in November 2026. That old site stays up: its pages
forward here, with the address and query intact so old "Read the thread" links land on the same
thread, and its `embed/frame.html` keeps serving the box for sites that embedded it from there.

Hand-written files, no build step, no dependencies — the same claim the extension makes about
itself, so it had better be true here too.

- `index.html` — the landing page, with `screenshot.png`, `demo.mp4` and `demo-poster.png`
- `thread/` — the thread reader, for people without the extension
- `embed/` — the comment box for site owners
- `lib/` — the reading and drawing code shared by those two
- `tests/` — offline checks for the reader and the embed

## Setting it up

    git init && git add -A && git commit -m "Landing page"
    git branch -M main
    git remote add origin git@github-brisknessbyte:nostrcomments/nostrcomments.github.io.git
    git push -u origin main

Then Settings → Pages → Source: `main` / root. It is live at https://nostrcomments.github.io/
within a minute or two.

## The thread reader

`/thread/` reads a page's comment thread straight from the relays, in the visitor's browser. The
reading code is in `lib/read.js`, shared with the embed; the page scripts and styles are separate
files because the Content-Security-Policy allows nothing inline.

    node tests/run.mjs           # offline: policy, rendering, signatures, the reading rules
    node thread/check.mjs        # drift from the extension (fails if any copied rule changed),
                                 # then a real thread read from the real relays
    node tests/make-fixtures.mjs # re-sign tests/fixtures.json; needs the extension repo next door

The reading rules are copied from the extension's content.js, block by block, between
`// content.js: <name>` markers in lib/read.js. When check.mjs reports a block as changed, copy
the new lines from content.js into that block and run both checks again.

## The embed

`/embed/` tells site owners how to put the thread under their articles, and runs the snippet as a
live example. The snippet is inline and fixed: a button that, when pressed, opens a sandboxed
iframe on `/embed/frame.html` with the page address in the fragment. The frame uses the same
`lib/read.js` and `lib/draw.js` as `/thread/`. The snippet's CSP hash is given on the page;
`tests/run.mjs` fails if the snippet shown, the snippet that runs and that hash ever differ.
