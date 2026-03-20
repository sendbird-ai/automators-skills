---
name: delight-ai-slide-deck-creator
description: Use this skill when the user wants to create a delight.ai branded sales deck, pitch deck, or presentation. Always attach delight-logo-white.png as the logo asset.
version: 3.0.0
author: Yongchang Kang
---

# Delight.ai Slide Deck Creator

> **Attach this skill + `delight-logo-white.png` to Claude to auto-generate on-brand slide decks.**
> All outputs follow the official delight.ai design system extracted from the original On-Demand Slide Deck.

---

## ⚠️ Logo Usage — Mandatory

**The logo is delight.ai's most important brand asset. Never replace it with text.**

- Always use the `delight-logo-white.png` image file
- **Never fake the logo with text like "✱ delight.ai"** — the shape will be wrong
- Maintain original aspect ratio (1676:326 = 5.141:1) when resizing
- Color changes allowed; shape distortion is never allowed

**Logo loading code (required in every deck):**
```javascript
const fs = require("fs");
const logoPath = "/mnt/user-data/uploads/delight-logo-white.png";
const logoB64 = "image/png;base64," + fs.readFileSync(logoPath).toString("base64");

function addLogo(slide, x, y, h) {
  const w = h * (1676 / 326); // original ratio 5.141:1
  slide.addImage({ data: logoB64, x, y, w, h });
}
```

---

## 1. Overview

This skill generates delight.ai branded sales/marketing presentations using PptxGenJS. It encodes the exact design system from the official deck, ensuring every output matches brand standards.

**Supported verticals:** On-Demand, Retail, Fintech, Travel & Hospitality, Healthcare, B2B SaaS

**Standard sales deck structure:**

| # | Slide Type | Purpose |
|---|-----------|---------|
| 1 | Cover | Logo + industry title + hero image |
| 2 | Statistics | 2–3 pain point stats in white cards |
| 3 | Testimonial | Customer quote + photo + logo |
| 4 | Overview | Two-column: Customers vs Service Providers |
| 5–10 | Feature Detail | Left: tag + headline + bullets + KPI / Right: chat UI |
| Last | Back Cover | CTA + contact info |

---

## 2. Design System

### 2.1 Canvas

- **Layout**: `LAYOUT_16x9` (10" × 5.625")
- **Background**: `121212` (near-black) — never use pure black `000000`

### 2.2 Color Tokens

| Token | Hex | Usage |
|-------|-----|-------|
| `bg-primary` | `121212` | Slide background |
| `bg-card-white` | `FFFFFF` | Stat cards, testimonial cards |
| `bg-card-dark` | `2A2A2A` | Overview columns, chat UI container |
| `text-primary` | `FFFFFF` | Headlines, primary text on dark bg |
| `text-secondary` | `E0E0E0` | Bullet points, descriptions |
| `text-on-card` | `000000` | Text on white cards |
| `accent` | `F2FF66` | Feature tag pills, highlights (yellow-green) |

**Color rules:**
- No `#` prefix in hex codes (PptxGenJS requirement)
- Background is always `121212` — never `000000`
- Accent `F2FF66` is used sparingly for tags and highlights only
- Never encode opacity in 8-char hex → use `opacity` property

### 2.3 Typography

| Role | Font | Fallback |
|------|------|----------|
| Headlines (serif) | `Lora` | `Georgia` |
| Body (sans) | `Roboto` | `Arial` |
| Body medium | `Roboto Medium` | `Arial` |
| Body light | `Roboto Light` | `Arial` |

| Element | Font | Size | Weight | Color |
|---------|------|------|--------|-------|
| Cover title | Lora | 46pt | Regular | `FFFFFF` |
| Section headline | Lora | 34pt | Regular | `FFFFFF` |
| Content title | Lora | 30pt | Regular | `FFFFFF` |
| Back cover title | Lora | 44pt | Regular | `FFFFFF` |
| Stat number | Roboto | 40pt | Bold | `000000` |
| Stat description | Roboto Medium | 13pt | Regular | `555555` |
| Feature tag pill | Roboto Medium | 9pt | Regular | `F2FF66` |
| Bullet point | Roboto | 10.5–11pt | Regular | `E0E0E0` |
| Testimonial quote | Roboto Light | 18pt | Regular | `000000` |
| Header text | Roboto | 9pt | Regular | `E0E0E0` |

