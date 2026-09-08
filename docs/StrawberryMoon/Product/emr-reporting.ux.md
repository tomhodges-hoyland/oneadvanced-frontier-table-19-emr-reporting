---
status: Draft
---

# UX Design Document

## Design Vision

### Product Experience Goals
The EMR Reporting platform delivers an effortless, confidence-inspiring medical reporting workflow across the General Practice team. For Medical Secretaries and Workflow Administrators, the product creates an automated drafting environment where raw electronic health records (EMIS Web, TPP SystmOne, Vision) are instantly synthesized into structured report templates or bespoke letters with automated third-party redactions. For General Practitioners, the platform transforms an arduous, out-of-hours administrative burden into a 2-minute, side-by-side clinical verification and digital sign-off experience. The overarching experience goal is zero cognitive friction, absolute data protection confidence, and seamless asynchronous collaboration between administrative and clinical staff.

### Design Principles
- **Clarity Over Density**: Present complex longitudinal clinical data in clear, scannable chronological units, using progressive disclosure so users only see dense raw EHR records on demand.
- **Defensive Data Safety**: Make redaction states, third-party markers, and clinical approval gates visually unambiguous with clear confirmation micro-interactions to prevent data leakage under UK GDPR.
- **Asynchronous Flow Preservation**: Respect the fragmented workday of primary care staff with auto-saving, robust queue states, persistent draft contexts, and handoff notes between administrators and GPs.
- **Single-Screen Clinical Verification**: Provide GPs with side-by-side source verification (EHR excerpt next to generated report response) to minimize context switching and enable rapid sign-off.

### Brand Alignment
The user experience embodies trust, precision, clinical safety, and NHS digital excellence. The visual language uses calm, clinical-grade slate and deep blue tones accented by crisp semantic indicators. It conveys the reliability of an accredited medical tool while feeling modern, lightweight, and frictionless compared to legacy desktop healthcare software.

## User Research Summary

### Target Users
- **Primary**: Medical Secretary / Practice Workflow Administrator (e.g., Sarah). High-volume task handler responsible for logging incoming third-party requests (insurers, DVLA, DWP, SARs, solicitors), running initial EHR data ingestion, selecting target templates or bespoke letter formats, verifying automated redactions, and packaging clean drafts for GP review.
- **Secondary**: General Practitioner / GP Partner (e.g., Dr. Aris). Time-constrained clinician fitting report reviews into clinical breaks or between surgery sessions. Requires instant access to pre-collated evidence, high-confidence redaction verification, one-click clinical sign-off, and automated billing escrow dispatch.

### Pain Points Addressed
- **Fragmented Out-of-Hours Workload**: Addressed by a structured two-stage relay workflow where administrators complete the initial 90% extraction and formatting, allowing GPs to review and sign off in under 2 minutes during normal surgery hours.
- **Accidental Third-Party Data Breach Risk**: Addressed by AI-assisted redaction highlighting with high-visibility visual chips, one-click toggle controls, and an explicit pre-sign-off redaction audit summary.
- **Manual Printing and Marker-Pen Redaction**: Addressed by digital-native side-by-side document editing, automated SNOMED-CT timeline extraction, and structured questionnaire mapping.
- **Uncollected Private Practice Revenue**: Addressed by visual billing and escrow status tags directly integrated into the request queue, withholding report dispatch until commercial payment confirmation.

## Information Architecture

### Site Map / App Structure
- **Global Dashboard / Worklist Queue**
  - All Requests (Filtered views: Incoming, Admin Draft, GP Review, Escrow / Payment Pending, Completed / Dispatched)
  - Metrics & Compliance Overview (SAR 30-day countdown timers, pending private revenue)
- **Report Drafting Workspace (Administrator View)**
  - Patient & Request Details Header
  - Template & Layout Selector (DVLA, ABI/Life Insurance, DWP, SAR Full Record, Bespoke Free-Text Letter)
  - Side-by-Side Ingestion & Redaction Panel (Left: Extracted EHR Timeline & Redaction Controls | Right: Live Form Preview)
  - Routing & Clinical Handoff Modal
