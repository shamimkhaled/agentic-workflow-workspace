# FlowSpark AI - Screen-by-Screen Design Specifications
## Detailed Layout & Interaction Specifications

---

## 1. LANDING PAGE

### 1.1 Hero Section

**Layout:**
- Full-width hero with gradient background
- Left: Headline, subheadline, CTA buttons
- Right: Product illustration or video preview

**Components:**
- Primary headline: "Transform Documents with AI-Powered Workflows" (H1, 48px)
- Subheadline: "Create professional documents, presentations, and insights in minutes, not hours" (24px)
- Primary CTA: "Start Free Trial" (button, primary)
- Secondary CTA: "Watch Demo" (button, outline)
- Trust indicators: "Used by 500+ organizations"

**Responsive:**
- Mobile: Stacked layout, centered text
- Tablet: Adjusted spacing
- Desktop: Side-by-side layout

### 1.2 Features Section

**Layout:**
- 3-column grid (desktop), 1-column (mobile)
- Icon + Title + Description cards

**Content:**
- Feature cards:
  1. AI Document Generation (icon, title, description)
  2. Knowledge Base Integration (icon, title, description)
  3. Multi-Agent Workflows (icon, title, description)
  4. Enterprise Security (icon, title, description)
  5. Token-Based Pricing (icon, title, description)
  6. Template Library (icon, title, description)

### 1.3 Pricing Section

**Layout:**
- 4 pricing cards in horizontal row
- Toggle: Monthly / Yearly billing

**Pricing Cards:**
- Free: $0/month
- Standard: $49/month
- Professional: $199/month
- Enterprise: Custom

**Each Card Includes:**
- Plan name and price
- Feature list (checkmarks)
- Limits (users, documents, storage)
- Included tokens
- CTA button

---

## 2. AUTHENTICATION SCREENS

### 2.1 Login Page

**Layout:**
- Centered card on background
- Left side (desktop): Branding illustration
- Right side: Login form

**Form Fields:**
- Email input (required, email validation)
- Password input (required, show/hide toggle)
- "Remember me" checkbox
- "Forgot password?" link
- Submit button: "Sign In"
- Divider: "OR"
- SSO buttons: "Continue with Google", "Continue with Microsoft"

**Footer:**
- "Don't have an account? Sign up" link

### 2.2 Sign Up Page

**Layout:**
- Similar to login, multi-step process

**Step 1: Account Creation**
- Email
- Password (strength indicator)
- Confirm password
- Accept terms checkbox
- Continue button

**Step 2: Organization Setup**
- Organization name
- Industry (dropdown)
- Team size (dropdown)
- Continue button

**Step 3: Plan Selection**
- Plan comparison cards
- Select plan button
- "Start with free trial" option

### 2.3 Password Reset

**Flow:**
1. Request reset: Email input
2. Email sent confirmation
3. Reset link in email
4. New password form
5. Success confirmation

---

## 3. DASHBOARD

### 3.1 Header Section

**Components:**
- Logo (left)
- Primary navigation (center): Dashboard, Documents, Knowledge Base, Templates, Analytics
- Search bar (global search, keyboard shortcut /)
- Notification bell icon (with badge count)
- User avatar menu (dropdown)

**User Menu Dropdown:**
- Profile
- Settings
- Billing
- Help & Support
- Switch Workspace
- Logout

### 3.2 Main Dashboard Content

**Welcome Section:**
- Personalized greeting: "Welcome back, [Name]"
- Quick stats: Documents created this month, Token balance, Storage used
- Quick action buttons: Create Document, Upload to KB, View Templates

**Recent Activity Grid:**
- 2-3 column grid of recent documents
- Each card shows:
  - Thumbnail/preview
  - Title
  - Type icon
  - Last modified date
  - Quick actions (hover): Download, Share, Delete

**Usage Statistics Cards:**
- Token usage chart (line/bar chart)
- Documents created (sparkline)
- Storage usage (progress bar)
- Monthly comparison