### 2.4 Layout Grid

**Margins (all content must stay within these — nothing outside):**

```javascript
const M = { left: 0.50, right: 9.50, top: 0.50, bottom: 5.125, contentW: 9.00 };
```

- All four margins are **0.50"** (US standard half-inch)
- Bottom safe zone at 5.125" (0.50" from slide bottom edge)

**Logo sizes:**
- Cover / Back Cover: `h = 0.38"` (~2.4× header size)
- Content slide header: `h = 0.16"`

**Header bar** (content slides):
- Logo → vertical separator (same height as logo) → tagline → copyright (right-aligned)

**Title line spacing**: `lineSpacingMultiple: 1.0` — mandatory on all Lora headlines

**Tight text box rule**: Always calculate text box height with `tH()` function so the box wraps the actual text, not larger. This ensures visual spacing between elements is accurate.

```javascript
function tH(fontSize, lines) {
  const lineH = fontSize / 72;
  return Math.round((lineH * lines + lineH * 0.20) * 100) / 100;
}
```

**Content slide vertical rhythm:**
- Header: y = 0.50 (M.top)
- Feature tag: y = 0.85
- Title: y = 1.31
- Cards: y = title bottom + 0.25" gap

### 2.5 Design Rules (Non-Negotiable)

1. **Logo must be PNG image asset** — never fake with text, ratio 5.141:1
2. **Cover/back cover logo h=0.38"**, header logo h=0.16"
3. **All content inside margins** — left 0.50", right 9.50", top 0.50", bottom 5.125"
4. **Title line spacing `lineSpacingMultiple: 1.0`** — mandatory on all Lora headlines
5. **Header separator line height = logo height** — must match exactly
6. **Title font is Lora Regular** — no bold (preserves serif elegance)
7. Background is always `121212`
8. Text primary `FFFFFF`, secondary `E0E0E0`, accent `F2FF66`
9. Feature tag: `F2FF66` text + 1pt border, transparent fill
10. **No accent lines under titles** — hallmark of AI-generated slides
11. Never reuse PptxGenJS option objects (internal mutation issue)
12. Use `bullet: true`, never unicode "•" (creates double bullets)
13. **Maximize whitespace** — simple, clean, breathing room
14. **Title-to-card gap**: minimum 0.25" between title bottom and first card
15. **Back cover = same logo size as cover**, lineSpacing 1.0
16. **Text box height = `tH(fontSize, lines)`** — prevents visual spacing mismatch

---

## 3. PptxGenJS Code Templates

### 3.0 Setup & Helpers