- **Clinical Review Workspace (GP Focus View)**
  - Summary Header (Patient ID, Requestor, SLA Status, Draft Prepared By)
  - Side-by-Side Clinical Verification Canvas (Questionnaire Prompt, Synthesized Answer, Linked EHR Evidence Snippet)
  - Redaction & Safety Audit Drawer
  - One-Click Digital Sign-Off & Escrow Release Bar
- **Finance & Escrow Ledger**
  - Invoice Status, Payment Gateway Webhooks, Commercial Requestor Billing Records
- **Practice Settings & Audit Logs**
  - EHR Partner Integration Status (EMIS/SystmOne/Vision), User Roles, Redaction Dictionary Rules, DCB0129 Clinical Audit Log

### Navigation Model
- **Primary Navigation**: Persistent left-hand collapsible navigation sidebar with icon and label hierarchy: Worklist Queue (with dynamic badge counters for pending actions), Finance Ledger, Audit & Compliance, Practice Settings.
- **Secondary Navigation**: In-workspace top tab bar separating "EHR Chronology & Ingestion", "Form & Letter Builder", "Redaction Review", and "Audit Trail".
- **Search**: Global header search supporting NHS Number, Patient Name, Date of Birth, Requestor Reference Number, and Claim ID with instantaneous typeahead filtering.

### Content Hierarchy
- **Level 1 (Queue & Patient Context)**: Urgency / SLA Badge (SAR expiry countdown), Patient Demographics (NHS Number, DOB, Name), Request Type (DVLA, ABI, Bespoke Letter), Workflow State.
- **Level 2 (Section & Questionnaire Structure)**: Structured Form Questions (e.g., Cardiovascular History, Substance Misuse, Mental Health) mapped to chronological SNOMED-CT EHR extractions.
- **Level 3 (Granular Clinical Evidence & Metadata)**: Consultation Date, Clinician Attribution, Original Encounter Text, Flagged Third-Party Entity Tag, Redaction State Toggle.

## Interaction Design

### Interaction Patterns
- **Split-Pane Synchronized Review**: Side-by-side dual-pane layout pairing questionnaire fields on the right with interactive, highlighted EHR timeline snippets on the left; clicking any generated answer auto-scrolls to and spotlights the corresponding clinical source evidence.
- **One-Click Redaction Toggles**: Third-party names, familial references, and sensitive safeguarding markers appear as floating interactive chips within the text stream; clicking toggles between `[REDACTED]` black-box mask and visible clinical text with instant visual confirmation.
- **Bespoke Letter Builder with Smart Inserts**: Drag-and-and-drop or click-to-insert clinical timeline entries directly into a rich-text markdown letter editor for non-standard solicitor and occupational health inquiries.
- **Sticky Clinical Action Bar**: Bottom-anchored persistent action bar for the GP review screen containing "Request Secretarial Rework", "Edit Entry", and prominent "Approve & Apply Digital Signature" buttons.

### Micro-interactions
- **Redaction State Toggle**: Hovering over a third-party entity chip reveals a quick tooltip with entity classification (e.g., "Third-Party Family Member — UK GDPR Rule"). Clicking triggers an animated collapse into an irreversible visual redaction block.
- **Auto-Save Pulse Indicator**: Subtle green pulse dot and timestamp update ("Saved 2s ago") in the workspace header following every keyboard or redaction interaction to reassure users during interrupted workflows.
- **Sign-Off Confirmation & Stamp Animation**: Completing GP digital sign-off initiates a smooth checkmark stamp animation, dynamically transitioning the status pill from "In Review" to "Signed & Held in Escrow".