**Activity Feed:**
- Timeline of recent actions
- Filter by: All, Documents, KB, Team Activity
- Load more option

### 3.3 Empty State (First Time)

**Content:**
- Large illustration (friendly, welcoming)
- Headline: "Let's create your first document"
- Subheadline: "Choose a template or start from scratch"
- Primary CTA: "Create Document"
- Secondary CTA: "Browse Templates"
- Helper text: "Need help? Watch our 2-minute tutorial"

---

## 4. DOCUMENT CREATION WORKFLOW

### 4.1 Creation Modal (Step 1: Type Selection)

**Layout:**
- Centered modal, 800px max width
- Progress indicator: Step 1 of 4
- Title: "Create New Document"

**Type Cards (Grid):**
- Word Document (icon, label)
- PDF Report (icon, label)
- Excel Spreadsheet (icon, label)
- PowerPoint Presentation (icon, label)

**Selection:**
- Hover: Highlight border
- Selected: Filled background, checkmark
- Continue button (disabled until selection)

### 4.2 Template Selection (Step 2)

**Layout:**
- Same modal, progress: Step 2 of 4
- Search bar for templates
- Filter tabs: All, Business, Legal, Technical, Custom
- Grid of template cards

**Template Card:**
- Preview thumbnail
- Template name
- Usage count
- Star/favorite icon
- "Use This Template" button

**Options:**
- "Start from scratch" button
- "Recent templates" section

### 4.3 Content Input (Step 3)

**Layout:**
- Two-column: Form (left), Preview/Preview placeholder (right)
- Progress: Step 3 of 4

**Form Fields:**
- Document Title (required)
- Description (textarea, optional)
- Content Input:
  - Mode toggle: "Simple" / "Advanced"
  - Simple: Large textarea with placeholder
  - Advanced: Structured form with sections

**Knowledge Base Integration:**
- Toggle: "Use Knowledge Base Context"
- Search/filter KB documents
- Selected documents shown as chips
- Max documents indicator

**Advanced Options (Collapsible):**
- Output format preferences
- Style preferences
- Brand guidelines
- Custom instructions

**Preview Panel:**
- Live preview of generated structure
- Estimated token cost
- Estimated generation time

### 4.4 Generation Progress (Step 4)

**Layout:**
- Full-screen overlay or modal
- Centered progress indicator

**Progress Display:**
- Circular progress indicator (0-100%)
- Current step: "Generating content..." (dynamic text)
- Agent activity indicators:
  - Orchestrator Agent: Active
  - Document Agent: Processing
  - KB Agent: (if used) Retrieving context
  - Image Agent: (if used) Generating visuals
- Estimated time remaining

**Options:**
- Cancel button (only if cancellable)
- Minimize to background option

### 4.5 Document Preview/Editor

**Layout:**
- Split view: Preview (left 70%), Actions (right 30%)

**Preview Panel:**
- Full document preview
- Scrollable content
- Zoom controls
- Print preview option

**Actions Panel:**
- Document Info:
  - Title, type, size
  - Created date, tokens used
  - Generation time
- Actions:
  - Download (format selector)
  - Share (modal trigger)
  - Duplicate
  - Edit/Regenerate
  - Delete
- Metadata:
  - Template used
  - AI model used
  - KB sources (if applicable)

**Edit Mode:**
- Inline editing capabilities
- Formatting toolbar
- Save changes button

---

## 5. DOCUMENTS LIST PAGE

### 5.1 Page Header

**Components:**
- Page title: "Documents"
- Primary action: "+ Create Document" button
- View toggle: Grid / List view icons
- Filter button (with active filter count badge)
- Sort dropdown: Date, Name, Type, Size

### 5.2 Filters Panel (Slide-out)

