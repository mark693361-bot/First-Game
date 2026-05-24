# Lost Pet Finder — Design Document

---

## 1. Overview

### 1.1 Purpose
Lost Pet Finder is a web application that connects pet owners with strangers who have found their animal. When a pet goes missing, the owner has no reliable way to know if someone nearby has found it, and the finder has no way to reach the owner without an ID tag. This site solves that by giving owners a place to post a listing and giving finders a place to search, browse, and report.

### 1.2 Problem statement
Every year millions of pets go missing. The current options — posting on neighbourhood Facebook groups, stapling flyers to lampposts, calling local shelters — are fragmented, slow, and geography-limited. Lost Pet Finder brings the problem and solution into one place: a shared, searchable, real-time map of missing animals and found reports that anyone can access from their phone or computer.

### 1.3 Success metrics

#### Qualitative goals
- An owner posts a listing in under three minutes
- A finder can browse missing pets and contact an owner with a single tap
- A lost pet is marked as reunited within days, not weeks
- The site works equally well on a phone (finder walking around) and a desktop (owner posting at home)

#### Measurable KPIs — targets for first 6 months post-launch

| Metric | Definition | Target |
|---|---|---|
| Posting completion rate | % of users who open the Post form and successfully submit | ≥ 70% |
| Reunification rate | % of listings confirmed as reunited before expiry | ≥ 40% |
| Time to first claim | Median hours between a listing going live and receiving its first found claim | ≤ 72 hours |
| Browse-to-contact rate | % of Browse sessions where a phone number is tapped | ≥ 15% |
| Listing renewal rate | % of listings renewed by the owner before they expire | ≥ 30% |
| Found report match rate | % of found reports that trigger at least one owner notification | ≥ 50% |

Metrics are reviewed monthly. Any metric more than 20% below target triggers a prioritised investigation before the next release.

---

## 2. Audience

### 2.1 Who uses this site
The site is open to the general public — no special knowledge or membership required.

**Owners** — people whose pet has gone missing. They need to:
- Post a detailed listing quickly, ideally from their phone
- Be contactable by anyone who spots their pet
- Stay informed when someone reports a possible sighting
- Update or close their listing when the situation changes

**Finders** — strangers who have spotted a pet that may be lost. They need to:
- Browse listings near them without creating an account
- Contact the owner directly if they recognise a listing
- Report a found animal even if no matching listing exists yet

### 2.2 Roles summary

| Action | Account needed? |
|---|---|
| Browse missing pets | No |
| View pet details and phone number | No |
| Post a missing pet listing | **Yes** |
| Upload photos | **Yes** |
| Set listing duration | **Yes** |
| Edit a listing | **Yes** (owner only) |
| Submit a found claim on a specific listing | No — but contact info is required |
| Confirm or reject a found claim | **Yes** (owner only) |
| View notifications | **Yes** |
| Report a found pet with no matching listing | No |

---

## 3. Pages and features

### 3.1 Navigation
Four top-level pages accessed via a sticky navigation bar:
1. Post Missing Pet
2. Browse Missing Pets
3. Report Found Pet
4. Notifications (badge shows unread count)

Logged-in owners also have access to their Account Dashboard via an avatar icon in the top-right corner of the header. The navigation bar is always visible so users can switch context quickly.

---

### 3.2 Post a Missing Pet

#### Purpose
Allow a logged-in owner to create a public listing for their missing pet.

#### Form fields