```javascript
const pptxgen = require("pptxgenjs");
const fs = require("fs");
const pres = new pptxgen();
pres.layout = "LAYOUT_16x9";
pres.author = "delight.ai";

// ── Load logo (required) ──
const logoB64 = "image/png;base64," + fs.readFileSync("delight-logo-white.png").toString("base64");
const LOGO_RATIO = 1676 / 326; // 5.141:1

// ── Design tokens ──
const C = {
  bg: "121212", white: "FFFFFF", textPrimary: "FFFFFF",
  textSecondary: "E0E0E0", textDark: "000000",
  accent: "F2FF66", cardWhite: "FFFFFF", cardDark: "2A2A2A",
};
const F = {
  headline: "Lora",    // title font — Lora only
  body: "Arial", bodyMed: "Arial", bodyLight: "Arial",
};

// ── Margins (all content inside these bounds) ──
const M = { left: 0.50, right: 9.50, top: 0.50, bottom: 5.125, contentW: 9.00 };
const COVER_LOGO_H = 0.38;
const HDR_LOGO_H = 0.16;

// ── Tight text height (includes descender space) ──
function tH(fontSize, lines) {
  const lineH = fontSize / 72;
  return Math.round((lineH * lines + lineH * 0.20) * 100) / 100;
}

// ── Logo (auto-preserves aspect ratio) ──
function addLogo(slide, x, y, h) {
  slide.addImage({ data: logoB64, x, y, w: h * LOGO_RATIO, h });
}

// ── Header (logo + separator of equal height) ──
function addHeader(slide) {
  const logoW = HDR_LOGO_H * LOGO_RATIO;
  addLogo(slide, M.left, M.top, HDR_LOGO_H);
  const lineX = M.left + logoW + 0.08;
  slide.addShape(pres.shapes.LINE, {
    x: lineX, y: M.top, w: 0, h: HDR_LOGO_H,
    line: { color: "666666", width: 0.5 }
  });
  slide.addText("Your branded AI concierge", {
    x: lineX + 0.10, y: M.top, w: 3, h: HDR_LOGO_H,
    fontFace: F.body, fontSize: 9, color: C.textSecondary,
    align: "left", valign: "middle", margin: 0
  });
  slide.addText("© 2026 Sendbird, Inc", {
    x: M.right - 2.2, y: M.top, w: 2.2, h: HDR_LOGO_H,
    fontFace: F.body, fontSize: 9, color: C.textSecondary,
    align: "right", valign: "middle", margin: 0
  });
}

// ── Feature tag pill ──
function addFeatureTag(slide, text) {
  const y = 0.85, h = 0.26;
  const w = Math.max(text.length * 0.075 + 0.5, 1.4);
  slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
    x: M.left, y, w, h,
    fill: { color: C.bg }, line: { color: C.accent, width: 1 }, rectRadius: 0.05
  });
  slide.addText(text, {
    x: M.left, y, w, h,
    fontFace: F.bodyMed, fontSize: 9, color: C.accent,
    align: "center", valign: "middle", margin: 0
  });
  return y + h;
}

// ── KPI badges ──
function addKPIBadges(slide, badges) {
  slide.addShape(pres.shapes.LINE, {
    x: M.left, y: 4.70, w: 3.38, h: 0, line: { color: C.white, width: 0.5 }
  });
  let xOff = M.left;
  badges.forEach(b => {
    const arrow = b.direction === "up" ? "↑" : "↓";
    slide.addText(`${arrow} ${b.label}`, {
      x: xOff, y: 4.88, w: 2.0, h: 0.20,
      fontFace: F.bodyMed, fontSize: 11, color: C.white,
      align: "left", valign: "middle", margin: 0
    });
    xOff += 1.8;
  });
}

// ── Right zone container ──
function addRightZoneContainer(slide) {
  slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
    x: 4.50, y: M.top, w: 5.00, h: 4.65,
    fill: { color: C.cardDark }, rectRadius: 0.15
  });
}

// ── Layout constants ──
const TITLE_Y = 1.31;
const TITLE_GAP = 0.25;
```

### 3.1 Cover Slide

```javascript
function createCoverSlide(title, heroImagePath) {
  const slide = pres.addSlide();
  slide.background = { color: C.bg };
  addLogo(slide, M.left, M.top, COVER_LOGO_H);

  const titleY = M.top + COVER_LOGO_H + 0.22;
  const titleH = tH(46, 2);
  slide.addText(title, {
    x: M.left, y: titleY, w: M.contentW, h: titleH,
    fontFace: F.headline, fontSize: 46, color: C.white,
    align: "left", valign: "top", margin: 0, lineSpacingMultiple: 1.0
  });

  const cardY = titleY + titleH + 0.20;
  if (heroImagePath) {
    slide.addImage({ path: heroImagePath, x: M.left, y: cardY, w: M.contentW, h: M.bottom - cardY,
      sizing: { type: "cover", w: M.contentW, h: M.bottom - cardY } });
  } else {
    slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
      x: M.left, y: cardY, w: M.contentW, h: M.bottom - cardY,
      fill: { color: C.cardDark }, rectRadius: 0.10
    });
  }
  return slide;
}
```

### 3.2 Statistics Slide