**Filters:**
- Type: Checkboxes (Word, PDF, Excel, PPT)
- Status: Checkboxes (Complete, Processing, Failed)
- Date Range: Date picker
- Tags: Multi-select dropdown
- Created By: User dropdown
- Visibility: All, My Docs, Shared

**Actions:**
- Apply Filters button
- Clear All link
- Save Filter Set (for power users)

### 5.3 Document Grid View

**Layout:**
- Responsive grid: 4 columns (desktop), 3 (tablet), 2 (mobile)
- Card-based layout

**Document Card:**
- Thumbnail/preview (top)
- Title (truncated, 2 lines max)
- Metadata row:
  - Type icon
  - File size
  - Created date (relative: "2 days ago")
- Status badge (if processing/failed)
- Hover: Quick actions overlay
  - Download
  - Share
  - Duplicate
  - Delete

### 5.4 Document List View

**Layout:**
- Table format with sortable columns

**Columns:**
- Checkbox (for bulk actions)
- Thumbnail (small)
- Title (sortable)
- Type (sortable)
- Size (sortable)
- Created (sortable, relative time)
- Status
- Actions (3-dot menu)

**Row Actions:**
- Hover: Highlight row
- Click row: Open detail view
- Click 3-dot menu: Context menu

### 5.5 Bulk Actions Bar

**Trigger:** Select multiple documents

**Bar Components:**
- Selected count: "3 documents selected"
- Actions:
  - Download (zip)
  - Share (batch)
  - Delete (batch)
  - Apply Tag
- Clear selection button

---

## 6. DOCUMENT DETAIL VIEW

### 6.1 Document Viewer

**Layout:**
- Full-screen or split-panel view
- Document preview (main area)
- Side panel: Metadata and actions (toggleable)

**Preview:**
- Document rendered in iframe or PDF viewer
- Zoom controls: -, +, fit to width, actual size
- Navigation: Previous/Next document arrows
- Print button
- Download button (format selector)

**Side Panel (Collapsible):**
- Document info section
- Metadata section
- Actions section
- Share settings section
- Version history section

### 6.2 Share Modal

**Layout:**
- Modal, 600px width

**Tabs:**
- Share Link
- Share via Email
- Team Members

**Share Link Tab:**
- Toggle: Enable link sharing
- Link display (copy button)
- Permissions dropdown: View, Edit, Admin
- Expiration date (optional)
- Password protection (optional)

**Email Tab:**
- Email input (multiple recipients)
- Message (optional)
- Permission selector
- Send button

**Team Members Tab:**
- Search/add team members
- Permission level per person
- List of current shares
- Remove access option

---

## 7. KNOWLEDGE BASE INTERFACE

### 7.1 KB Dashboard

**Layout:**
- Header: "Knowledge Base" + "Upload Documents" button
- Stats cards: Total docs, Total pages, Storage used
- Quick actions: Upload, Query Assistant

**Tabs:**
- All Documents
- Collections
- Query History

### 7.2 Upload Interface

**Layout:**
- Modal or dedicated page

**Upload Methods:**
1. Drag & Drop Zone:
   - Large drop area
   - "Drop files here" text
   - Supported formats listed
   - File size limits shown

2. File Browser:
   - "Choose Files" button
   - File picker dialog

**Upload Queue:**
- List of queued files
- Progress bars per file
- Cancel option per file
- Processing status indicators

**Processing Status:**
- Uploading (progress bar)
- Extracting text
- Chunking
- Generating embeddings
- Indexing
- Complete (checkmark)

### 7.3 Document Library

**Layout:**
- List/Grid view toggle
- Search bar
- Filter sidebar: Status, Type, Date, Tags

**Document List Item:**
- Thumbnail or icon
- Title
- File info: Type, size, pages
- Processing status
- Uploaded date
- Chunks count
- Actions menu: View, Delete, Re-index

### 7.4 Query Assistant Interface

**Layout:**
- Chat-like interface (similar to ChatGPT)