| Field | Type | Required |
|---|---|---|
| Animal type | Select (dog, cat, bird, other) | Yes |
| Breed / species | Text | No |
| Pet name | Text | Yes |
| Colour | Text | Yes |
| Size | Select (small, medium, large) | Yes |
| Last seen date & time | Date + time picker | Yes |
| Location last seen | Map pin (draggable) | Yes |
| Search radius | Slider (0.5 km – 10 km, default 1 km) | Yes |
| Reward offered | Number (owner's account currency) | No |
| Extra details | Textarea | No |
| Phone number | Tel input (pre-filled from account if set) | Yes |
| Photos | File upload (up to 6 images) | No |
| Listing duration | Select (7 / 14 / 30 / 60 days) | Yes |
| Phone visibility checkbox | Checkbox | Yes — must be ticked |

The phone number field is pre-filled from the owner's account if one is stored. The owner may override it per listing.

#### Photo upload rules
- Up to 6 photos per listing
- Accepted formats: JPG, PNG, WEBP — max 5 MB per file
- Photos upload immediately on selection (not on form submit) with a progress indicator
- First uploaded photo is the card thumbnail; remaining photos appear in the gallery on the detail view
- If no photo is uploaded, a greyed-out animal silhouette is shown as a placeholder

#### Listing duration
- Owner picks: 7, 14, 30, or 60 days
- SMS reminder sent 3 days before expiry
- On expiry: archived (removed from Browse, kept in account history)
- Owner can renew at any time from the Account Dashboard

#### Editing a listing
- Any field editable at any time while active
- Edits go live immediately — no re-approval
- A "last updated" timestamp is shown on the card

#### Validation
- Animal type and pet name are required
- Phone number must match a valid format for the user's locale
- If no location pin is set, posting is blocked with a prompt to drop a pin
- Phone visibility checkbox must be ticked
- Inline error messages appear next to the failing field on submit

#### Post-submit behaviour
- Owner is taken to their new listing's detail view
- SMS confirmation sent to owner's phone number

---

### 3.3 Browse Missing Pets

#### Purpose
A publicly accessible page where anyone can see all active missing pet listings and contact owners.

#### Layout
- Responsive grid: 2 columns (mobile), 3 columns (tablet), 4 columns (desktop)
- Cards ordered by date posted — newest first
- Filter bar above the grid
- 24 cards per page with numbered pagination controls at the bottom

#### Filter: date posted
- All time (default) · Last 24 hours · Last 7 days · Last 30 days

#### Empty states
- No results for active filter: "🔍 No missing pets found for this time range. Try 'All time' or check back soon."
- Zero listings in database: "🐾 No missing pets have been posted yet. If your pet is missing, post a listing above."

#### Pet card — summary view
- Photo (or silhouette placeholder)
- Animal emoji + pet name
- Breed (if provided)
- Size and colour
- Last seen area (neighbourhood/town — not exact address)
- Date posted · Days remaining
- Reward badge (if set)
- Phone number (tap to call on mobile)
- "I Found This Pet!" button

#### Pet card — detail view (own URL)
- Swipeable photo gallery (mobile) / arrow-navigated (desktop) with dot indicators
- All listing fields
- Interactive map: pin + radius circle
- Owner's phone number
- Found claim button
- Report option (three-dot menu, top right)

#### Reunited state
- Card stays on Browse for 3 days after confirmation
- Green ✅ Reunited badge, top-right corner
- Card desaturated (greyscale)
- "I Found This Pet!" button removed
- Auto-archived after 3 days

---

### 3.4 Map and location

#### Per-listing map
- Marker at exact loss location
- Shaded radius circle for the owner's chosen search area
- Interactive: zoom and pan

#### Map provider
Google Maps JavaScript API (or Leaflet + OpenStreetMap as a cost-free fallback — decided at build time).

#### Location privacy
- Exact coordinates shown only on the detail view, not on Browse cards
- Browse card shows only the neighbourhood/town from reverse geocoding

#### Owner location input
1. Type an address — geocoded automatically to a pin
2. Allow browser geolocation
3. Drag the pin manually

---

### 3.5 Report a Found Pet

#### Purpose
Allow any user (no account required) to report a found pet, even without a matching listing.

#### Form fields

| Field | Type | Required |
|---|---|---|
| Animal type | Select | Yes |
| Location found | Map pin or address | Yes |
| Description | Textarea | No |
| Contact (phone or email) | Text | No (encouraged) |

#### Matching logic
When a found report arrives at location **F** with animal type **T**, notify every owner whose listing satisfies:

1. Animal type equals **T**
2. `distance(F, L) ≤ R + 2 km` where L = loss pin, R = listing search radius

The 2 km buffer accounts for pets that have wandered and for imprecision in the finder's reported location. Distance uses the Haversine formula, calculated server-side. This buffer value is a tuning parameter to be revisited after launch using match-rate data.

---

### 3.6 Found claim flow

When a finder clicks **"I Found This Pet!"**:

**Step 1 — Claim modal**

| Field | Required |
|---|---|
| Where did you find them? (map pin or address) | Yes |
| Handover notes | No |
| Your contact — phone or email | **Yes** |

Contact is required here (unlike Report Found Pet) so the outcome notification can always be delivered. The modal explains: *"We need a way to reach you once the owner reviews your claim."*

**Step 2 — Owner notified via SMS and in-app notification**

**Step 3 — Owner reviews claim in Account Dashboard**

- **Confirms** → listing moves to Reunited. Finder notified via their provided contact.
- **Rejects** → listing stays Active. Finder notified via their provided contact.

**Step 4 — No response**
- 48 h: follow-up SMS sent to owner
- 96 h total: claim expires, listing stays Active, finder notified: *"We didn't hear back from the owner. The listing is still active — you can try calling them directly."*

Because contact info is required to submit a claim, there is always a delivery channel. Finders who don't want to share contact details can use the Report Found Pet form (section 3.5) instead.

---

### 3.7 Notifications

Only logged-in owners see the Notifications page. Unread badge count shown on the nav button.

**Triggers:**
- Found claim submitted on an owner's listing
- Found report matching owner's listing (section 3.5 logic)
- Listing 3 days from expiry
- Listing confirmed reunited

**Delivery:** in-app always; SMS for claims and expiry reminders.

**Feed item shows:** animal emoji · message · relative timestamp · link to relevant listing or claim.

**Read state:** opening the Notifications page clears the badge count.

---

### 3.8 Account Dashboard

Private page for logged-in owners. Accessed via the avatar icon — not a main nav item.

**URL:** `/account`

#### Active listings list
Each row shows:
- Thumbnail (or silhouette) · pet name · animal type
- Days remaining with ⚠ warning if ≤ 3 days
- Pending claim count + **Review** button if any claims await
- **Edit** → `/account/listing/:id/edit` (post form pre-filled, all fields editable)
- **Renew** → inline duration picker, expiry recalculated from today
- **Close** → confirmation dialog, then immediate archive

#### Pending claims section
Shown above listing list if any claims are unreviewed:
- Which listing · finder's notes · finder's contact · small map pin · **Confirm** / **Reject** buttons

#### Archived listings
Collapsible list at the bottom. Shows: name, type, reason for archiving (expired / closed / reunited). Archived listings cannot be reactivated.

#### Account settings (sub-page `/account/settings`)
- Change display name · email (re-verification required) · password · phone number
- Delete account (section 9.3)

#### Empty state
> You have no active listings. [Post a Missing Pet →]

---

## 4. Accounts and authentication

### 4.1 Provider
Firebase Authentication — email and password. Google sign-in is a stretch goal for a later release.

### 4.2 Sign-up flow
1. Enter email and password
2. Verification email sent
3. User verifies email
4. Welcome screen shown (see wireframe 17.6) — two panels: "I've lost a pet" → Post, "I've found a pet" → Browse or Report
5. User lands on the chosen page

### 4.3 Sign-in flow
Email and password. Forgotten password resets via email link.

### 4.4 Account data stored
- Display name (internal only, not shown publicly)
- Email address
- Phone number (for SMS — collected at first post if not already set)
- Currency code (ISO 4217 — set from browser locale at signup)
- List of active and archived listing IDs
- SMS opt-out flag

### 4.5 Anonymous posting
No name, username, or profile page is shown publicly. Only the owner's phone number appears on the listing. Owner must acknowledge this with a checkbox before posting.

---

## 5. Design tokens

Design tokens are the single source of truth for all visual decisions. All values below are used directly in CSS custom properties and referenced in every component.

### 5.1 Colour palette

```
── Brand ────────────────────────────────────────────────
primary:          #e8622a   Main orange — buttons, headings, borders
primary-dark:     #b84a18   Shadow and pressed state for primary buttons
primary-light:    #f4943a   Gradient end, light accents
primary-xlight:   #fff0e6   Hover background on nav buttons and ghost buttons
primary-gradient: linear-gradient(135deg, #e8622a 0%, #f4943a 100%)

── Backgrounds ──────────────────────────────────────────
bg-page:          #fff8f2   Page background
bg-surface:       #ffffff   Cards, modals, form panels
bg-input:         #fffaf6   Form field backgrounds (unfocused)
bg-card:          linear-gradient(135deg, #fde8d8 0%, #fce8b4 100%)

── Borders ──────────────────────────────────────────────
border-default:   #f0c8a0   Default input and card borders
border-focus:     #e8622a   Focused input border
border-strong:    #d4956e   Stronger dividers

── Text ─────────────────────────────────────────────────
text-primary:     #2c1a0e   Main body text
text-secondary:   #5a3520   Labels, secondary content
text-muted:       #7a5030   Meta text, timestamps
text-placeholder: #a07050   Input placeholder text
text-inverse:     #ffffff   Text on dark or coloured backgrounds

── Semantic ─────────────────────────────────────────────
error:            #e53935   Error states, destructive actions
error-dark:       #b71c1c   Error button shadow
error-bg:         #fff5f5   Error message backgrounds
success:          #4caf50   Success states, confirm buttons, reunited
success-dark:     #2e7d32   Success button shadow
success-bg:       #f1f8f1   Success message backgrounds
warning:          #ff9800   Expiry warnings
warning-bg:       #fff8e1   Warning message backgrounds

── Special ──────────────────────────────────────────────
gold:             #c8960c   Reward badge text and icon
gold-bg:          #fff3cd   Reward badge background
gold-border:      #e6b800   Reward badge border
overlay:          rgba(0, 0, 0, 0.50)   Modal backdrop
```

### 5.2 Typography

**Font stack**
```
font-family: 'Segoe UI', system-ui, -apple-system, Arial, sans-serif;
```
No custom web font is loaded — system fonts ensure fast paint and consistent rendering across platforms.

**Scale**

| Token | Size | Line height | Weight | Usage |
|---|---|---|---|---|
| text-xs | 12px | 1.4 | 400 | Timestamps, fine print |
| text-sm | 14px | 1.5 | 400 / 600 | Labels, captions, error messages |
| text-base | 16px | 1.5 | 400 | Body text, form inputs |
| text-lg | 18px | 1.4 | 700 | Card pet name, sub-headings |
| text-xl | 20px | 1.3 | 700 | Section headings (h3) |
| text-2xl | 24px | 1.3 | 700 | Page headings (h2) |
| text-3xl | 32px | 1.2 | 700 | Brand heading (h1) |

**Weights**
- 400 Regular — body, placeholder, meta
- 600 Semibold — labels, badge text, nav buttons
- 700 Bold — headings, button text, pet name on card

### 5.3 Spacing scale

All spacing uses an 8 px base grid. Use only these values.

| Token | Value | Common use |
|---|---|---|
| space-1 | 4 px | Icon-to-label gap, tight internal padding |
| space-2 | 8 px | Badge padding, small element gap |
| space-3 | 12 px | Tag padding, compact list gap |
| space-4 | 16 px | Standard gap, page horizontal padding (mobile) |
| space-5 | 20 px | Form field gap |
| space-6 | 24 px | Section inner padding |
| space-7 | 28 px | Card padding |
| space-8 | 32 px | Section gap, page horizontal padding (desktop) |
| space-10 | 40 px | Large section spacing |
| space-12 | 48 px | Hero vertical padding |
| space-16 | 64 px | Between major page sections |

### 5.4 Border radius

| Token | Value | Usage |
|---|---|---|
| radius-sm | 8 px | Inputs, selects, textareas, tags |
| radius-md | 12 px | Pet cards, modals, toasts |
| radius-lg | 16 px | Section panels, large containers |
| radius-pill | 9999 px | Buttons, badges, nav active state |

### 5.5 Shadows

```
shadow-xs:  0 1px 3px rgba(0, 0, 0, 0.05)    Very subtle lift
shadow-sm:  0 2px 8px rgba(0, 0, 0, 0.06)    Sticky nav, filter bar
shadow-md:  0 2px 12px rgba(0, 0, 0, 0.08)   Cards (resting state)
shadow-lg:  0 4px 16px rgba(0, 0, 0, 0.20)   Toasts, dropdowns
shadow-xl:  0 8px 32px rgba(0, 0, 0, 0.25)   Modals
shadow-btn-primary: 0 4px 0 #b84a18          Primary button depth
shadow-btn-success: 0 4px 0 #2e7d32          Success / found button depth
shadow-btn-error:   0 4px 0 #b71c1c          Danger button depth
```

### 5.6 Breakpoints and grid

**Breakpoints**

| Name | Range | Notes |
|---|---|---|
| mobile | 0 – 639 px | Single-column, full-width elements, bottom-sheet modals |
| tablet | 640 – 1023 px | Two-column Browse grid, side-by-side panels |
| desktop | 1024 px + | Three/four-column Browse grid, wider map view |

**Content width**
- Max content width: 760 px (centred, with 16 px side padding on mobile, 32 px on desktop)
- Max page width: 1200 px

**Browse grid columns**

| Breakpoint | Columns | Min card width |
|---|---|---|
| mobile | 2 | ~160 px |
| tablet | 3 | ~175 px |
| desktop | 4 | ~175 px |

---

## 6. Component specifications

### 6.1 Buttons

All buttons use `font-family: inherit`, `font-weight: 700`, `border-radius: radius-pill`, `cursor: pointer`. Minimum touch target: 44 × 44 px (WCAG 2.5.5).

#### Primary button
```
Background:  primary-gradient
Text:        text-inverse · text-base
Padding:     12px 32px  (default)  |  8px 16px (small)
Shadow:      shadow-btn-primary

States:
  Hover   → translateY(-2px), shadow 0 6px 0 #b84a18
  Active  → translateY(2px), shadow 0 2px 0 #b84a18
  Focus   → 2px #e8622a outline, 2px offset (via :focus-visible)
  Disabled → opacity: 0.40, cursor: not-allowed, no shadow, no transform
  Loading  → spinner (20px, white) replaces label text; disabled state applied
```

#### Secondary button
```
Background:  bg-surface
Text:        primary · text-base
Border:      2px solid primary

States:
  Hover   → bg-primary-xlight
  Active  → background #fde8d8
  Focus   → 2px #e8622a outline, 2px offset
  Disabled → opacity: 0.40, cursor: not-allowed
```

#### Ghost button
```
Background:  transparent
Text:        primary · text-base
Border:      none
Shadow:      none

States:
  Hover   → bg-primary-xlight
  Active  → background #fde8d8
```

#### Danger button
```
Background:  error
Text:        text-inverse · text-base
Shadow:      shadow-btn-error
(same hover/active/focus rules as primary — substituting error colours)
```

#### Success / green button (used for "I Found This Pet!")
```
Background:  success
Text:        text-inverse · text-base
Width:       100% (full width within card)
Shadow:      shadow-btn-success
(same hover/active/focus rules as primary — substituting success colours)
```

### 6.2 Form fields

Inputs, selects, and textareas share the same base styles.

```
Height:       44px (input, select) — satisfies WCAG 2.5.5
Background:   bg-input
Border:       2px solid border-default
Border-radius: radius-sm
Font:         text-base / 400
Padding:      10px 14px
Color:        text-primary
Width:        100%

States:
  Placeholder → color: text-placeholder
  Focus       → border-color: border-focus, background: bg-surface
                2px #e8622a outline on :focus-visible for keyboard users
  Error       → border-color: error
                Error message (text-sm, error colour) shown below field
                Associated via aria-describedby
  Disabled    → background: #f5f5f5, opacity: 0.60, cursor: not-allowed
```

**Textarea** — same as above but no fixed height; `min-height: 88px`, `resize: vertical`.

**Select** — same as above; custom caret icon replaces default system arrow.

**Label** — `text-sm / 600`, `text-secondary`, `display: block`, `margin-bottom: space-1`.

**Field wrapper** — `display: flex; flex-direction: column; gap: space-2`. Contains label, input, and error message as siblings.

**Error message** — `text-sm / 400`, `color: error`, rendered as a `<div>` with `role="alert"` and linked to its input via `aria-describedby`.

### 6.3 Pet card

```
Background:   bg-card
Border:       2px solid #f0a870
Border-radius: radius-md
Padding:      space-4 (16px)
Display:      flex, flex-direction: column, gap: space-2
Shadow:       shadow-md (resting)

Interior layout (top to bottom):
  1. Photo or silhouette — 100% width, aspect-ratio 4/3, object-fit cover,
     border-radius: radius-sm, overflow hidden
  2. Pet name — text-lg / 700
  3. Breed (optional) — text-sm / 400, text-muted
  4. Size · Colour — text-sm / 400, text-secondary (separated by ·)
  5. Last seen area — text-sm / 400, text-muted
  6. Date posted · Days remaining — text-xs / 400, text-muted
  7. Reward badge (if set) — see badge spec, margin-top: auto before button
  8. Phone number — text-sm / 600, text-primary (tap-to-call link on mobile)
  9. "I Found This Pet!" button — full-width success button

Hover state (card is clickable to open detail view):
  shadow: 0 6px 20px rgba(0,0,0,0.12)
  transform: translateY(-2px)
  transition: 150ms ease-out

Reunited state:
  filter: grayscale(100%)
  opacity: 0.70
  pointer-events: none on the Found button
  Reunited badge overlaid top-right

Skeleton loading state:
  Replace photo, name, meta with animated shimmer placeholders
  (background: linear-gradient 90deg, #f0c8a0 25%, #fde8d8 50%, #f0c8a0 75%)
  (background-size: 200% 100%, animation: shimmer 1.5s infinite)
```

### 6.4 Modal

```
Backdrop:     fixed inset-0, background: overlay, z-index: 200
Container:    bg-surface, border-radius: radius-md, padding: 28px 36px,
              max-width: 480px, width: calc(100% - 32px), shadow: shadow-xl
              position: fixed top 50% left 50%, transform: translate(-50%,-50%)
              z-index: 201

Mobile:       slides up from bottom, full-width, border-radius top only
Desktop:      centred, as above

Animation:    open  — scale(0.95) → scale(1), opacity 0 → 1, 200ms ease-out
              close — reverse, 150ms ease-in

Behaviour:
  - Closes on backdrop click (unless destructive action is in progress)
  - Closes on Escape key
  - Focus trapped inside while open (Tab cycles only within modal)
  - On open: focus moves to first interactive element or the modal heading
  - On close: focus returns to the trigger element
  - role="dialog", aria-modal="true", aria-labelledby pointing to modal heading
```

### 6.5 Toast notification

```
Position:     fixed, bottom: 24px, right: 24px, z-index: 300
Background:   #2c1a0e
Text:         text-inverse, text-base
Padding:      14px 20px
Border-radius: radius-md
Max-width:    300px
Shadow:       shadow-lg
Left border:  4px solid (success colour for success type, primary for info type)

Animation:
  Enter: translateY(20px) → translateY(0), opacity 0 → 1, 300ms ease-out
  Exit:  reverse, 250ms ease-in

Auto-dismiss: 3200ms after fully entered
Manual dismiss: not available (auto-dismiss only)
Only one toast visible at a time — queued if a second fires while one is showing
```

### 6.6 Badges

**Unread count badge (on nav button)**
```
Background:   error
Text:         text-inverse, text-xs / 700
Min-width:    18px, height: 18px, border-radius: radius-pill
Padding:      0 4px
Position:     absolute, top: -4px, right: -4px
Shows "9+" when count > 9
Animation:    scale(0) → scale(1), 200ms cubic-bezier(0.34, 1.56, 0.64, 1) on appear
```

**Reunited badge (on card)**
```
Background:   success
Text:         text-inverse, text-xs / 700
Content:      "✅ Reunited"
Padding:      4px 10px, border-radius: radius-pill
Position:     absolute, top: 8px, right: 8px
Animation:    opacity 0 → 1, scale(0.8) → scale(1), 400ms ease-out on appear
```

**Reward badge (on card)**
```
Background:   gold-bg
Border:       1px solid gold-border
Text:         gold, text-xs / 700
Content:      "🏅 Reward: [formatted amount + currency]"
Padding:      4px 10px, border-radius: radius-pill
```

**Expiry warning badge (on dashboard listing row)**
```
Background:   warning-bg
Text:         #7a4500, text-xs / 600
Content:      "⚠ Expires in N days"
Shown when:   ≤ 3 days remaining
```

### 6.7 Navigation bar

```
Background:   bg-surface
Border-bottom: 2px solid border-default
Height:       52px
Padding:      0 space-4 (mobile) / 0 space-8 (desktop)
Shadow:       shadow-sm
Position:     sticky, top: 0, z-index: 100

Nav button (default):
  Background:   transparent
  Text:         text-muted, text-sm / 600
  Padding:      8px 18px
  Border-radius: radius-pill
  Border:       2px solid transparent

Nav button (hover):
  Background:   primary-xlight
  Text:         primary

Nav button (active / current page):
  Background:   primary-gradient
  Text:         text-inverse
  Shadow:       0 3px 0 #b84a18

Nav button focus:
  2px #e8622a outline, 2px offset (via :focus-visible)

On mobile (< 640px):
  Nav labels shortened or icon-only with aria-label
  Bar scrolls horizontally if buttons overflow
```

### 6.8 Pagination

```
Display:       flex, gap: space-2, justify-content: center, margin-top: space-8

Page button (default):
  Width/height: 36px, border-radius: radius-sm
  Background:   bg-surface, border: 2px solid border-default
  Text:         text-secondary, text-sm / 600

Page button (current):
  Background:   primary-gradient, border-color: transparent
  Text:         text-inverse

Page button (hover):
  Background:   primary-xlight, border-color: primary

Prev / Next arrows:
  Same sizing as page button
  Disabled (on first/last page): opacity 0.40, pointer-events: none
```

### 6.9 Photo gallery (detail view)

```
Container:     width: 100%, aspect-ratio: 4/3, border-radius: radius-sm,
               overflow: hidden, position: relative

Photos:        object-fit: cover, width: 100%, height: 100%
               Transition between photos: 250ms ease-in-out slide

Desktop controls:
  Left/right arrow buttons — position: absolute, vertically centred
  Background: rgba(0,0,0,0.4), border-radius: 50%
  Width/height: 40px, color: white

Mobile controls:
  Swipe gesture (touch events)
  Dot indicators below gallery
  Dot: 8px circle, bg: rgba(255,255,255,0.5) | Active: white, scale(1.2)

Keyboard:
  Left/right arrow keys navigate photos when gallery is focused
  aria-label on each photo: "[Pet name] — photo N of M"

Thumbnail strip (desktop, when > 1 photo):
  Row of 6 thumbnail slots below main image
  Active thumbnail: 2px primary border
```

### 6.10 Map container

```
Width:         100%
Height:        280px (card detail view) / 400px (post form, larger screens)
Border-radius: radius-sm
Border:        2px solid border-default
Overflow:      hidden

Loading state:
  Background: #e8e0d8 (neutral warm grey)
  Centred spinner (24px, primary colour)

Error / unavailable state (see section 16.3):
  Background: #f5f0eb
  Centred message: map icon + "Map unavailable" text
  Address shown as plain text below
```

### 6.11 Empty states

Used in Browse, Notifications, Account Dashboard, and Archived listings.

```
Layout:   text-align: center, padding: space-10 space-4
Icon:     font-size: 3rem, margin-bottom: space-3
Heading:  text-xl / 700, text-secondary
Body:     text-base / 400, text-muted, max-width: 320px, margin: 0 auto
CTA link: primary colour, text-base / 600 (optional)
```

### 6.12 Skeleton loading cards

Shown in the Browse grid while the first page of listings loads from Firestore.

```
Same outer dimensions as a real pet card
Replace content areas with shimmer blocks:
  Photo area:  full-width rect, aspect-ratio 4/3
  Name:        rect, width 70%, height 18px
  Meta:        rect, width 50%, height 14px
  Two more:    rect, width 80%, height 14px

Shimmer animation:
  background: linear-gradient(90deg, #f0c8a0 25%, #fde8d8 50%, #f0c8a0 75%)
  background-size: 200% 100%
  animation: shimmer 1.5s ease-in-out infinite
  @keyframes shimmer { 0% { background-position: 200% 0 }
                       100% { background-position: -200% 0 } }

Quantity: show 8 skeleton cards while loading
```

---

## 7. Motion and animation

### 7.1 Timing tokens

```
duration-fast:    100ms   Button press active state
duration-base:    150ms   Hover states, nav active, card lift
duration-enter:   200ms   Modals opening, badge appearing
duration-toast:   300ms   Toast sliding in
duration-slow:    400ms   Reunited badge fading in, badge bounce
duration-dismiss: 3200ms  Toast auto-dismiss delay
```

**Easing functions**
```
ease-out:     cubic-bezier(0.0, 0.0, 0.2, 1.0)   Most enter transitions
ease-in:      cubic-bezier(0.4, 0.0, 1.0, 1.0)   Exit transitions
ease-bounce:  cubic-bezier(0.34, 1.56, 0.64, 1)  Badge appear (slight overshoot)
```

### 7.2 Transition inventory

| Element | Property | Duration | Easing | Trigger |
|---|---|---|---|---|
| Primary button | transform, box-shadow | 100ms | ease-out | hover / active |
| Secondary button | background | 150ms | ease-out | hover |
| Nav button | background | 150ms | ease-out | hover / active change |
| Pet card | transform, box-shadow | 150ms | ease-out | hover |
| Form field border | border-color | 200ms | ease-out | focus / blur / error |
| Modal backdrop | opacity | 200ms | ease-out / ease-in | open / close |
| Modal container | opacity, transform(scale) | 200ms | ease-out | open; 150ms ease-in close |
| Toast | opacity, transform(Y) | 300ms | ease-out | enter; 250ms ease-in exit |
| Photo gallery slide | transform(X) | 250ms | ease-in-out | prev / next |
| Unread badge | transform(scale), opacity | 200ms | ease-bounce | appear |
| Reunited badge | opacity, transform(scale) | 400ms | ease-out | confirm |
| Skeleton shimmer | background-position | 1500ms | ease-in-out | continuous loop |

### 7.3 Reduced motion

All users who have `prefers-reduced-motion: reduce` set in their OS receive:
- All `transform` animations disabled (elements appear at final position immediately)
- All `transition` durations capped at `1ms` for transforms
- Opacity-only transitions preserved at their normal duration (opacity conveys state, not motion)
- Skeleton shimmer replaced by a static warm grey block
- Photo gallery slide replaced by an instant cut

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 1ms !important;
    transition-duration: 1ms !important;
  }
}
```

---

## 8. Accessibility

### 8.1 Conformance target
WCAG 2.1 Level AA. All pages and interactive components must pass before shipping.

### 8.2 Colour contrast

All contrast ratios are calculated against actual background values.

| Foreground | Background | Ratio | Requirement | Result |
|---|---|---|---|---|
| text-primary #2c1a0e | bg-page #fff8f2 | 16.8 : 1 | 4.5 : 1 | ✅ Pass |
| text-secondary #5a3520 | bg-surface #ffffff | 9.1 : 1 | 4.5 : 1 | ✅ Pass |
| text-muted #7a5030 | bg-surface #ffffff | 5.1 : 1 | 4.5 : 1 | ✅ Pass |
| text-inverse #ffffff | primary #e8622a | 3.6 : 1 | 3 : 1 (large) | ✅ Pass* |
| text-inverse #ffffff | success #4caf50 | 2.9 : 1 | 3 : 1 (large) | ⚠ Verify |
| error #e53935 | bg-surface #ffffff | 4.8 : 1 | 4.5 : 1 | ✅ Pass |
| gold #c8960c | gold-bg #fff3cd | 3.8 : 1 | 3 : 1 (UI) | ✅ Pass |

*White on primary orange passes for large text (≥ 18px or ≥ 14px bold). Button text at 16px/700 qualifies as large text per WCAG.
⚠ The success green / white combination is marginal — verify with a contrast checker before finalising. If it fails, darken the success green to `#3d9140` or increase button text size.

