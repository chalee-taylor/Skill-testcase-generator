# SPEC_UNSPLASH.md — Feature Specification: Unsplash.com (Refined I/O + Workflow)

**Version:** 1.1.0  
**Author:** QC Team — Cook A Skill  
**Last Updated:** 2026-02-25  
**Target Site:** https://unsplash.com/  
**Spec Type:** Product Feature Specification (PRD)

---

## Table of Contents

1. [Product Overview](#1-product-overview)
2. [Standard Feature Format](#2-standard-feature-format)
3. [Feature Specs](#3-feature-specs)
   - [F1. Photo Search & Discovery](#f1-photo-search--discovery)
   - [F2. Photo Download](#f2-photo-download)
   - [F3. User Registration & Login](#f3-user-registration--login)
   - [F4. Photo Upload (Contributors)](#f4-photo-upload-contributors)
   - [F5. Collections Management](#f5-collections-management)
   - [F6. Like / Save Photos](#f6-like--save-photos)
   - [F7. User Profile](#f7-user-profile)
   - [F8. Unsplash+ Subscription](#f8-unsplash-subscription)
   - [F9. Visual Search](#f9-visual-search)
   - [F10. API Access (Developers)](#f10-api-access-developers)
4. [Cross-Feature Constraints](#4-cross-feature-constraints)
5. [Appendix — Business Rule Index](#5-appendix--business-rule-index)

---

## 1. Product Overview

**Product Name:** Unsplash  
**Type:** Community-powered stock photo platform  
**Core Value:** Discover, share, and legally use high-quality photos.

### Primary User Roles

| Role | What they can do |
|---|---|
| Guest | Browse and download free photos |
| Registered User | Like photos, create collections, manage profile |
| Contributor | Upload photos for review and publication |
| Unsplash+ Subscriber | Access/download premium content |
| Developer | Use Unsplash API via app registration |
| Admin | Moderate content/accounts and enforce policy |

---

## 2. Standard Feature Format

Each feature below follows this structure to make test design easier and unambiguous:

1. **Goal** — business purpose of the feature  
2. **Input** — actor, entry point, required data, optional data  
3. **Output** — UI/system response, state change, notification/API response  
4. **Workflow** — main flow + alternative/exception flows  
5. **Validation & Business Rules** — constraints and rule IDs

---

## 3. Feature Specs

## F1. Photo Search & Discovery

### Goal
Help users quickly discover relevant photos, collections, and creators.

### Input
- **Actor:** Guest or registered user
- **Entry points:** Header search bar, homepage, category nav
- **Required input:** `keyword` (1–200 chars after trim)
- **Optional input:** `tab` (Photos/Collections/Users), `sort` (Relevance/Latest), category filter

### Output
- Search result page with tabbed results and tag suggestions
- Infinite-scroll photo grid on homepage/search pages
- Empty state when no result: `We couldn't find anything for [keyword]`

### Workflow
1. User enters keyword and submits.
2. System validates keyword length and non-empty value.
3. System returns tabbed results (Photos, Collections, Users).
4. User may refine via tag/category/sort.
5. System loads next result batch on scroll.

**Exception workflow**
- If result set is empty, show empty state + suggested alternatives.

### Validation & Business Rules
- BR-001, BR-002, BR-003, BR-004, BR-005, BR-006, BR-007, BR-008, BR-009, BR-010.
- Whitespace-only keyword is invalid; keywords > 200 chars are truncated.

---

## F2. Photo Download

### Goal
Allow users to download free images and guide premium users through Unsplash+ access.

### Input
- **Actor:** Guest, registered user, or Unsplash+ subscriber
- **Entry points:** Photo card or photo detail page
- **Required input:** photo selection + download action
- **Optional input:** size choice (`Small`, `Medium`, `Large`, `Original`)

### Output
- File download starts (JPEG at selected/available resolution)
- Download stats increment for selected photo
- For premium locked photo: subscription CTA / redirect to subscription page

### Workflow
1. User opens photo detail.
2. User clicks `Download free` (or download dropdown).
3. System checks photo tier (free vs Unsplash+) and availability.
4. If free: user selects size, system downloads file, increments count.
5. If Unsplash+: system checks active subscription before allowing full-quality download.

**Exception workflow**
- Deleted photo: disable download button and show `Photo no longer available`.

### Validation & Business Rules
- BR-011 to BR-020.
- Do not show size options larger than source resolution.

---

## F3. User Registration & Login

### Goal
Enable secure account creation and authentication.

### Input
- **Actor:** Guest user
- **Entry points:** `/join`, `/login`, OAuth buttons
- **Required registration fields:** first name, last name, email, username, password, TOS agreement
- **Required login fields:** email, password
- **Optional auth methods:** Google/Facebook OAuth

### Output
- Account created + verification email + authenticated session
- Successful login redirects to homepage or intended page
- Generic login error for invalid credentials
- Temporary lockout after repeated failures

### Workflow
**Registration**
1. User fills form and accepts legal terms.
2. System validates uniqueness + format + password policy.
3. On pass: create account, send confirmation email, log user in.

**Login**
1. User submits credentials.
2. System validates credentials and lock status.
3. On pass: establish session and redirect.
4. On fail: return generic message; apply lockout policy after threshold.

### Validation & Business Rules
- BR-021 to BR-035.
- Email regex, username constraints, password complexity enforced.

---

## F4. Photo Upload (Contributors)

### Goal
Allow contributors to submit high-quality, rights-compliant photos for review.

### Input
- **Actor:** Authenticated user/contributor
- **Entry point:** `/submit`
- **Required input:** image file
- **Optional metadata:** description, alt text, location, tags (≤30)

### Output
- Upload accepted with status `Under Review`
- Confirmation notification/email to contributor
- Admin moderation outcome: approved (published) or rejected (reason sent)

### Workflow
1. User selects file and submits metadata.
2. System validates file type/size/resolution.
3. On pass: store submission with review status.
4. Admin reviews for quality/compliance.
5. System publishes or rejects + notifies contributor.

### Validation & Business Rules
- BR-036 to BR-048.
- Format: JPEG/PNG only; max 50MB; min 5MP.

---

## F5. Collections Management

### Goal
Let users curate personal/public sets of photos.

### Input
- **Actor:** Authenticated user
- **Entry points:** photo card `Collect`, profile collections page
- **Required input:** collection name when creating
- **Optional input:** visibility (`Public`/`Private`), cover photo, reorder actions

### Output
- Collection created/updated/deleted
- Photo added/removed from collection
- Notifications sent to contributor when photo is collected

### Workflow
1. User opens collect action on photo.
2. User creates new collection or selects existing one.
3. System saves association and updates UI status.
4. User can manage collection details from profile.

### Validation & Business Rules
- BR-049 to BR-060.
- Duplicate add in same collection is blocked.

---

## F6. Like / Save Photos

### Goal
Enable engagement through one-click like/unlike.

### Input
- **Actor:** Authenticated user (guest redirected)
- **Entry points:** photo card, photo detail
- **Required input:** like toggle action

### Output
- Like state toggled, count updated
- Notification to contributor on like (not on unlike)
- Liked photos visible in user profile Likes tab

### Workflow
1. User clicks heart icon.
2. System checks authentication.
3. If authenticated: toggle like state and update count.
4. If guest: prompt login/signup.

### Validation & Business Rules
- BR-061 to BR-067.
- One like per user per photo; second click unlikes.

---

## F7. User Profile

### Goal
Provide a public identity page and editable personal account settings.

### Input
- **Actor:** Registered user (owner edit), guest/other users (view)
- **Entry points:** user profile URL, account settings
- **Required editable fields:** display name / username (as configured)
- **Optional fields:** bio, location, social links, website, avatar

### Output
- Public profile shows photos, likes, collections, stats
- Profile updates persist and reflect immediately
- Invalid update requests return inline validation errors

### Workflow
1. Viewer opens a profile page.
2. System returns public profile content based on visibility/privacy rules.
3. Owner enters edit mode and updates fields.
4. System validates and saves updates.

### Validation & Business Rules
- BR-068 to BR-080.
- Username uniqueness and format constraints remain enforced.

---

## F8. Unsplash+ Subscription

### Goal
Sell and manage premium plan access for paid photo content.

### Input
- **Actor:** Registered user
- **Entry points:** header CTA, locked photo CTA, billing settings
- **Required input (subscribe):** selected plan + payment details (Stripe)
- **Required input (cancel):** cancellation confirmation

### Output
- Active subscription on successful payment
- Access to premium downloads during active period
- Invoice records and billing status updates
- Cancellation applied at period end (no immediate cutoff unless policy changes)

### Workflow
**Subscribe**
1. User picks plan and proceeds to Stripe checkout.
2. Stripe processes payment.
3. On success: activate subscription + email confirmation.
4. On fail: return payment error and keep user unsubscribed.

**Cancel**
1. User opens billing page and confirms cancellation.
2. System disables auto-renew and keeps access until cycle end.

### Validation & Business Rules
- BR-081 to BR-090.
- Unsplash does not store raw card data.

---

## F9. Visual Search

### Goal
Allow image-based discovery using uploaded images or public image URLs.

### Input
- **Actor:** Any user
- **Entry points:** camera icon, visual search page
- **Required input:** image file or image URL
- **Constraints:** JPEG/PNG upload, max 20MB, URL publicly accessible

### Output
- Up to 20 visually similar photo results ranked by similarity
- Optional post-filtering by keyword
- Empty state when no similar results

### Workflow
1. User provides image file/URL.
2. System validates input type and accessibility.
3. Similarity engine processes image.
4. System returns result grid.
5. User optionally refines with keyword.

### Validation & Business Rules
- BR-091 to BR-097.
- Inaccessible URL returns explicit error message.

---

## F10. API Access (Developers)

### Goal
Provide controlled API integration with rate limits, attribution, and compliance controls.

### Input
- **Actor:** Developer account owner
- **Entry points:** `/developers`, API endpoints
- **Required input:** registered app + `Authorization: Client-ID {ACCESS_KEY}`
- **Optional input:** endpoint params (`query`, `page`, `per_page`, etc.)

### Output
- App credentials issued after app registration
- JSON API responses with pagination metadata
- Standard errors (`401`, `429`) for auth/rate-limit violations

### Workflow
1. Developer registers app and obtains keys.
2. Developer sends authorized API requests.
3. System validates key + mode (Demo/Production) + limits.
4. System returns data or standardized error response.

### Validation & Business Rules
- BR-098 to BR-108.
- Demo mode: 50 req/hour; production: up to 5,000 req/hour after approval.

---

## 4. Cross-Feature Constraints

- Authentication-required actions must consistently redirect or prompt login.
- All user-entered text fields must enforce length + trim + format constraints.
- Moderation and abuse rules apply across uploads, collections, profiles, and API usage.
- Analytics events (downloads, likes, collection actions) must be idempotent and auditable.

---

## 5. Appendix — Business Rule Index

- **Search & Discovery:** BR-001 → BR-010
- **Download:** BR-011 → BR-020
- **Registration/Login:** BR-021 → BR-035
- **Upload:** BR-036 → BR-048
- **Collections:** BR-049 → BR-060
- **Likes:** BR-061 → BR-067
- **Profile:** BR-068 → BR-080
- **Unsplash+:** BR-081 → BR-090
- **Visual Search:** BR-091 → BR-097
- **API:** BR-098 → BR-108

