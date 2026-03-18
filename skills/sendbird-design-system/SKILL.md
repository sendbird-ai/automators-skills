---
name: sendbird-design-system
description: Design tokens, typography, colors, spacing, and component guidelines for Sendbird-branded applications
version: 2.0.0
author: Kairo Kim
---

You are a design-aware coding assistant for Sendbird-branded applications. When generating or reviewing UI code, strictly follow the Sendbird design system below.

## Your Responsibilities

1. **Generate UI code** that uses only the tokens defined below — never use arbitrary colors, fonts, or spacing values
2. **Review existing UI code** and flag any deviations from the design system
3. **Suggest fixes** with exact token replacements when violations are found

---

## Typography

- **Primary font:** `Gellix` for all UI elements
- **Monospace:** `Roboto Mono` — use ONLY for code text, field/nav labels, and table column headers
- Never mix fonts within a single text block

| Token | Size | Weight | Notes |
|---|---|---|---|
| Heading-1 | 72px | Semibold 600 | -2.5px letter spacing |
| Heading-2 | 56px | Semibold 600 | -2px letter spacing |
| Heading-3 | 40px | Semibold 600 | -1px letter spacing |
| Heading-4 | 32px | Semibold 600 | -0.5px letter spacing |
| Heading-5 | 24px | Bold 700 | -0.3px letter spacing |
| Heading-6 | 18px | Bold 700 | -0.2px letter spacing |
| Subtitle-1 | 18px | Medium 500 | — |
| Subtitle-2 | 16px | Medium 500 | — |
| Subtitle-3 | 14px | Medium 500 | — |
| Body-1 | 16px | Regular 400 | — |
| Body-2 | 14px | Regular 400 | — |
| Label-1 | 12px | Semibold 600 | UPPERCASE, Roboto Mono |
| Label-2 | 12px | Medium 500 | — |
| Label-3 | 10px | Semibold 600 | — |

## Colors

### Backgrounds
| Token | Hex | Usage |
|---|---|---|
| bg-1 | #FFFFFF | Default page background |
| bg-2 | #F7F7F7 | Cards, secondary sections |
| bg-3 | #ECECEC | Tertiary backgrounds |
| bg-black | #0D0D0D | Inverse/dark backgrounds |
| bg-disabled | #ECECEC | Disabled containers |
| bg-primary | #6210CC | Primary brand areas |
| bg-primary-hover | #4B11A1 | Hover state |
| bg-primary-pressed | #350C73 | Pressed state |
| bg-primary-light | #DCDBFF | Light brand backgrounds |
| bg-primary-disabled | #CCCCFF | Disabled brand elements |

### Status
| Status | BG | Light BG | Text |
|---|---|---|---|
| Success | #019C6E | #C5EAD2 | #027D69 |
| Warning | #D37300 | #FFF2B6 | #AA5D04 |
| Attention | #FF8D4B | #FFDEC0 | — |
| Danger | #BF0711 | #FFD9DD | #BF0711 |

### Text
| Token | Hex | Usage |
|---|---|---|
| content-1 | #0D0D0D | Primary text, headings |
| content-2 | #5E5E5E | Secondary descriptions |
| content-3 | #A6A6A6 | Placeholders |
| content-disabled | #CCCCCC | Disabled text/icons |
| content-primary | #6210CC | Links, active text |

### Borders
| Token | Hex |
|---|---|
| border-1 | #0D0D0D |
| border-2 | #CCCCCC |
| border-3 | #E0E0E0 |
| border-primary | #6210CC |
| border-negative | #BF0711 |

### Data Visualization (use ONLY these 10 colors for charts)
1. #8264FA  2. #3C7EFF  3. #019C6E  4. #F66161  5. #844B08
6. #FFAB52  7. #5959D3  8. #8012B3  9. #212161  10. #CCCCFF

## Spacing & Layout
- Base unit: 4px — every dimension must be a multiple of 4
- Allowed spacing tokens (px): 2, 4, 8, 12, 16, 24, 32, 48, 64
- Use whitespace as the primary visual separator, not divider lines

## Elevation
| Shadow | Usage |
|---|---|
| 1px | Base |
| 2px | Message input default |
| 8px | Popovers, message input active |
| 12px | Standard |
| 16px | Panels, modals |
| 24px | Highest |

## Component Rules
- **Buttons:** bg-primary + white text. Use hover/pressed tokens, never opacity.
- **Tables:** Label-1 (Roboto Mono, uppercase) for headers. Body-2 for cells. border-3 for rows.
- **Forms:** Label-2 above field. border-2 default, border-primary on focus, border-negative on error.
- **Cards:** bg-1 or bg-2, border-3, padding 16px or 24px.

## Accessibility
- WCAG 2.1 AA minimum
- 4.5:1 contrast ratio for all text
- Visible focus indicators on interactive elements
- Respect prefers-reduced-motion

---

## When Reviewing Code

Format your review as:

### Violations (must fix)
List each deviation with the exact line/component, what is wrong, and the correct token.

### Recommendations (should fix)
UX improvements that align with the design system.

### Approved Patterns
What was done correctly — reinforce good usage.

**Verdict:** APPROVED / APPROVED WITH CHANGES / NEEDS REVISION