### 8.3 Keyboard navigation

- Every interactive element is reachable with Tab in logical visual order
- No keyboard traps except inside open modals (intentional — see section 6.4)
- Focus visible style: `2px solid #e8622a, outline-offset: 2px` applied on `:focus-visible`
  Mouse clicks do not show the focus ring (`:focus-visible` vs `:focus`)
- Dropdown menus: open with Enter or Space, navigate with arrow keys, close with Escape
- Photo gallery: left/right arrow keys when gallery container is focused
- Map: Google Maps API provides built-in keyboard support; verify on implementation

### 8.4 Screen reader support

**Images**
- Pet photos: `alt="[Pet name] — [breed if known] [colour] [size]"` e.g. `alt="Buddy — Labrador golden brown large"`
- Silhouette placeholder: `alt="No photo uploaded — dog silhouette placeholder"`
- Decorative icons: `aria-hidden="true"`

**Buttons and links**
- Icon-only buttons must have `aria-label` e.g. `aria-label="Next photo"`, `aria-label="Report this listing"`
- The three-dot menu button: `aria-label="Listing options"`, `aria-expanded` toggled on open/close
- Phone number link: `aria-label="Call owner on 07700 000000"`

**Dynamic content**
- Toast notifications: rendered in a `role="status" aria-live="polite"` region
- Unread badge count: announced with `aria-label="N unread notifications"` on the nav button
- Form validation errors: `role="alert"` on each error `<div>` for immediate announcement
- Listing status changes (Reunited): announced via `aria-live="polite"` region

