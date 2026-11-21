---
name: design-wireframe
description: Create low-fidelity, grayscale wireframes of app or web screens as one editable HTML file. Elements keep their real shapes (toggles, inputs, steppers, chips, tab bars, cards) but there's no color, no device frame, no status bar and no animation. Text that varies is shown as skeleton bars and images as placeholder boxes. Use this skill whenever the user asks for a wireframe, low-fi or lo-fi mockup, skeleton layout, screen blueprint, UX flow, layout sketch, or wants to "rough out", "block out" or "sketch" screens for a mobile app, web app, dashboard or landing page, even if they don't say the word "wireframe". Also use it when they want to change or extend a wireframe made earlier.
---

# Grayscale Wireframe

Make wireframes that show **structure and content hierarchy, not visual design**. The result should look like a clean product-design wireframe: white screens on a light gray canvas. Each element has its true shape (a toggle looks like a toggle, a stepper like `− ▢ +`), everything is grayscale, and anything that's content rather than structure is a placeholder.

The point of a wireframe is to let people discuss *what goes where* without arguing about colors, fonts or photos. Every rule below serves that goal: anything that pulls attention toward visual polish gets removed, and anything that makes the layout easy to read and easy to change stays.

## What it looks like

**Always**
- Pure grays only (R = G = B), from white to near-black. Black is used for the single primary action on a screen.
- The real shapes of UI controls, at realistic sizes (touch targets of about 44–48px on mobile).
- Outline icons drawn with a thin stroke (1.5px) in one consistent style.
- One sans-serif font family, with hierarchy shown only through size and weight.
- Flat surfaces. Separate things with 1px borders, light gray fills and whitespace.

**Never**
- Device frames, notches, status bars (9:41, battery, wifi), home indicators or browser chrome. A screen is a plain white rectangle that holds only the app's content.
- Color, gradients, drop shadows, blur or textures.
- Animations, transitions or hover effects.
- Real photos, illustrations, brand logos or emoji. Use an image placeholder, a gray circle for an avatar, and a `Logo` pill for branding.
- Lorem ipsum paragraphs. Use skeleton bars instead.

## Real text or skeleton bar?

This choice matters most for how the wireframe reads. **Structural text is real; variable content is a skeleton.**

| Real text (the structure depends on it) | Skeleton bar or placeholder (content that changes) |
|---|---|
| Screen titles ("Checkout", "My Cart") | Names of people and products |
| Button labels ("Add to Cart", "Sign up") | Descriptions, body copy, messages |
| Navigation items, tab labels | Notification and list item text |
| Section headers ("Delivery option", "TODAY") | Prices and numbers in lists (unless the layout is about them) |
| Form labels and input placeholders ("Email") | Dates, timestamps, metadata |
| Table column headers, field labels ("Items", "Total") | Values in summary rows |
| Links that drive the flow ("Forgot password?", "See all") | Images: placeholder box with an image icon |

When it's unclear, ask: "If this text were different, would the layout change?" If no, it's a skeleton. If the user gives real content or asks for realistic copy, use real text. Keep it short and plausible.

Make skeleton bars vary in width (e.g. 60% / 85% / 40%) so they read like real lines of text. Use 2–3 bars for a text block, one wider and taller bar for a title, and short bars for metadata.

## Workflow

1. **Pin down the scope.** Work out which screens, which platform (mobile at 390px wide, tablet at 834px, desktop at 1440px) and which flow. Infer these from the request; ask only if it's genuinely ambiguous and a wrong guess would mean redoing several screens.
2. **List the sections for each screen, top to bottom**, before writing any markup. For example, Product Details: app bar (back, title, more) → image carousel with dots → title skeleton + quantity stepper → divider → description skeleton → two buttons (primary "Add to Cart", secondary "Buy Now").
3. **Mark each element** as real text or skeleton, using the table above.
4. **Build** with the base file and component classes below. Put every screen on one board, side by side, in flow order, with a small label above each.
5. **Check** against the checklist at the end, then deliver.

## Output format: one editable HTML file