### Error Handling
- **Validation**: Inline form field validation prevents workflow progression if mandatory questionnaire fields are left blank or if high-confidence safeguarding flags remain unreviewed by the administrator.
- **Error Messages**: Clear, non-technical, human-readable banner notifications placed directly above the affected section (e.g., "1 unreviewed third-party name detected on Page 3. Please confirm or dismiss redaction before routing to GP.").
- **Recovery**: Automatic draft versioning and continuous session caching allow users to recover interrupted reports immediately upon logging back in, even following unexpected browser closure or EHR session timeouts.

## Visual Design Direction

### Color System
- **Primary**: NHS Blue / Clinical Slate (`#005EB8` primary action, `#1E293B` text and dark accents) providing authority, clinical clarity, and accessibility.
- **Secondary**: Soft Steel Gray (`#F1F5F9` background, `#E2E8F0` borders, `#64748B` supporting text) creating a distraction-free canvas.
- **Semantic**:
  - Success / Escrow Cleared: Emerald (`#059669` / `#ECFDF5`)
  - Warning / SLA Approaching / Flagged Data: Amber (`#D97706` / `#FEF3C7`)
  - Error / Redaction Mask / SLA Breach: Crimson (`#DC2626` / `#FEE2E2`)
  - Info / Review State: Indigo (`#4F46E5` / `#EEF2FF`)

### Typography
- **Headings**: Inter / System Sans-Serif, Semi-Bold (H1: 24px/32px, H2: 20px/28px, H3: 16px/24px) for crisp scannability.
- **Body**: Inter / System Sans-Serif, Regular (14px/20px) optimized for high legibility across dense medical text and clinical notes.
- **UI Elements**: Inter Medium (12px/16px for status badges, button labels, and clinical coding chips; 11px Monospace for NHS Numbers and SNOMED codes).

### Spacing & Layout
- **Grid System**: 12-column responsive fluid grid with fixed 240px collapsible left sidebar and flexible dual 50/50 split workspace containers.
- **Spacing Scale**: 4px base unit (4px, 8px, 12px, 16px, 24px, 32px, 48px) ensuring consistent vertical rhythm and component padding.
- **Breakpoints**: Optimized for standard healthcare desktop displays (1440px wide-screen dual-pane, 1280px standard desktop, 1024px compact view).

### Iconography
- **Style**: Outlined, 1.5px stroke weight, clean geometric curves matching modern clinical software standards.
- **Library**: Lucide Icons (ShieldCheck, FileText, CheckCircle2, AlertTriangle, EyeOff, Lock, Clock, Search, ChevronRight).

## Accessibility

### Standards
- **WCAG Level**: AA Compliance (WCAG 2.1) across all user interfaces, meeting NHS Digital Service Manual standards.
- **Key Requirements**: Minimum 4.5:1 color contrast for all standard text and UI elements; clear focus indicators for keyboard navigation; non-reliance on color alone for redaction and SLA statuses.

### Considerations
- **Screen Readers**: Meaningful ARIA labels for all interactive redaction chips, status badges, and dual-pane synchronization buttons (e.g., `aria-label="Toggle redaction for third-party name John Smith"`).
- **Keyboard Navigation**: Full keyboard tab order and dedicated keyboard shortcuts for high-frequency admin/GP tasks (e.g., `[J]`/`[K]` for next/previous question, `[R]` to toggle redaction, `[Ctrl+Enter]` to sign off).
- **Color Contrast**: High-contrast mode compatibility; all status badges pair semantic color with explicit text labels and supporting icons.
- **Motion**: Full respect for `prefers-reduced-motion` media queries, replacing slide and expand animations with instant state transitions for vestibular safety.

## Responsive Design

### Breakpoints
- **Mobile**: 320px – 767px (Read-only status lookup, emergency review alerts, and billing verification).
- **Tablet**: 768px – 1023px (Single-pane stacked view with tab toggle between EHR source record and report form for tablet-based GP ward/home rounds).
- **Desktop**: 1024px – 1920px+ (Primary operational viewport featuring full side-by-side synchronized split-pane drafting, redaction management, and clinical sign-off).