**Forms**
- All inputs have a visible `<label>` with matching `for`/`id`
- Required fields have `aria-required="true"` and a visible "required" indicator (asterisk with legend)
- Error messages linked to their input via `aria-describedby`

**Modals**
- `role="dialog"`, `aria-modal="true"`, `aria-labelledby="[heading id]"`
- First interactive element receives focus on open
- Trigger element receives focus on close

### 8.5 Additional requirements
- All pages have a unique, descriptive `<title>`
- Skip-to-content link as the very first focusable element on every page (visually hidden until focused)
- Heading hierarchy: one `<h1>` per page, logical `<h2>` / `<h3>` nesting — never skip levels
- Map embeds include a text alternative showing the address
- All form instructions visible before the input, not only as placeholder text

---

## 9. Data and backend

### 9.1 Database
Firebase Firestore (or Supabase PostgreSQL — decided at build time). All listings shared in real time.

### 9.2 File storage
Firebase Storage for pet photos. Files named with UUID, stored under `/listings/:id/photos/`.

### 9.3 Data model

**Users**
`uid, email, phone, displayName, currency (ISO 4217), smsOptOut, createdAt`

**Listings**
`id, ownerUid, animalType, breed, name, colour, size, lastSeenAt, location {lat, lng}, radius, reward, rewardCurrency, details, phone, photoUrls[], duration, expiresAt, status (active | reunited | archived), createdAt, updatedAt`

