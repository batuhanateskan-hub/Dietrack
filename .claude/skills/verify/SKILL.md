---
name: verify
description: How to build, run, and verify the DieTrack single-file app in this repo.
---

# Verifying DieTrack (single-file `index.html`)

The whole app is `index.html` (Tailwind CDN + Chart.js CDN + LocalStorage + Groq API).
Surface is a mobile web GUI — verify by driving it with Playwright/Chromium.

## Launch

```bash
python3 -m http.server 8123 &        # serve the repo root
node <driver>.js                      # playwright is global: require('/opt/node22/lib/node_modules/playwright')
```

Use a 390x844 viewport (mobile-first layout).

## Gotcha: CDN domains are blocked in the CCR sandbox

`cdn.tailwindcss.com` and `cdn.jsdelivr.net` get a 403 CONNECT from the proxy,
but `registry.npmjs.org` is direct-allowed. Do NOT edit index.html — shim the
CDNs with `page.route()`:

1. `npm install tailwindcss@3.4.17 chart.js@4.4.7` in the scratchpad.
2. Compile Tailwind with a config mirroring the inline `tailwind.config` in
   index.html (custom colors: page/surface/raise/hair/line/ink/ink2/mute/brand/kcal/mass/viol/good/crit)
   and `content: ['index.html']`, then fulfill `**cdn.tailwindcss.com**` with a JS
   snippet that defines `window.tailwind = {}` and injects the compiled CSS.
3. Fulfill `**cdn.jsdelivr.net**` with `node_modules/chart.js/dist/chart.umd.js`.

## Gotcha: mock the Groq API

Route `**api.groq.com**` and fulfill with
`{ choices: [{ message: { role: 'assistant', content } }] }` (errors:
`{ error: { message } }` with a non-2xx status). Append a
`:::data\n{...json...}\n:::` block to `content` to exercise the extraction engine.

## Flows worth driving

- First run: no-key banner visible → open settings (`#btn-settings`), fill
  `#s-apikey` + goals, "Save settings" → banner hides.
- Chat: fill `#chat-input`, click `#chat-send` → AI bubble must NOT contain `:::`.
- Dashboard KPIs `#kpi-kcal/#kpi-weight/#kpi-water` reflect extracted data.
- Metrics form save + history table row, edit/delete (auto-accept `dialog`).
- Reload → LocalStorage persistence of KPIs and chat.
- Error path: fulfill Gemini route with status 400 → `.bubble-err` appears.