**Components:**
- Chat history (scrollable)
- Input area:
  - Text input (multiline)
  - Send button
  - Voice input button (future)
  - Attach documents button
- Suggested queries (below input):
  - "What is our return policy?"
  - "Summarize Q4 performance"
  - "Find customer complaints"

**Message Bubbles:**
- User message: Right-aligned, blue
- AI response: Left-aligned, white/gray
  - Answer text
  - Sources section (collapsible):
    - List of source documents
    - Relevance scores
    - Click to view source
  - Citation numbers in text
  - Follow-up questions (chips)

**Query Controls:**
- Clear conversation
- Export conversation
- Feedback buttons (thumbs up/down)

---

## 8. TEMPLATES PAGE

### 8.1 Template Library

**Layout:**
- Header: "Templates" + "Upload Template" (admin only)
- Category tabs: All, Business, Legal, Technical, Marketing, Custom
- Search bar
- Filter options: Public, Private, Favorites

**Template Grid:**
- Template cards (similar to document cards)
- Preview thumbnail
- Template name
- Category badges
- Usage count
- Star/favorite icon
- "Use Template" button

### 8.2 Template Preview

**Modal:**
- Large preview
- Template details
- Variables list
- Preview with sample data
- "Use This Template" CTA

---

## 9. ANALYTICS DASHBOARD

### 9.1 Overview Section

**Key Metrics Cards:**
- Documents generated (total, this month)
- Token usage (current balance, used this month)
- Storage used (current, limit, %)
- Active users (this month)

### 9.2 Charts Section

**Usage Chart:**
- Line chart: Documents over time
- Date range selector: 7 days, 30 days, 90 days, Custom
- Comparison: Previous period toggle

**Token Consumption:**
- Bar chart: Tokens by operation type
- Breakdown: Document gen, KB queries, Image gen, etc.

**User Activity:**
- Table: Users, documents created, tokens used
- Sortable columns
- Export to CSV

**Cost Analysis:**
- Token cost breakdown
- Projected monthly cost
- Cost per user
- Optimization suggestions

---

## 10. SETTINGS PAGES

### 10.1 Profile Settings

**Form Sections:**
- Personal Information:
  - Avatar upload
  - First name, Last name
  - Email (non-editable, link to change)
  - Phone number
- Preferences:
  - Timezone (dropdown)
  - Language (dropdown)
  - Date format
  - Notification preferences
- Save button

### 10.2 Workspace Settings

**Sections:**
- Workspace Information:
  - Organization name
  - Logo upload
  - Industry
  - Description
- Branding:
  - Primary color picker
  - Logo upload
  - Font preferences
- Team Management:
  - Invite members (email input)
  - Member list table
  - Role management per user
- Danger Zone:
  - Delete workspace (with confirmation)

### 10.3 Billing & Subscription

**Current Plan Card:**
- Plan name and tier
- Renewal date
- Change plan button

**Usage Overview:**
- Token balance (prominent)
- Documents used / limit
- Storage used / limit
- Progress bars

**Billing History:**
- Table of invoices
- Download PDF link
- Payment method management

**Token Purchase:**
- Package options (cards)
- Quantity selector
- Total cost display
- Purchase button

---

## 11. MODAL COMPONENTS

### 11.1 Confirmation Modal

**Layout:**
- Small modal, centered
- Icon (warning, info, success)
- Title (bold)
- Message (body text)
- Actions:
  - Cancel (secondary)
  - Confirm (primary, may be destructive red)

### 11.2 Notification Toast

**Types:**
- Success (green)
- Error (red)
- Warning (yellow)
- Info (blue)

**Position:**
- Top-right corner (desktop)
- Top center (mobile)

**Behavior:**
- Auto-dismiss after 5 seconds
- Manual close button
- Stack multiple notifications
- Non-intrusive

---

**Document Version:** 1.0  
**Created:** December 28, 2025  
**Status:** Screen Specifications Complete