**FoundReports**
`id, listingId (nullable), animalType, location {lat, lng}, description, finderContact, createdAt`

**Claims**
`id, listingId, finderLocation {lat, lng}, finderNotes, finderContact, finderContactType (phone | email), status (pending | confirmed | rejected | expired), createdAt, resolvedAt`

**Notifications**
`id, ownerUid, type, listingId, claimId, message, read, createdAt`

### 9.4 LocalStorage
Development fallback and client-side unread count cache only. Not used for primary data in production.

---

## 10. Internationalisation (i18n)

Built for full multilingual support from day one.

### 10.1 Approach
- All user-facing strings in locale files (e.g. `en.json`, `fr.json`)
- Active language detected from `Accept-Language` header, overridable in account settings
- Dates, times, currencies, distances formatted via `Intl` API for the active locale
- Reward amounts stored as a plain number with a separate ISO 4217 currency code field

### 10.2 Launch languages
English only. New languages require only a new locale file — no code changes.

### 10.3 RTL support
Layout uses CSS logical properties (`margin-inline-start`, `padding-inline-end`, etc.) so RTL languages work by setting `dir="rtl"` on `<html>`.

---

## 11. Moderation and safety

### 11.1 Approach
Listings go live immediately. Users are trusted by default.

### 11.2 Report button
Three-dot menu on every listing detail view. Options: Fake listing / Inappropriate content / Spam / Other. Reports stored server-side. A listing receiving 3+ reports within 24 hours is auto-hidden pending admin review.