```javascript
function createStatisticsSlide(tagText, title, stats) {
  const slide = pres.addSlide();
  slide.background = { color: C.bg };
  addHeader(slide);
  addFeatureTag(slide, tagText);

  const titleH = tH(30, 2);
  slide.addText(title, {
    x: M.left, y: TITLE_Y, w: M.contentW, h: titleH,
    fontFace: F.headline, fontSize: 30, color: C.white,
    align: "left", valign: "top", margin: 0, lineSpacingMultiple: 1.0
  });

  const cardY = TITLE_Y + titleH + TITLE_GAP;
  const cardW = (M.contentW - 0.20) / 2;
  const cardH = 1.20;
  stats.forEach((stat, i) => {
    const col = i % 2, row = Math.floor(i / 2);
    const x = M.left + col * (cardW + 0.20);
    const y = cardY + row * (cardH + 0.18);
    slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
      x, y, w: cardW, h: cardH, fill: { color: C.cardWhite }, rectRadius: 0.08
    });
    slide.addText(stat.number, {
      x: x + 0.20, y: y + 0.08, w: 2.0, h: cardH - 0.16,
      fontFace: F.body, fontSize: 40, bold: true, color: C.textDark,
      align: "left", valign: "middle", margin: 0
    });
    slide.addText(stat.label, {
      x: x + 2.10, y: y + 0.08, w: cardW - 2.40, h: cardH - 0.16,
      fontFace: F.bodyMed, fontSize: 13, color: "555555",
      align: "left", valign: "middle", margin: 0
    });
  });
  return slide;
}
```

### 3.3 Testimonial Slide

```javascript
function createTestimonialSlide(headline, quote, authorName, authorTitle, photoPath) {
  const slide = pres.addSlide();
  slide.background = { color: C.bg };
  addHeader(slide);

  const titleH = tH(34, 1);
  slide.addText(headline, {
    x: M.left, y: 0.85, w: M.contentW, h: titleH,
    fontFace: F.headline, fontSize: 34, color: C.white,
    align: "left", valign: "top", margin: 0, lineSpacingMultiple: 1.0
  });

  const cardY = 0.85 + titleH + TITLE_GAP;
  const cardH = M.bottom - cardY;
  slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
    x: M.left, y: cardY, w: M.contentW, h: cardH,
    fill: { color: C.cardWhite }, rectRadius: 0.10
  });
  slide.addText(`\u201C${quote}\u201D`, {
    x: M.left + 0.30, y: cardY + 0.30, w: 4.50, h: cardH - 0.80,
    fontFace: F.bodyLight, fontSize: 16, color: C.textDark,
    align: "left", valign: "top", margin: 0, lineSpacingMultiple: 1.4
  });
  slide.addText([
    { text: authorName, options: { fontFace: F.body, fontSize: 10, bold: true, color: C.textDark } },
    { text: `, ${authorTitle}`, options: { fontFace: F.body, fontSize: 10, color: C.textDark } }
  ], { x: M.left + 0.30, y: cardY + cardH - 0.50, w: 4.50, h: tH(10, 1), margin: 0 });

  if (photoPath) {
    slide.addShape(pres.shapes.LINE, {
      x: 5.40, y: cardY + 0.25, w: 0, h: cardH - 0.50, line: { color: "CCCCCC", width: 1 }
    });
    slide.addImage({ path: photoPath, x: 5.70, y: cardY + 0.25, w: 3.50, h: cardH - 0.50,
      sizing: { type: "cover", w: 3.50, h: cardH - 0.50 } });
  }
  return slide;
}
```

### 3.4 Feature Detail Slide

