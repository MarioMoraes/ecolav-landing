# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static marketing landing page for **EcoLav**, a Brazilian sofa- and rug-cleaning service ("Limpeza de Sofás e Tapetes"). Single self-contained `index.html` (HTML + inline `<style>` + inline `<script>`, no build step, no framework). UI copy is **pt-BR**.

The core interaction is a **quote modal** (orçamento): a "Pedir orçamento" button opens a popup that collects `Tipo` (sofa/rug), `Largura` (width in meters), and a required photo, computes an on-screen price estimate, then sends the data to WhatsApp via a `wa.me` deep link.

## Running

No build/lint/test tooling. Open `index.html` directly, or serve it:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

Fonts (Google Fonts) and the Iconify icon component load from CDNs, so a network connection is needed for full fidelity; the layout degrades gracefully offline (gradients/system fonts, missing icon glyphs).

## Where things live

- `index.html` — the entire EcoLav site (the only file you normally edit).
- `frontend/design-modelo.html` — **design system reference** (living style guide: tokens, type ramp, colors, components, motion). `index.html` was built from this. When changing visual language, keep the two consistent.
- `frontend/index.html` + `frontend/assets/` — the original third-party Aura template (Liora Home Care) that `design-modelo.html` was reverse-engineered from. Reference only; not part of the EcoLav build. Note: its inner page CSS is corrupted (an HTML serializer ate the `<style>` inside the iframe `srcdoc`), so it does not render faithfully.

## Editing conventions

The design language is shared with `design-modelo.html` and depends on these being kept in sync inside `index.html`:

- **Design tokens** live in `:root` (`--hc-*` colors + `--bevel-*` clip-paths). Warm sand neutrals (`#f7f3ee` bg, `#2f312d` ink) + a single desaturated sage accent (`--hc-accent #8fa08c` → `--hc-accent-dark` → `--hc-accent-deep`). Build contrast with luminosity, not saturation — do not introduce a second hue.
- **Signature shape**: every surface uses a beveled `clip-path` (cut top-left + bottom-right) via the `--bevel-*` vars, sized by component weight. Reuse these vars rather than hand-writing polygons.
- **Type**: Urbanist (sans, structure) + Playfair Display *italic* applied with `class="serif"` for editorial accents. Titles use sub-unitary line-height and strong negative letter-spacing.
- **Motion**: `fadeUp` (entrance, `cubic-bezier(0.16,1,0.3,1)` + blur-resolve) and `floatSoft` (ambient). Scroll entrances use the `.reveal` class picked up by the `IntersectionObserver` near the bottom of the script — add `.reveal` to new blocks to animate them in.

## Quote-modal logic (the part most likely to change)

All in the inline `<script>` of `index.html`. Two constants at the top of the script are the intended edit points:

- `WHATSAPP_NUMERO` — **placeholder `5511999999999`; replace with EcoLav's real number** (international format, digits only).
- `PRECOS` — pricing table per service. Estimate formula is `base + porM2 * largura * altura` (area-based). Adjust rates here.

Flow: any element with `data-open-quote` opens the modal (optional `data-tipo="sofa|tapete"` preselects the type). Submitting the form validates Tipo/Largura/Altura/photo, computes the estimate, and reveals the "Confirmar e enviar no WhatsApp" button, which opens a prefilled `wa.me` message. The photo cannot be attached through a `wa.me` link — the message tells the user to attach it in the chat, and the file is only previewed client-side. Generic WhatsApp links (footer link, floating button) are wired via the `data-wa-link` attribute.