### 11.3 Prohibited content
Terms of service prohibit: fake listings, soliciting money under false pretences, abusive or discriminatory content. Violations: listing removal and/or account suspension.

---

## 12. Legal and privacy

### 12.1 Privacy policy
Linked from footer and sign-up screen. Covers: data collected, how used, who it is shared with (Firebase/hosting providers only), retention (2 years after last activity), and deletion process.

### 12.2 Phone number visibility
Disclaimer shown before the owner enters their number. Checkbox acknowledgement required before posting.

### 12.3 Data deletion
Account deletion removes all personal data and archives active listings within 30 days.

---

## 13. SMS notifications

**Provider:** Twilio (or equivalent — decided at build time).

### Message templates

| Trigger | Message |
|---|---|
| Listing posted | "Your listing for [Name] is live on Lost Pet Finder. Good luck!" |
| Found claim submitted | "Someone thinks they found [Name]! Log in to review the claim: [link]" |
| Claim not reviewed — 48 h | "Reminder: a found claim for [Name] is waiting for your response: [link]" |
| Claim expired — 96 h | "Your claim for [Name] has expired. The listing is still active." |
| Claim confirmed — finder (SMS) | "Great news! [Name]'s owner confirmed you found them. Thank you!" |
| Claim rejected — finder (SMS) | "The owner of [Name] wasn't able to confirm this time. Thank you for trying!" |
| Listing 3 days from expiry | "Your listing for [Name] expires in 3 days. Log in to renew it: [link]" |
| Listing expired | "Your listing for [Name] has expired and is now archived." |
| Listing confirmed reunited | "Great news — [Name] is marked as reunited! We hope they're home safe." |

Every SMS includes a STOP opt-out instruction. Owners who opt out still receive in-app notifications.

---

## 14. Listing lifecycle

```
Posted → Active (counts down) → Expiry warning (T-3 days) → Expired → Archived
              │
              ▼
       Claim submitted (finder provides contact info)
              │
              ▼
       Owner reviews (48 h window)
              │
       ┌──────┴──────┐
       ▼             ▼
   Confirms       Rejects
       │             │
       ▼             ▼
   Reunited      Active (continues)
  (badge 3d)    Finder notified via their contact
       │
       ▼
   Archived

No response after 96 h total:
  → Claim status: expired
  → Listing: remains Active
  → Finder notified via their contact
```

---

## 15. URL structure

All routes are lowercase kebab-case. Dynamic segments prefixed with `:`.

### Public routes

| URL | Page |
|---|---|
| `/` | Browse Missing Pets (home) |
| `/listing/:id` | Pet detail view |
| `/found` | Report a Found Pet |
| `/privacy` | Privacy policy |
| `/terms` | Terms of service |

### Auth routes

| URL | Page |
|---|---|
| `/auth/signup` | Create account |
| `/auth/login` | Sign in |
| `/auth/forgot-password` | Request password reset |
| `/auth/verify-email` | Email verification holding page |
| `/auth/welcome` | Welcome screen (once, after first sign-in) |

### Owner routes (redirect to `/auth/login?redirect=[url]` if signed out)

| URL | Page |
|---|---|
| `/post` | Post a Missing Pet form |
| `/notifications` | Notifications feed |
| `/account` | Account Dashboard |
| `/account/listing/:id/edit` | Edit a listing |
| `/account/listing/:id/claims` | View all claims for a listing |
| `/account/settings` | Account settings |

---

## 16. Error states

### 16.1 Post form errors

| Scenario | User sees |
|---|---|
| Map API fails to load | Text address input shown. Notice: "Map unavailable — enter your last-seen address as text." Listing can still post; pin geocoded server-side when API recovers. |
| Geolocation denied | "Use my location" button shows: "Location access was denied. Type your address or drag the pin manually." |
| Photo upload — network error | Failed photo shows red border + Retry button. Other photos and form unaffected. |
| Photo upload — file too large | Inline error: "This photo is X MB — maximum is 5 MB. Please choose a smaller file." File not queued. |
| Photo upload — wrong format | Inline error: "Only JPG, PNG, and WEBP files are accepted." |
| Form submit — offline | Toast (info): "You appear to be offline. Your listing is saved as a draft and will post when you reconnect." Draft stored in localStorage. |
| Form submit — server error | Toast (info): "Something went wrong on our end. Please try again." Form data not cleared. |
| Duplicate listing detected | Warning banner: "You already have an active listing for a pet called [Name]. Are you sure?" Continue / Cancel. Not a hard block. |