Deliver one self-contained `.html` file: no external assets, no JavaScript needed. Make it easy to edit by hand and by a later agent:

- **Design tokens** in `:root` (grays, radius, spacing, screen widths). Changing `--radius` from `8px` to `0` turns the whole wireframe from rounded to sharp-cornered.
- **Component classes** (`.btn`, `.input`, `.toggle`…) instead of inline styles. Width utilities such as `.w-60` are the only exception.
- **One `<section>` per screen**, with an HTML comment naming it and a `data-screen` attribute.
- **Icons in an SVG sprite** defined once and referenced with `<use href="#i-name">`. To swap an icon, change one word.
- **Only flex and grid** for layout. Use absolute positioning only for overlays such as a floating tab bar.

When the user asks for changes, edit the existing file in place and keep its classes and structure. Don't regenerate it from scratch, so their own edits survive.

### Base file

Start from this file and add component styles as needed:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Wireframe – [Product name]</title>
<style>
:root{
  /* grayscale tokens */
  --canvas:#E6E6E6; --surface:#FFFFFF; --fill:#EFEFEF; --skeleton:#E2E2E2;
  --line:#E5E5E5; --border:#C9C9C9; --muted:#8C8C8C; --text:#1A1A1A; --ink:#111111;
  /* shape & spacing */
  --radius:8px; --pill:999px; --gap:16px; --pad:20px;
  /* screen widths */
  --mobile:390px; --tablet:834px; --desktop:1440px;
  --font: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
}
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;animation:none!important;transition:none!important}
body{background:var(--canvas);font:14px/1.4 var(--font);color:var(--text);padding:48px}
button{font:inherit;color:inherit;background:none;border:0}

/* board & screens */
.board{display:flex;flex-wrap:wrap;gap:48px;align-items:flex-start}
.frame{display:flex;flex-direction:column;gap:10px}
.frame-label{font-size:12px;color:var(--muted);letter-spacing:.06em;text-transform:uppercase}
.screen{position:relative;width:var(--mobile);min-height:844px;background:var(--surface);
  display:flex;flex-direction:column;gap:var(--gap);padding:0 var(--pad) var(--pad);overflow:hidden}
.screen.tablet{width:var(--tablet);min-height:1112px}
.screen.desktop{width:var(--desktop);min-height:900px;padding:0;flex-direction:row;gap:0}

/* type */
.h1{font-size:28px;font-weight:600} .h2{font-size:18px;font-weight:600}
.title{font-size:16px;font-weight:500} .caption{font-size:12px;color:var(--muted);text-transform:uppercase;letter-spacing:.04em}
.muted{color:var(--muted)} .link{font-size:13px;text-decoration:underline}

/* skeletons & placeholders */
.sk{display:block;height:10px;border-radius:var(--pill);background:var(--skeleton)}
.sk.lg{height:16px} .sk.sm{height:8px}
.stack{display:flex;flex-direction:column;gap:8px}
.w-20{width:20%}.w-30{width:30%}.w-40{width:40%}.w-50{width:50%}.w-60{width:60%}.w-70{width:70%}.w-85{width:85%}.w-100{width:100%}
.img{background:var(--fill);display:grid;place-items:center;color:var(--muted);border-radius:var(--radius)}
.img .icon{width:48px;height:48px}
.ar-1{aspect-ratio:1}.ar-43{aspect-ratio:4/3}.ar-169{aspect-ratio:16/9}
.avatar{width:48px;height:48px;border-radius:50%;background:var(--fill);flex:none}
.logo{display:inline-flex;align-items:center;height:32px;padding:0 14px;border:1px solid var(--text);border-radius:var(--pill);font-weight:500}

/* icons */
.icon{width:20px;height:20px;fill:none;stroke:currentColor;stroke-width:1.5;stroke-linecap:round;stroke-linejoin:round;flex:none}
.icon-btn{width:40px;height:40px;display:grid;place-items:center;border-radius:50%}
.icon-btn.outline{border:1px solid var(--line)}