```javascript
function createFeatureSlide(feature) {
  /* feature = {
    tag: "Proactive Delay Management",
    headline: "Turn delays into trust-building moments",
    bullets: ["Detect supplier delays in real-time", ...],
    kpis: [{ direction: "up", label: "Customer retention" }, ...],
    chatImage: null,
    chatMockup: null
  } */
  const slide = pres.addSlide();
  slide.background = { color: C.bg };
  addHeader(slide);
  addFeatureTag(slide, feature.tag);

  const titleH = tH(30, 2);
  slide.addText(feature.headline, {
    x: M.left, y: TITLE_Y, w: 3.6, h: titleH,
    fontFace: F.headline, fontSize: 30, color: C.white,
    align: "left", valign: "top", margin: 0, lineSpacingMultiple: 1.0
  });

  const bulletY = TITLE_Y + titleH + 0.15;
  const bullets = feature.bullets.map((b, i) => ({
    text: b, options: { fontFace: F.body, fontSize: 11, color: C.textSecondary,
      bullet: true, breakLine: i < feature.bullets.length - 1 }
  }));
  slide.addText(bullets, { x: M.left, y: bulletY, w: 3.19, h: 1.70, margin: 0, paraSpaceAfter: 8 });
  addKPIBadges(slide, feature.kpis);

  // Right zone
  addRightZoneContainer(slide);
  if (feature.chatImage) {
    slide.addImage({ path: feature.chatImage, x: 5.20, y: 0.80, w: 3.12, h: 4.30,
      sizing: { type: "contain", w: 3.12, h: 4.30 } });
  } else {
    slide.addText("[Chat UI Mockup]", {
      x: 4.50, y: M.top, w: 5.00, h: 4.65,
      fontFace: F.body, fontSize: 14, color: C.textSecondary, align: "center", valign: "middle"
    });
  }
  return slide;
}
```

### 3.5 Back Cover (CTA) Slide

```javascript
function createBackCover(title, contactUrl) {
  const slide = pres.addSlide();
  slide.background = { color: C.bg };
  addLogo(slide, M.left, M.top, COVER_LOGO_H);

  const titleY = M.top + COVER_LOGO_H + 0.35;
  const titleH = tH(44, 3);
  slide.addText(title, {
    x: M.left, y: titleY, w: M.contentW, h: titleH,
    fontFace: F.headline, fontSize: 44, color: C.white,
    align: "left", valign: "top", margin: 0, lineSpacingMultiple: 1.0
  });

  const btnY = titleY + titleH + 0.35;
  slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
    x: M.left, y: btnY, w: 2.40, h: 0.45,
    fill: { color: C.accent }, rectRadius: 0.06
  });
  slide.addText("Contact Sales →", {
    x: M.left, y: btnY, w: 2.40, h: 0.45,
    fontFace: F.bodyMed, fontSize: 14, bold: true, color: C.textDark,
    align: "center", valign: "middle", margin: 0
  });

  const infoY = btnY + 0.45 + 0.40;
  slide.addText(contactUrl || "sendbird.com/contact-sales  |  delight.ai", {
    x: M.left, y: infoY, w: 5, h: tH(11, 1),
    fontFace: F.body, fontSize: 11, color: C.textSecondary,
    align: "left", valign: "top", margin: 0
  });

  slide.addText("© 2026 Sendbird, Inc. All rights reserved.", {
    x: M.left, y: M.bottom - tH(9, 1), w: M.contentW, h: tH(9, 1),
    fontFace: F.body, fontSize: 9, color: "555555",
    align: "left", valign: "top", margin: 0
  });
  return slide;
}
```

### 3.6 Full Deck Assembly Example

```javascript
createCoverSlide("The on-demand shift", null);

createStatisticsSlide("Industry Pain Points",
  "Support gaps erode\nmarketplace trust", [
  { number: "73%", label: "of providers switch platforms over support issues" },
  { number: "51%", label: "of customers stop spending after one bad experience" },
]);

createTestimonialSlide(
  "Great support keeps transactions moving.",
  "delight.ai has really helped our support speed in that we can now serve our tradespeople 24/7.",
  "Jeremy Burton", "CTO of hipages", null
);

createFeatureSlide({
  tag: "Proactive Delay Management",
  headline: "Turn delays into\ntrust-building moments",
  bullets: [
    "Detect supplier delays in real-time",
    "Coordinate communication across all parties",
    "Offer intelligent alternatives and compensation",
    "Keep providers productive during wait times"
  ],
  kpis: [
    { direction: "up", label: "Customer retention" },
    { direction: "down", label: "Provider downtime" }
  ]
});

createBackCover("Ready to unleash\nthe power of AI\ncustomer service?");

pres.writeFile({ fileName: "delight-ai-deck.pptx" });
```

---

## 4. Industry Vertical Content

### On-Demand (Delivery, Rideshare, Home Services)
- **Stats**: 73% provider churn, 51% customer spending reduction
- **Features**: Delay Management, Payment Recovery, Provider Onboarding, Earnings Support, Active Job Support, Dispute Protection
- **KPIs**: Customer retention, Provider downtime, Revenue recovery, Provider activation