### 16.2 Found claim errors

| Scenario | User sees |
|---|---|
| Listing already confirmed reunited at submit time | Modal: "This pet has just been marked as reunited. Thank you for checking!" Modal closes, card updates. |
| Multiple claims on same listing | All accepted. Owner sees them all in Dashboard, sorted newest first. |
| Claim submit — offline | Toast: "You appear to be offline. Try again when you have a connection — the listing is still active." Modal stays open. |
| Claim submit — server error | Toast: "Something went wrong. Please try again." Modal stays open, data preserved. |
| 96 h no owner response | Claim → expired. Listing → Active. Finder notified via contact (see section 13). |

### 16.3 Map errors

| Scenario | User sees |
|---|---|
| Map unavailable on Browse / detail view | Map area shows: "Map unavailable. Location: [reverse-geocoded address]." Rest of page unaffected. |
| Geocoding fails for typed address | Input error: "We couldn't find that address. Try a different format or drag the pin manually." |
| Browser does not support Geolocation API | "Use my location" button hidden entirely. |
| Stored coordinates are malformed | Map area silently hidden for that listing. Issue logged server-side. |

### 16.4 Photo upload errors

| Scenario | User sees |
|---|---|
| Upload interrupted mid-stream | Red border, frozen progress bar, Retry button. |
| Storage quota exceeded | Toast: "Photo storage is temporarily unavailable. Please try again or contact support." Can post without photos. |
| More than 6 photos selected at once | Only first 6 queued. Notice: "You can upload up to 6 photos. The remaining files were not added." |

---

## 17. Wireframes

Layout wireframes for all primary pages plus key overlays. These define structure, hierarchy, and content placement — not final visual design. All component styles are defined in sections 5 and 6.

---

### 17.1 Browse Missing Pets (home)

```
┌─────────────────────────────────────────────────────────────┐
│  🐾 Lost Pet Finder                          [Sign in / 👤] │
├──────────┬──────────────────┬──────────────┬────────────────┤
│ Post Pet │  Browse (active) │  Found Pet   │  🔔 Notifs (3) │
├─────────────────────────────────────────────────────────────┤
│  Filter by date posted:  [ All time ▼ ]   Showing 12 results│
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│ │  [Photo]    │ │  [Photo]    │ │ [Silhouette]│            │
│ │─────────────│ │─────────────│ │─────────────│            │
│ │ 🐕 Buddy    │ │ 🐈 Whiskers │ │ 🐕 Max      │            │
│ │ Lab · Large │ │ Sm · Grey   │ │ Med · Black │            │
│ │ Riverside   │ │ Town centre │ │ Park Road   │            │
│ │ 2 days ago  │ │ 5 hours ago │ │ 1 day ago   │            │
│ │ 🏅 Reward   │ │             │ │             │            │
│ │ 07700 0000  │ │ 07700 1111  │ │ 07700 2222  │            │
│ │[I Found It!]│ │[I Found It!]│ │[I Found It!]│            │
│ └─────────────┘ └─────────────┘ └─────────────┘            │
│                                                             │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│ │ [skeleton]  │ │ [skeleton]  │ │ [skeleton]  │            │
│ └─────────────┘ └─────────────┘ └─────────────┘            │
│                                                             │
│              [← Prev]  1  [2]  3  [Next →]                 │
├─────────────────────────────────────────────────────────────┤
│  Lost Pet Finder · Privacy policy · Terms of service        │
└─────────────────────────────────────────────────────────────┘
```

---

### 17.2 Post a Missing Pet

```
┌─────────────────────────────────────────────────────────────┐
│  🐾 Lost Pet Finder                              [👤 Sarah] │
├──────────────────┬────────────────┬──────────────┬──────────┤
│  Post (active)   │  Browse Pets   │  Found Pet   │  🔔 2   │
├─────────────────────────────────────────────────────────────┤
│  Post a Missing Pet                                         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Animal type *   [ Dog                ▼ ]             │  │
│  │  Breed           [ e.g. Labrador          ]           │  │
│  │  Pet name *      [                        ]           │  │
│  │  Colour *        [                        ]           │  │
│  │  Size *          [ Medium              ▼ ]            │  │
│  │                                                       │  │
│  │  Last seen *     [ 12/05/2026 ] [ 15:00 ]             │  │
│  │                                                       │  │
│  │  Location last seen *                                 │  │
│  │  ┌───────────────────────────────────────────────┐   │  │
│  │  │  [Interactive map]                            │   │  │
│  │  │                       📍                     │   │  │
│  │  │    (shaded radius circle around pin)          │   │  │
│  │  └───────────────────────────────────────────────┘   │  │
│  │  [ Use my location ]  or  [ Type an address... ]      │  │
│  │  Search radius  ●──────────────── 1.0 km             │  │
│  │                                                       │  │
│  │  Reward (optional)  [ £            ]                  │  │
│  │                                                       │  │
│  │  Extra details  [                              ]      │  │
│  │                 [                              ]      │  │
│  │                                                       │  │
│  │  Phone number *  [ 07700 000000          ]            │  │
│  │  ⚠ Your number will be visible on your listing        │  │
│  │                                                       │  │
│  │  Photos (up to 6)                                     │  │
│  │  [ ＋ Add ][ ＋ Add ][ ＋ Add ][ ＋ Add ][ ＋ Add ][ ＋ ]│  │
│  │                                                       │  │
│  │  Listing duration *  [ 30 days           ▼ ]          │  │
│  │                                                       │  │
│  │  ☐  I understand my phone number will be public       │  │
│  │                                                       │  │
│  │              [      Post Missing Pet      ]           │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

### 17.3 Pet Detail View

```
┌─────────────────────────────────────────────────────────────┐
│  🐾 Lost Pet Finder                              [👤 Sarah] │
├──────────┬────────────────────┬──────────────┬──────────────┤
│ Post Pet │  Browse (active)   │  Found Pet   │  🔔 2       │
├─────────────────────────────────────────────────────────────┤
│  ← Back to Browse                                    ⋮ Menu │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  [ Photo 1 ]  [ Photo 2 ]  [ Photo 3 ]   ← arrows   │   │
│  │                                                      │   │
│  │              (main photo displayed)                  │   │
│  │                                                      │   │
│  │       ●  ○  ○  ←  dot indicators                    │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  🐕 Buddy                                🏅 Reward: £50    │
│  Labrador · Large · Golden brown                            │
│  Last seen: 12 May 2026, 3:00 pm                            │
│  Riverside Park, Northampton                                │
│  Posted 2 days ago · Expires in 28 days                     │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  [ Interactive map — pin + shaded radius circle ]    │   │
│  │                                                      │   │
│  │              ○ ─ ─ ─ 📍 ─ ─ ─ ○                    │   │
│  │                                                      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  About Buddy                                                │
│  Friendly dog. Wears a red collar. Responds to "Buddy".     │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Contact the owner                                   │   │
│  │  📞 07700 000000               [ Tap to call ]       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│            [       I Found This Pet! 🎉       ]             │
└─────────────────────────────────────────────────────────────┘
```

---

### 17.4 Report a Found Pet

```
┌─────────────────────────────────────────────────────────────┐
│  🐾 Lost Pet Finder                              [ Sign in ]│
├──────────┬──────────────────┬──────────────────┬────────────┤
│ Post Pet │  Browse Pets     │  Found (active)  │  🔔       │
├─────────────────────────────────────────────────────────────┤
│  Report a Found Pet                                         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Animal type *   [ Cat                ▼ ]             │  │
│  │                                                       │  │
│  │  Where did you find them? *                           │  │
│  │  ┌───────────────────────────────────────────────┐   │  │
│  │  │  [Map — tap to drop a pin]                    │   │  │
│  │  │                    📍                         │   │  │
│  │  └───────────────────────────────────────────────┘   │  │
│  │  or  [ Type an address...                    ]        │  │
│  │                                                       │  │
│  │  Description    [                              ]      │  │
│  │                 [                              ]      │  │
│  │                                                       │  │
│  │  Your contact   [  Phone or email              ]      │  │
│  │  Sharing your contact helps the owner reach you       │  │
│  │                                                       │  │
│  │              [      Report Found Pet      ]           │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

