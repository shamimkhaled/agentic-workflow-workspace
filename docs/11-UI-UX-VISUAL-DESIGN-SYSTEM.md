# FlowSpark AI - Visual Design System
## Colors, Typography, Components & Design Tokens

---

## 1. COLOR PALETTE

### 1.1 Primary Colors

**Brand Primary:**
- Primary 50: `#E6F2FF` (lightest background)
- Primary 100: `#CCE5FF`
- Primary 200: `#99CBFF`
- Primary 300: `#66B1FF`
- Primary 400: `#3397FF`
- Primary 500: `#0066CC` (main brand color)
- Primary 600: `#0052A3`
- Primary 700: `#003D7A`
- Primary 800: `#002952`
- Primary 900: `#001429` (darkest)

**Usage:**
- Primary 500: Main CTAs, links, active states
- Primary 100-200: Light backgrounds, hover states
- Primary 700-900: Text on light backgrounds (when needed)

### 1.2 Secondary Colors

**Accent Colors:**
- Success (Green): `#10B981` (500), `#D1FAE5` (100)
- Warning (Amber): `#F59E0B` (500), `#FEF3C7` (100)
- Error (Red): `#EF4444` (500), `#FEE2E2` (100)
- Info (Blue): `#3B82F6` (500), `#DBEAFE` (100)

### 1.3 Neutral Colors

**Grayscale:**
- Gray 50: `#F9FAFB` (lightest background)
- Gray 100: `#F3F4F6`
- Gray 200: `#E5E7EB` (borders, dividers)
- Gray 300: `#D1D5DB`
- Gray 400: `#9CA3AF` (placeholder text)
- Gray 500: `#6B7280` (secondary text)
- Gray 600: `#4B5563` (body text)
- Gray 700: `#374151` (headings)
- Gray 800: `#1F2937` (primary text)
- Gray 900: `#111827` (darkest text)

**Usage:**
- Gray 50-100: Backgrounds
- Gray 200-300: Borders, dividers
- Gray 400-500: Secondary text, icons
- Gray 600-800: Primary text
- Gray 900: High contrast text

### 1.4 Semantic Colors

**Document Types:**
- Word: `#2B579A` (Microsoft Blue)
- PDF: `#F40F02` (Adobe Red)
- Excel: `#217346` (Microsoft Green)
- PowerPoint: `#D04423` (Microsoft Orange)

**Status Colors:**
- Processing: `#3B82F6` (Blue)
- Completed: `#10B981` (Green)
- Failed: `#EF4444` (Red)
- Pending: `#6B7280` (Gray)

---

## 2. TYPOGRAPHY

### 2.1 Font Family

**Primary Font:** Inter (system fallback: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif)

**Rationale:**
- Excellent readability at all sizes
- Professional appearance
- Wide language support
- Web-safe with system fallbacks

### 2.2 Type Scale

**Heading Hierarchy:**
- H1: `48px / 3rem` | `600 weight` | `1.2 line-height`
- H2: `36px / 2.25rem` | `600 weight` | `1.3 line-height`
- H3: `30px / 1.875rem` | `600 weight` | `1.4 line-height`
- H4: `24px / 1.5rem` | `600 weight` | `1.4 line-height`
- H5: `20px / 1.25rem` | `600 weight` | `1.5 line-height`
- H6: `18px / 1.125rem` | `600 weight` | `1.5 line-height`

**Body Text:**
- Body Large: `18px / 1.125rem` | `400 weight` | `1.6 line-height`
- Body: `16px / 1rem` | `400 weight` | `1.6 line-height`
- Body Small: `14px / 0.875rem` | `400 weight` | `1.5 line-height`
- Caption: `12px / 0.75rem` | `400 weight` | `1.4 line-height`

**Special Text:**
- Label: `14px / 0.875rem` | `500 weight` | `1.4 line-height`
- Helper Text: `12px / 0.75rem` | `400 weight` | `1.4 line-height`
- Code: `14px / 0.875rem` | `400 weight` | `1.5 line-height` | Monospace font