/* layout helpers */
.row{display:flex;align-items:center;gap:12px} .between{justify-content:space-between} .grow{flex:1}
.grid-2{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.divider{height:1px;background:var(--line)}
.or{display:flex;align-items:center;gap:12px;color:var(--muted);font-size:13px}
.or::before,.or::after{content:"";flex:1;height:1px;background:var(--line)}

/* app bar */
.appbar{display:grid;grid-template-columns:40px 1fr 40px;align-items:center;height:56px;margin:0 calc(var(--pad) * -.5)}
.appbar .title{text-align:center}

/* buttons */
.btn{height:48px;padding:0 20px;display:inline-flex;align-items:center;justify-content:center;gap:8px;
  border-radius:var(--radius);font-size:15px;font-weight:500}
.btn-primary{background:var(--ink);color:#FFF}
.btn-secondary{border:1px solid var(--text)}
.btn-tertiary{border:1px solid var(--border);color:var(--muted)}
.btn-block{width:100%}
.btn-social{width:88px;height:64px;border:1px solid var(--border);border-radius:var(--pill)}

/* form controls */
.field{display:flex;flex-direction:column;gap:6px} .label{font-size:13px}
.input{height:48px;display:flex;align-items:center;gap:10px;padding:0 14px;border:1px solid var(--border);
  border-radius:var(--radius);color:var(--muted)}
.checkbox{width:18px;height:18px;border:1.5px solid var(--border);border-radius:4px;flex:none;display:grid;place-items:center}
.checkbox.on{background:var(--ink);border-color:var(--ink);color:#FFF}
.radio{width:20px;height:20px;border:1.5px solid var(--border);border-radius:50%;flex:none;display:grid;place-items:center}
.radio.on{border-color:var(--ink)} .radio.on::after{content:"";width:10px;height:10px;border-radius:50%;background:var(--ink)}
.toggle{width:44px;height:26px;border:1.5px solid var(--border);border-radius:var(--pill);padding:2px;display:flex;flex:none}
.toggle::after{content:"";width:19px;height:19px;border-radius:50%;border:1.5px solid var(--border);background:var(--surface)}
.toggle.on{background:var(--ink);border-color:var(--ink);justify-content:flex-end} .toggle.on::after{border-color:var(--ink)}
.stepper{display:inline-flex;align-items:center;gap:14px;height:40px;padding:0 12px;border:1px solid var(--line);border-radius:var(--radius)}
.stepper .val{width:16px;height:22px;border-radius:var(--pill);background:var(--skeleton)}
.chip{height:34px;padding:0 16px;display:inline-flex;align-items:center;border:1px solid var(--border);border-radius:var(--pill);font-size:13px;white-space:nowrap}
.chip.active{background:var(--ink);border-color:var(--ink);color:#FFF}
.chips{display:flex;gap:8px;overflow:hidden}
.segmented{display:flex;padding:4px;background:var(--fill);border-radius:var(--pill)}
.segmented>*{flex:1;height:36px;display:grid;place-items:center;border-radius:var(--pill);font-size:13px}
.segmented>.active{background:var(--surface);border:1px solid var(--line)}

/* content blocks */
.card{border:1px solid var(--line);border-radius:var(--radius);padding:16px;display:flex;flex-direction:column;gap:12px}
.card.filled{background:var(--fill);border-color:transparent}
.list-item{display:flex;align-items:center;gap:12px;padding:14px 0;border-bottom:1px solid var(--line)}
.dots{display:flex;gap:6px;justify-content:center}
.dots>i{width:6px;height:6px;border-radius:50%;background:var(--border)} .dots>i.on{width:18px;border-radius:var(--pill);background:var(--muted)}
.section-head{display:flex;align-items:baseline;justify-content:space-between}
.summary-row{display:flex;justify-content:space-between;align-items:center;padding:6px 0}

/* mobile navigation */
.tabbar{position:absolute;left:50%;bottom:24px;transform:translateX(-50%);display:flex;gap:8px;padding:8px;
  background:var(--surface);border:1px solid var(--line);border-radius:var(--pill)}
.tabbar .tab{height:44px;min-width:44px;padding:0 12px;display:flex;align-items:center;gap:8px;border-radius:var(--pill)}
.tabbar .tab.active{background:var(--fill)}

/* desktop */
.sidebar{width:240px;flex:none;border-right:1px solid var(--line);padding:24px 16px;display:flex;flex-direction:column;gap:4px}
.nav-item{height:40px;display:flex;align-items:center;gap:12px;padding:0 12px;border-radius:var(--radius)}
.nav-item.active{background:var(--fill);font-weight:500}
.main{flex:1;display:flex;flex-direction:column;min-width:0}
.topbar{height:64px;display:flex;align-items:center;gap:16px;padding:0 24px;border-bottom:1px solid var(--line)}
.content{padding:24px;display:flex;flex-direction:column;gap:20px}
.table{width:100%;border-collapse:collapse;font-size:13px}
.table th{text-align:left;font-weight:500;color:var(--muted);padding:12px;border-bottom:1px solid var(--line)}
.table td{padding:14px 12px;border-bottom:1px solid var(--line)}
.badge{display:inline-flex;height:22px;padding:0 8px;align-items:center;background:var(--fill);border-radius:4px;font-size:12px}
</style>
</head>
<body>

<!-- ICON SPRITE: swap an icon by changing the href in <use href="#i-…"> -->
<svg width="0" height="0" style="position:absolute" aria-hidden="true">
  <symbol id="i-back" viewBox="0 0 24 24"><path d="m15 18-6-6 6-6"/></symbol>
  <symbol id="i-next" viewBox="0 0 24 24"><path d="m9 18 6-6-6-6"/></symbol>
  <symbol id="i-down" viewBox="0 0 24 24"><path d="m6 9 6 6 6-6"/></symbol>
  <symbol id="i-more" viewBox="0 0 24 24"><circle cx="5" cy="12" r="1"/><circle cx="12" cy="12" r="1"/><circle cx="19" cy="12" r="1"/></symbol>
  <symbol id="i-search" viewBox="0 0 24 24"><circle cx="11" cy="11" r="7"/><path d="m20 20-3.5-3.5"/></symbol>
  <symbol id="i-filter" viewBox="0 0 24 24"><path d="M4 6h16M4 12h16M4 18h16"/><circle cx="9" cy="6" r="2" fill="#fff"/><circle cx="15" cy="12" r="2" fill="#fff"/><circle cx="8" cy="18" r="2" fill="#fff"/></symbol>
  <symbol id="i-bell" viewBox="0 0 24 24"><path d="M6 8a6 6 0 0 1 12 0c0 7 3 9 3 9H3s3-2 3-9"/><path d="M10.3 21a1.94 1.94 0 0 0 3.4 0"/></symbol>
  <symbol id="i-bag" viewBox="0 0 24 24"><path d="M6 2 3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z"/><path d="M3 6h18M16 10a4 4 0 0 1-8 0"/></symbol>
  <symbol id="i-user" viewBox="0 0 24 24"><circle cx="12" cy="8" r="4"/><path d="M4 21a8 8 0 0 1 16 0"/></symbol>
  <symbol id="i-home" viewBox="0 0 24 24"><path d="M3 10.5 12 3l9 7.5V20a1 1 0 0 1-1 1h-5v-6h-6v6H4a1 1 0 0 1-1-1z"/></symbol>
  <symbol id="i-heart" viewBox="0 0 24 24"><path d="M19 14c1.49-1.46 3-3.21 3-5.5A5.5 5.5 0 0 0 16.5 3c-1.76 0-3 .5-4.5 2-1.5-1.5-2.74-2-4.5-2A5.5 5.5 0 0 0 2 8.5c0 2.3 1.5 4.05 3 5.5l7 7Z"/></symbol>
  <symbol id="i-plus" viewBox="0 0 24 24"><path d="M12 5v14M5 12h14"/></symbol>
  <symbol id="i-minus" viewBox="0 0 24 24"><path d="M5 12h14"/></symbol>
  <symbol id="i-image" viewBox="0 0 24 24"><rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-5-5L5 21"/></symbol>
  <symbol id="i-settings" viewBox="0 0 24 24"><circle cx="12" cy="12" r="3"/><path d="M12 2v3M12 19v3M2 12h3M19 12h3M4.9 4.9 7 7M17 17l2.1 2.1M4.9 19.1 7 17M17 7l2.1-2.1"/></symbol>
  <symbol id="i-check" viewBox="0 0 24 24"><path d="M20 6 9 17l-5-5"/></symbol>
  <symbol id="i-menu" viewBox="0 0 24 24"><path d="M4 6h16M4 12h16M4 18h16"/></symbol>
  <symbol id="i-calendar" viewBox="0 0 24 24"><rect x="3" y="4" width="18" height="18" rx="2"/><path d="M16 2v4M8 2v4M3 10h18"/></symbol>
</svg>

<main class="board">

  <!-- SCREEN: Notifications -->
  <div class="frame">
    <span class="frame-label">01 · Notifications</span>
    <section class="screen" data-screen="notifications">
      <header class="appbar">
        <button class="icon-btn"><svg class="icon"><use href="#i-back"/></svg></button>
        <span class="title">Notifications</span>
        <button class="icon-btn outline"><svg class="icon"><use href="#i-settings"/></svg></button>
      </header>
      <span class="caption">Today</span>
      <div class="list-item">
        <span class="avatar"></span>
        <div class="stack grow"><span class="sk w-50"></span><span class="sk sm w-85"></span><span class="sk sm w-70"></span></div>
      </div>
      <!-- repeat list items… -->
    </section>
  </div>

</main>
</body>
</html>
```

Add icons in the same outline style (24-unit viewBox, stroke only) when a screen needs one that isn't in the sprite.

## Common patterns

Build these from the components above:

- **Image + text card**: `.img.ar-43` on top, then a `.stack` with a `.sk.lg` title and two `.sk.sm` lines. A heart `.icon-btn` sits in the top-right corner when relevant.
- **Product row (cart)**: a square `.img` about 96px wide on the left; on the right a title skeleton, a real "Price" label, a "Remove" `.link` and a `.stepper` aligned right.
- **Settings row**: a `.card` with `.row.between` holding a skeleton title, a skeleton subtitle and a `.toggle`.
- **Option row (radio)**: a `.card` with a real label ("Standard Delivery"), a `.radio` on the right and two short skeleton bars below.
- **Auth screen**: a `.logo` circle or pill, an `.h1` greeting, `.input` fields with real placeholders, a "Forgot password?" link aligned right, a block `.btn-primary`, an `.or` divider, `.btn-social` buttons with an icon inside, and a muted sign-in or sign-up line at the bottom.
- **Search bar**: an `.input.grow` with a search icon and the placeholder "Search here..", next to a square `.icon-btn` with the filter icon and a border.
- **Order summary**: `.summary-row`s with real labels ("Items", "Vat") and skeleton values aligned right, a `.divider`, a bold "Total" row, then a block `.btn-primary` CTA.
- **Desktop dashboard**: `.sidebar` with `.nav-item`s (icon + real label), `.topbar` with search, bell, avatar and a skeleton name, then `.content` with a toolbar row (date nav, filter `.btn-tertiary`, primary action) and a `.table` or a grid of `.card`s. Header cells are real text; cell values are mostly skeletons with a few real fixed labels.

## Final check

Before delivering, confirm:
- [ ] No color: every hex value is a gray (R = G = B).
- [ ] No device frame, status bar, notch, home indicator or browser chrome.
- [ ] No shadows, gradients, animations, transitions or hover styles.
- [ ] Titles, buttons, labels, navigation and section headers are real text; variable content is skeleton bars of varying widths.
- [ ] Every control has its true shape (toggle, radio, checkbox, stepper, chip, input, tab bar).
- [ ] Icons all come from the sprite, in one outline style.
- [ ] Styles live in classes and tokens, not inline; each screen has a labeled `<section>`.
- [ ] The file opens on its own and screens sit side by side in flow order on the gray canvas.