### 17.5 Account Dashboard

```
┌─────────────────────────────────────────────────────────────┐
│  🐾 Lost Pet Finder                              [👤 Sarah] │
├──────────┬──────────────────┬──────────────┬────────────────┤
│ Post Pet │  Browse Pets     │  Found Pet   │  🔔 2         │
├─────────────────────────────────────────────────────────────┤
│  My Account                                                 │
│                                                             │
│  ─── Pending claims (1) ───────────────────────────────     │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ 🐕 Buddy — claim received 1 hour ago                 │   │
│  │ "Found near the canal. I'm at home — call me."       │   │
│  │ Contact: 07900 123456                                │   │
│  │ 📍 [mini map pin] — 0.4 km from loss location        │   │
│  │                    [ Confirm ]    [ Reject ]          │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ─── Active listings (2) ──────────────────────────────     │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ [thumb] 🐕 Buddy · Posted 3 days ago                 │   │
│  │         ● Active · Expires in 27 days                │   │
│  │         1 pending claim                              │   │
│  │                   [ Edit ]  [ Renew ]  [ Close ]     │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ [silh]  🐈 Whiskers · Posted 10 days ago             │   │
│  │         ● Active · ⚠ Expires in 3 days               │   │
│  │         No claims yet                               │   │
│  │                   [ Edit ]  [ Renew ]  [ Close ]     │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ─── Archived listings (3) ─────────── [ Show archive ▼ ]  │
│                                                             │
│  ─── Account settings ─────────────────────────────────     │
│  Email      sarah@example.com                  [ Change ]   │
│  Phone      07700 000000                       [ Change ]   │
│  Password   ••••••••                           [ Change ]   │
│                                          [ Delete account ] │
└─────────────────────────────────────────────────────────────┘
```

---

### 17.6 Welcome Screen (shown once after first sign-in)

```
┌─────────────────────────────────────────────────────────────┐
│  🐾 Lost Pet Finder                                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│              Welcome to Lost Pet Finder                     │
│          Helping pets find their way home 🐾                │
│                                                             │
│  ┌──────────────────────┐  ┌──────────────────────────┐    │
│  │        🔍            │  │           📍             │    │
│  │                      │  │                          │    │
│  │   I've lost a pet    │  │   I've found a pet       │    │
│  │                      │  │                          │    │
│  │  Post a listing so   │  │  Browse missing pets     │    │
│  │  finders can reach   │  │  or report the one       │    │
│  │  you directly.       │  │  you've found.           │    │
│  │                      │  │                          │    │
│  │  [ Post a listing ]  │  │  [ Browse listings ]     │    │
│  └──────────────────────┘  └──────────────────────────┘    │
│                                                             │
│              [ Skip — take me to Browse ]                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### 17.7 Found Claim Modal

```
┌─────────────────────── overlay ─────────────────────────────┐
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  You found Buddy! 🎉                                  │  │
│  │                                                       │  │
│  │  Tell the owner a bit more so they can confirm.       │  │
│  │                                                       │  │
│  │  Where did you find them? *                           │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │ [Small map — tap to drop pin]   📍              │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │  or  [ Type an address...                    ]        │  │
│  │                                                       │  │
│  │  Handover notes (optional)                            │  │
│  │  [ e.g. "I have Buddy at my flat on Oak Street" ]     │  │
│  │                                                       │  │
│  │  Your contact *                                       │  │
│  │  [ Phone or email                              ]      │  │
│  │  We'll use this to send you the owner's response.     │  │
│  │                                                       │  │
│  │      [ Cancel ]        [ Submit Claim ]               │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### 17.8 Browse — Error / Empty States

```
── No results for active filter ──────────────────────────────

  Filter: [ Last 24 hours ▼ ]   0 results

  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │                      🔍                             │
  │                                                     │
  │     No missing pets found for this time range.      │
  │       Try "All time" or check back soon.            │
  │                                                     │
  └─────────────────────────────────────────────────────┘


── Form validation error state ───────────────────────────────

  Pet name *
  ┌────────────────────────────────────────────────────┐
  │                                                    │  ← red border
  └────────────────────────────────────────────────────┘
  ⚠ Please enter the pet's name                         ← error-sm, error colour

  Animal type *
  ┌────────────────────────────────────────────────────┐
  │  -- pick one --                                ▼   │  ← red border
  └────────────────────────────────────────────────────┘
  ⚠ Please select an animal type


── Map unavailable — detail view ─────────────────────────────

  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │              🗺  Map unavailable                    │
  │                                                     │
  │   Last seen at: Riverside Park, Northampton         │
  │                                                     │
  └─────────────────────────────────────────────────────┘
```

---

## 18. What can wait until later

- **Filters:** Animal type and size filters on Browse (only date filter in v1)
- **Full overview map:** Single map with all active listings clustered
- **In-app messaging:** Chat without exchanging phone numbers
- **Email notifications:** In addition to SMS
- **Push notifications:** Browser or mobile push
- **Social sharing:** Share buttons or copy-link per listing
- **Google / social sign-in:** OAuth in addition to email and password
- **Admin dashboard:** UI for flagged listings and account management
- **Breed autocomplete:** Searchable breed database
- **Distance filter:** Filter Browse by proximity to user's location
- **Additional languages:** Locale files beyond English
- **Public owner profile:** Optional named profile with all active listings
- **Microchip / ID number field**
- **Last seen date filter:** Filter Browse by when the pet was last seen
- **Found report map overlay:** Second layer on the map for found reports
- **Matching radius tuning:** A/B test the 2 km buffer using post-launch reunification rate data
- **Colour contrast fix for success green:** If `#4caf50` on white fails AA at small sizes, darken to `#3d9140`