### 2.3 Typography Usage

**Headings:**
- H1: Landing page hero, main page titles
- H2: Section headers, modal titles
- H3: Subsection headers
- H4: Card titles, smaller section headers
- H5-H6: Minor headings, table headers

**Body:**
- Body Large: Important descriptions, intro text
- Body: Default text, paragraphs, descriptions
- Body Small: Secondary text, metadata
- Caption: Timestamps, labels, fine print

---

## 3. SPACING SYSTEM

### 3.1 Base Unit

**8px base unit** (recommended for consistency)

### 3.2 Spacing Scale

- `4px` (0.25rem) - Tight spacing, icon padding
- `8px` (0.5rem) - Small spacing, compact elements
- `12px` (0.75rem) - Default padding for small components
- `16px` (1rem) - Default spacing, card padding
- `24px` (1.5rem) - Medium spacing, section gaps
- `32px` (2rem) - Large spacing, major sections
- `48px` (3rem) - Extra large spacing, page sections
- `64px` (4rem) - Huge spacing, hero sections

### 3.3 Component Spacing

**Padding:**
- Small components: `8px 12px`
- Medium components: `12px 16px`
- Large components: `16px 24px`
- Cards: `24px`
- Modals: `32px`

**Margins:**
- Between related elements: `8-16px`
- Between sections: `32-48px`
- Page margins: `24px` (mobile), `48px` (desktop)

---

## 4. SHADOWS & ELEVATION

### 4.1 Shadow Scale

**Elevation Levels:**
- Level 1 (Cards): `0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24)`
- Level 2 (Hover): `0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23)`
- Level 3 (Modals): `0 10px 20px rgba(0, 0, 0, 0.19), 0 6px 6px rgba(0, 0, 0, 0.23)`
- Level 4 (Dropdowns): `0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22)`
- Level 5 (Tooltips): `0 19px 38px rgba(0, 0, 0, 0.30), 0 15px 12px rgba(0, 0, 0, 0.22)`

**Usage:**
- Level 1: Default card state
- Level 2: Card hover, buttons hover
- Level 3: Modals, drawers
- Level 4: Dropdowns, popovers
- Level 5: Tooltips (if needed)

---

## 5. BORDER RADIUS

### 5.1 Radius Scale

- `2px` - Small elements (badges, tags)
- `4px` - Buttons, inputs, small cards
- `8px` - Cards, modals (default)
- `12px` - Large cards, hero sections
- `16px` - Round elements, avatars (circular: 50%)

### 5.2 Usage

**Components:**
- Buttons: `4px` (small), `6px` (medium), `8px` (large)
- Inputs: `4px`
- Cards: `8px`
- Modals: `8px` (top corners)
- Badges: `12px` (pill shape)
- Avatars: `50%` (circle)

---

## 6. COMPONENT LIBRARY

### 6.1 Buttons

**Primary Button:**
- Background: Primary 500 (`#0066CC`)
- Text: White
- Padding: `12px 24px`
- Border: None
- Border Radius: `6px`
- Font: `14px, 500 weight`
- Hover: Primary 600, shadow level 2
- Active: Primary 700
- Disabled: Gray 300, gray text, cursor not-allowed

**Secondary Button:**
- Background: Transparent
- Text: Primary 500
- Border: 1px solid Primary 500
- Padding: `12px 24px`
- Hover: Primary 50 background
- Active: Primary 100 background

**Tertiary Button:**
- Background: Transparent
- Text: Gray 700
- Border: None
- Padding: `8px 16px`
- Hover: Gray 100 background

**Destructive Button:**
- Background: Error 500 (`#EF4444`)
- Text: White
- Same padding/styling as primary
- Confirmation required for destructive actions

**Button Sizes:**
- Small: `8px 16px`, `12px font`
- Medium: `12px 24px`, `14px font` (default)
- Large: `16px 32px`, `16px font`

### 6.2 Form Inputs