### Retail (E-commerce, Omnichannel)
- **Stats**: 70–80% cart abandonment, 2–3% conversion rate, 16.5% return rate
- **Features**: Product Discovery, Order Tracking, Return/Exchange, Recommendations, Post-Purchase, Loyalty
- **KPIs**: Cart recovery, CSAT, Repeat purchase, Return reduction

### Fintech (Payments, Banking, Insurance)
- **Stats**: 67% onboarding drop-off, 40% tickets = transaction issues, $3.7M avg annual cost
- **Features**: Transaction Support, Fraud Alert, Claim Processing, Account Onboarding, Payment Troubleshooting, Compliance
- **KPIs**: Resolution speed, Customer retention, Cost per interaction

### Travel & Hospitality
- **Stats**: 81% expect instant responses, 39% abandon complex bookings, 73% rebook with personalization
- **Features**: Booking Assistance, Trip Disruption, Concierge, Loyalty Recognition, Post-Stay, Group Coordination
- **KPIs**: Booking completion, Guest satisfaction, Rebooking rate

### Healthcare
- **Stats**: 23% no-show rate, care coordination delays, patient engagement gaps
- **Features**: Appointment Management, Symptom Triage, Care Coordination, Medication Reminders, Insurance Nav, Follow-Up
- **KPIs**: Appointment adherence, Patient satisfaction, Care gap closure

### B2B SaaS
- **Stats**: Onboarding drop-off, time-to-value delays, support ticket volume
- **Features**: Onboarding Guidance, Feature Adoption, Technical Support, Account Health, Renewal, Upsell
- **KPIs**: Time to value, Feature adoption, Retention rate, NPS

---

## 5. Prompt Templates

### Full custom deck

```
Using the delight.ai slide deck skill and the attached logo file,
create a [INDUSTRY] vertical sales deck in PptxGenJS.

Customer: [COMPANY NAME]
Industry: [On-Demand / Retail / Fintech / Travel / Healthcare / B2B SaaS]
Date: [YYYY-MM-DD]

Pain point stats (Slide 2):
1. [XX]% — [description]
2. [XX]% — [description]

Testimonial (Slide 3):
- Headline: [one-line summary]
- Quote: "[customer quote]"
- Attribution: [Name], [Title] of [Company]

Features to highlight (Slides 5–10):
1. [Feature] — [one-line description]
2. [Feature] — [one-line description]
3. [Feature] — [one-line description]

Chat UI: Generate simple mockups with code
```

### Quick swap (keep structure, change content)

```
Using the delight.ai slide deck skill, keep the standard On-Demand deck structure
but swap these details:

- Customer: [NAME]
- Stat 1: [XX]% — [description]
- Stat 2: [XX]% — [description]
- Testimonial: "[quote]" — [Name], [Title]
- Date: [YYYY-MM-DD]
```

---

## 6. Image Assets Checklist

| Asset | Format | Used In | Required |
|-------|--------|---------|----------|
| **delight-logo-white.png** | **PNG (1676×326, RGBA)** | **Header, cover, cards** | **Yes** |
| Customer logo | PNG/SVG | Cover, header | Optional |
| Hero photo | JPG (min 1920px) | Cover bottom | Optional |
| Testimonial photo | JPG | Slide 3 right | Optional |
| Chat UI screenshots | PNG (~750×1100px) | Feature slides right | Optional |

**Logo specs:**
- Filename: `delight-logo-white.png`
- Dimensions: 1676 × 326px (ratio 5.141:1, trimmed — no transparent padding)
- Mode: RGBA (white wordmark on transparent background)
- Extracted from official PPTX asset
- Use on dark backgrounds only

---

## 7. QA Process

After generating a deck, always run:

```bash
# 1. Text check
python -m markitdown output.pptx

# 2. Visual check — convert to images
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
rm -f slide-*.jpg
pdftoppm -jpeg -r 150 output.pdf slide
ls -1 "$PWD"/slide-*.jpg
```

**Check for**: Element overlap, text overflow, uneven spacing, low-contrast text, leftover placeholders, text boxes larger than actual text.