**Text Input:**
- Background: White
- Border: 1px solid Gray 300
- Border Radius: `4px`
- Padding: `12px 16px`
- Font: `14px, 400 weight`
- Focus: 2px solid Primary 500, Primary 100 shadow
- Error: Red border, Error 100 background
- Disabled: Gray 100 background, Gray 400 text

**Textarea:**
- Same as text input
- Min height: `100px`
- Resizable: Vertical only

**Select Dropdown:**
- Same styling as text input
- Dropdown arrow icon (right)
- Options: Hover state (Primary 50)

**Checkbox:**
- Size: `20px x 20px`
- Border: 2px solid Gray 300
- Border Radius: `4px`
- Checked: Primary 500 background, white checkmark
- Focus: Primary 100 outline

**Radio Button:**
- Size: `20px x 20px`
- Border: 2px solid Gray 300
- Border Radius: `50%` (circle)
- Checked: Primary 500 inner circle (8px)

**Switch Toggle:**
- Width: `44px`, Height: `24px`
- Border Radius: `12px` (pill)
- Background: Gray 300 (off), Primary 500 (on)
- Thumb: `20px circle`, white
- Transition: `0.2s ease`

### 6.3 Cards

**Default Card:**
- Background: White
- Border: 1px solid Gray 200
- Border Radius: `8px`
- Padding: `24px`
- Shadow: Level 1
- Hover: Shadow level 2, slight scale (1.02)

**Card Variants:**
- Compact: `16px padding`
- Spacious: `32px padding`
- Bordered: Emphasized border (Gray 300)
- Elevated: Shadow level 2 (default)

### 6.4 Badges & Tags

**Badge:**
- Background: Gray 100
- Text: Gray 700
- Padding: `4px 8px`
- Border Radius: `12px` (pill)
- Font: `12px, 500 weight`

**Badge Variants:**
- Success: Success 100 background, Success 700 text
- Warning: Warning 100 background, Warning 700 text
- Error: Error 100 background, Error 700 text
- Info: Info 100 background, Info 700 text
- Primary: Primary 100 background, Primary 700 text

**Tag:**
- Similar to badge
- Removable: X icon on right
- Interactive: Hover state

### 6.5 Modals & Overlays

**Modal:**
- Backdrop: Black 50% opacity (rgba(0, 0, 0, 0.5))
- Container: Centered, max-width `600px` (small), `800px` (medium), `1200px` (large)
- Background: White
- Border Radius: `8px`
- Shadow: Level 3
- Padding: `32px`

**Modal Header:**
- Title: H2, 600 weight
- Close button: X icon, top-right
- Divider: Bottom border (Gray 200)

**Modal Footer:**
- Actions: Right-aligned buttons
- Spacing: `16px` between buttons

**Drawer (Mobile/Side):**
- Slides from edge
- Width: `320px` (mobile), `400px` (desktop)
- Same styling as modal

### 6.6 Tables

**Table:**
- Width: 100%
- Border: 1px solid Gray 200 (top)
- Background: White

**Table Header:**
- Background: Gray 50
- Text: Gray 700, 600 weight, 14px
- Padding: `12px 16px`
- Border: 1px solid Gray 200 (bottom)

**Table Row:**
- Padding: `12px 16px`
- Border: 1px solid Gray 200 (bottom)
- Hover: Gray 50 background
- Selected: Primary 50 background

**Table Cell:**
- Padding: `12px 16px`
- Text: Gray 800, 14px

### 6.7 Loading States

**Spinner:**
- Size: `20px` (small), `40px` (medium), `60px` (large)
- Color: Primary 500
- Animation: Rotating 360deg, linear, infinite

**Skeleton Loader:**
- Background: Gray 200
- Animation: Shimmer effect (left to right gradient)
- Border Radius: `4px`
- Heights: Match content (text: 16px, card: 200px, etc.)

**Progress Bar:**
- Container: Gray 200 background, `8px` height, `100%` width
- Fill: Primary 500, `8px` height
- Border Radius: `4px` (pill)
- Animation: Smooth width transition

### 6.8 Tooltips

**Tooltip:**
- Background: Gray 800
- Text: White, 12px
- Padding: `8px 12px`
- Border Radius: `4px`
- Shadow: Level 5
- Arrow: Small triangle pointing to trigger
- Max width: `200px`

**Positioning:**
- Top (default)
- Bottom
- Left
- Right

---

## 7. ICON SYSTEM

### 7.1 Icon Library

**Recommended:** Heroicons (outline and solid variants)

**Sizes:**
- Small: `16px`
- Medium: `20px` (default)
- Large: `24px`
- XLarge: `32px`

**Colors:**
- Default: Gray 600
- Primary: Primary 500
- Success: Success 500
- Error: Error 500
- Disabled: Gray 400

### 7.2 Common Icons

- Document: File, folder, document-text
- Actions: Plus, edit, delete, download, share
- Navigation: Arrow-left, arrow-right, chevron-down
- Status: Check, X, warning, info
- User: User-circle, user-group
- Settings: Cog, gear

---

## 8. DARK MODE

### 8.1 Color Adjustments

**Backgrounds:**
- Primary: Gray 900 (`#111827`)
- Secondary: Gray 800 (`#1F2937`)
- Tertiary: Gray 700 (`#374151`)

**Text:**
- Primary: Gray 100 (`#F3F4F6`)
- Secondary: Gray 300 (`#D1D5DB`)

**Borders:**
- Default: Gray 700
- Subtle: Gray 800

**Components:**
- Cards: Gray 800 background
- Inputs: Gray 700 background
- Buttons: Adjusted primary colors (lighter variants)

### 8.2 Implementation

- CSS variables for color tokens
- Toggle in user settings
- System preference detection
- Smooth transition between modes

---

## 9. RESPONSIVE DESIGN TOKENS

### 9.1 Breakpoints

- Mobile: `320px - 767px`
- Tablet: `768px - 1023px`
- Desktop: `1024px - 1439px`
- Wide Desktop: `1440px+`

### 9.2 Grid System

**Desktop:**
- 12-column grid
- Gutter: `24px`
- Max width: `1440px` (container)

**Tablet:**
- 8-column grid
- Gutter: `16px`

**Mobile:**
- 4-column grid
- Gutter: `16px`

---

## 10. ANIMATION TOKENS

### 10.1 Timing Functions

- Ease-out: `cubic-bezier(0.0, 0, 0.2, 1)`
- Ease-in: `cubic-bezier(0.4, 0, 1, 1)`
- Ease-in-out: `cubic-bezier(0.4, 0, 0.2, 1)`
- Spring: `cubic-bezier(0.68, -0.55, 0.265, 1.55)`

### 10.2 Durations

- Instant: `0ms`
- Fast: `150ms`
- Normal: `200ms`
- Slow: `300ms`
- Slower: `500ms`

### 10.3 Common Animations

**Fade In:**
- Opacity: 0 → 1
- Duration: `200ms`
- Ease: Ease-out

**Slide Up:**
- Transform: translateY(10px) → translateY(0)
- Opacity: 0 → 1
- Duration: `300ms`
- Ease: Ease-out

**Scale:**
- Transform: scale(0.95) → scale(1)
- Duration: `200ms`
- Ease: Spring

---

## 11. ACCESSIBILITY TOKENS

### 11.1 Focus Indicators

**Default Focus:**
- Outline: 2px solid Primary 500
- Outline offset: `2px`
- Border radius: `4px`

**Keyboard Focus:**
- High contrast mode: 3px solid
- Visible on all interactive elements

### 11.2 Contrast Ratios

**Text:**
- Normal text on white: Gray 800 (4.5:1 minimum)
- Large text: Gray 700 (3:1 minimum)

**Interactive Elements:**
- Button text: White on Primary 500 (4.5:1)
- Link text: Primary 600 on white (4.5:1)

---

**Document Version:** 1.0  
**Created:** December 28, 2025  
**Status:** Visual Design System Complete

