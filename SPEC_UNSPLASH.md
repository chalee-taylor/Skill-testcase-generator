# SPEC_UNSPLASH.md — Feature Specification: Unsplash.com

**Version:** 2.0.0
**Author:** QC Team — Cook A Skill
**Last Updated:** 2026-02-25
**Target Site:** https://unsplash.com/
**Spec Type:** Product Feature Specification (PRD) — Optimized for AI Test Case Generation

---

## How to Use This Spec with the Test Case Generator Skill

> **This is Step 3 in the Cook-A-Skill workflow.** This spec is designed to be fed directly into the **Test Case Generator** skill (see `SPEC.md`). Each feature section below already includes complete Input / Output / Workflow details so the skill can immediately generate high-quality test cases.

### Input

| Field | Value | Note |
|---|---|---|
| `spec_content` | Copy one full feature section (from `## Feature:` through `Validation Rules`) | Process one feature at a time |
| `feature_name` | Feature name, for example: `"Photo Search & Discovery"` | Use the exact heading name |
| `output_format` | `markdown` or `json` or `csv` | Default: `markdown` |
| `coverage_level` | `basic` / `standard` / `full` | Recommended: `full` |
| `enable_security` | `true` | Always enabled for web apps |
| `mask_pii` | `true` | Always enabled — this spec contains no real PII |

### Output

| Component | Description |
|---|---|
| Test Case List | Test case list covering: Happy Path, Negative, Edge Case, Security |
| Traceability Matrix | Mapping BR-xxx → TC-xxx, including detection of rules without test coverage |
| Coverage Summary | Total number of test cases by category |
| Test Report Template | Template for recording execution results |

### Workflow

```
Step 1: Choose one feature from the Table of Contents below
         │
         ▼
Step 2: Copy the full section (heading + Input + Output + Workflow + BR + Validation)
         │
         ▼
Step 3: Paste it into `spec_content` of the Test Case Generator skill
         │
         ▼
Step 4: Set `coverage_level = full`, `enable_security = true`
         │
         ▼
Step 5: Run the skill → receive test cases
         │
         ▼
Step 6: Review output — refine the spec if needed, then re-run
         │
         ▼
Step 7: Export to Markdown / JSON / CSV for use in test sessions
```

### Pain Point Analysis

| Manual test-case writing pain point | How the skill solves it |
|---|---|
| Takes 2–4 hours to read spec, understand flow, and write test cases manually | Skill parses the spec and generates test cases in < 30 seconds |
| Easy to miss edge cases (boundary, empty, overflow) | Skill runs an automatic Edge Case Checklist |
| Security cases are often forgotten (SQLi, XSS, brute force) | Skill always generates a standard security test set |
| Inconsistent test-case format across QA engineers | Skill output follows a fixed, consistent schema |
| No traceability (which BRs are not tested?) | Skill automatically generates a Traceability Matrix |

---

## Table of Contents

1. [Product Overview](#1-product-overview)
2. [Feature: Photo Search & Discovery](#2-feature-photo-search--discovery)
3. [Feature: Photo Download](#3-feature-photo-download)
4. [Feature: User Registration & Login](#4-feature-user-registration--login)
5. [Feature: Photo Upload (Contributors)](#5-feature-photo-upload-contributors)
6. [Feature: Collections Management](#6-feature-collections-management)
7. [Feature: Like / Save Photos](#7-feature-like--save-photos)
8. [Feature: User Profile](#8-feature-user-profile)
9. [Feature: Unsplash+ Subscription](#9-feature-unsplash-subscription)
10. [Feature: Visual Search](#10-feature-visual-search)
11. [Feature: API Access (Developers)](#11-feature-api-access-developers)

---

## 1. Product Overview

**Product Name:** Unsplash
**Website:** https://unsplash.com/
**Type:** Free stock photo platform

### Description

Unsplash is a community-powered visual platform providing over 6 million high-resolution images contributed by 375,000+ photographers worldwide. Users can search, discover, and download photos for free for both commercial and noncommercial purposes. Photographers can upload their work to gain exposure. Premium content is available via the Unsplash+ subscription tier.

### Target Users

- Designers, developers, marketers, bloggers, content creators (Downloaders)
- Photographers and visual artists (Contributors)
- Brands and organizations (Brand accounts)
- Software developers (API consumers)

### Key Roles

| Role | Description |
|---|---|
| Guest (Unauthenticated) | Can browse and download free photos without an account |
| Registered User | Can like photos, create collections, upload photos, subscribe to Unsplash+ |
| Contributor | Registered user who has uploaded at least 1 photo |
| Unsplash+ Subscriber | Paid subscriber with access to premium photo library |
| Admin | Internal Unsplash team; moderates content and accounts |

---

## 2. Feature: Photo Search & Discovery

### Description

Users can search for photos by keyword using the search bar. Results include photos, collections, and user profiles. The homepage displays an infinite-scroll grid of curated photos. Discovery is supported through trending tags and themed category navigation.

### Input

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `keyword` | `string` | Yes (to search) | 1–200 characters; whitespace-only treated as empty | Search term entered by user |
| `result_tab` | `enum` | No | `Photos` \| `Collections` \| `Users`; default: `Photos` | Tab filter applied to results |
| `sort_order` | `enum` | No | `Relevance` (default) \| `Latest` | Sort order for results |
| `category` | `string` | No | One of the predefined category slugs in header nav | Filter by category (e.g., "Nature", "Travel") |
| `page` | `integer` | No | ≥ 1; auto-incremented on scroll | Pagination page number |
| `user_role` | `enum` | — | `Guest` \| `Registered User` | Auth state — both can search |

### Output

| Scenario | System Response |
|---|---|
| Valid keyword → results found | Grid of photos (Photos tab), plus Collections and Users tabs; tag suggestions shown at top; pagination auto-loads on scroll |
| Valid keyword → 0 results | Empty state: `"We couldn't find anything for [keyword]"` + suggested alternative search terms |
| Empty / whitespace keyword | Search does not trigger; cursor remains in search bar |
| Keyword > 200 characters | Input is silently truncated to 200 characters before submission |
| Category nav clicked | Photo grid filtered to that category; URL updates to category slug |
| User scrolls to bottom | Next batch of results loads automatically (infinite scroll) |
| Tab switched | Results filtered to Photos / Collections / Users respectively |

### Workflow

#### Happy Path — Keyword Search

1. User opens `https://unsplash.com/`
2. User types keyword (e.g., `"mountain"`) into the search bar
3. User presses Enter or clicks the search icon
4. System displays search results page with 3 tabs: **Photos** (active), **Collections**, **Users**
5. System shows tag suggestions relevant to the keyword at the top of the results
6. User can switch tabs to filter result type
7. User can click a tag suggestion to refine search
8. System auto-loads the next batch of photos when user scrolls to bottom of the page
9. User clicks a photo thumbnail → opens Photo Detail page

#### Alternative Flow — Category Navigation

1. User clicks a category link in the header (e.g., "Architecture")
2. System filters the homepage grid to photos tagged with that category
3. URL updates to `https://unsplash.com/s/photos/{category}`

#### Alternative Flow — Sort Results

1. On the search results page, user selects "Latest" from the sort dropdown
2. System re-fetches results sorted by upload date descending

### Business Rules

- **BR-001:** Search bar is accessible on all pages (header navigation) — for both authenticated and unauthenticated users
- **BR-002:** Search results are filtered into 3 tabs: Photos, Collections, Users
- **BR-003:** Each search result page displays a set of relevant tag suggestions at the top
- **BR-004:** The homepage features an infinite-scroll photo grid — new photos load automatically when user scrolls to the bottom
- **BR-005:** Search results return at most 20 photos per page by default; pagination is supported
- **BR-006:** If search returns 0 results, display empty-state message: `"We couldn't find anything for [keyword]"` with suggested alternatives
- **BR-007:** Trending/popular searches are displayed below the search bar on the homepage
- **BR-008:** Category navigation links filter the photo grid by category
- **BR-009:** Users can sort search results by: Relevance (default) or Latest
- **BR-010:** Search keyword must be between 1 and 200 characters

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| Keyword is whitespace only (e.g., `"   "`) | Treated as empty — search does not trigger |
| Keyword is exactly 200 characters | Accepted and submitted normally |
| Keyword is 201+ characters | Truncated to 200 characters silently |
| Special characters in keyword (e.g., `"#nature"`, `"@user"`) | Allowed — used for tag/mention searches |
| Unicode / emoji in keyword (e.g., `"🌊 ocean"`) | Accepted and processed |
| No results for keyword | Empty state message + suggestions displayed |
| Rapid search submissions (type fast + Enter) | Only latest request is processed; previous requests cancelled |
| Scroll-triggered load fails (network error) | Spinner shown; retry button appears |

---

## 3. Feature: Photo Download

### Description

Users can download free photos without creating an account. Each photo has a "Download free" button. Premium photos (Unsplash+) require an active subscription. Downloaded photos are provided in high resolution.

### Input

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `photo_id` | `string` | Yes | Valid Unsplash photo ID | The photo to download |
| `size` | `enum` | Yes | `Small` \| `Medium` \| `Large` \| `Original` | Selected download resolution |
| `user_role` | `enum` | — | `Guest` \| `Registered User` \| `Unsplash+ Subscriber` | Determines access to premium photos |
| `photo_type` | `enum` | — | `Free` \| `Unsplash+ Premium` | Determines download gate |

### Output

| Scenario | System Response |
|---|---|
| Free photo, any user selects size | File download initiated in browser; download count increments by 1 |
| Free photo, Original size selected | Highest-resolution JPEG available for that photo downloaded |
| Premium photo, non-subscriber clicks download | Lock icon shown; user redirected to `https://unsplash.com/plus` |
| Premium photo, active subscriber | Download initiated without watermark; JPEG at full quality |
| Photo has been deleted | Download button disabled; message: `"Photo no longer available"` |
| Size option exceeds original resolution | That size option is not shown in the dropdown |

### Workflow

#### Happy Path — Free Photo Download

1. User finds a photo via search or browsing
2. User clicks photo thumbnail → Photo Detail page opens
3. System displays photo in full view with: photographer info, stats (downloads, likes, views), EXIF data
4. User clicks `"Download free"` button (or the dropdown arrow)
5. System displays size options: **Small**, **Medium**, **Large**, **Original**
6. User selects desired size
7. System initiates JPEG file download to user's device
8. Download count on the photo increments by 1

#### Alternative Flow — Premium Photo (Active Subscriber)

1. User clicks on a photo with a lock icon overlay
2. System displays: `"This photo requires Unsplash+"` with a `"Subscribe"` CTA button
3. System checks subscription status → active subscription confirmed
4. Download proceeds at full quality without watermark

#### Alternative Flow — Premium Photo (No Subscription)

1. User clicks on a photo with a lock icon overlay
2. System displays: `"This photo requires Unsplash+"` with a `"Subscribe"` CTA button
3. System checks subscription status → no active subscription found
4. User is redirected to the Unsplash+ subscription page (`https://unsplash.com/plus`)

### Business Rules

- **BR-011:** Guest users (unauthenticated) can download free photos without an account
- **BR-012:** Free photo downloads do not require attribution (attribution to photographer is encouraged)
- **BR-013:** Download button labeled `"Download free"` for free photos; lock icon shown for Unsplash+ premium photos
- **BR-014:** Users can select download size: Small, Medium, Large, or Original (for free photos)
- **BR-015:** Unsplash+ photos require an active Unsplash+ subscription to download without watermark
- **BR-016:** Downloading a photo triggers a download count increment on the photo's stats
- **BR-017:** Downloaded photos are provided in JPEG format at the selected resolution
- **BR-018:** The platform tracks download events for analytics (no user account linkage for guest downloads)
- **BR-019:** Users cannot resell downloaded photos as-is on competing stock photo platforms
- **BR-020:** Users cannot redistribute downloaded photos as part of a competing stock photo service

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| Photo deleted by contributor | Download button disabled; `"Photo no longer available"` shown |
| Requested size exceeds original resolution | That size option not shown in dropdown |
| Download initiated but network drops mid-transfer | Browser handles partial download; user can retry |
| Guest downloads same free photo multiple times | All downloads succeed; each increments download count |
| Subscriber's subscription expires mid-session | On next premium download attempt, subscription gate is re-checked |
| User clicks download icon without selecting size | System defaults to Original or prompts size selection |

---

## 4. Feature: User Registration & Login

### Description

Users can create an account with email/password or via Google/Facebook OAuth. Registration unlocks additional features: liking photos, creating collections, uploading photos, and accessing Unsplash+.

### Input

#### Registration

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `first_name` | `string` | Yes | 1–50 characters | User's first name |
| `last_name` | `string` | Yes | 1–50 characters | User's last name |
| `email` | `string` | Yes | Valid email format; unique across platform | Account email address |
| `username` | `string` | Yes | 3–30 chars; alphanumeric + underscore only; unique | Public-facing username |
| `password` | `string` | Yes | Min 8 chars; must include uppercase, lowercase, digit | Account password |
| `tos_accepted` | `boolean` | Yes | Must be `true` | Agreement to Terms of Service and Privacy Policy |

#### Login

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `email` | `string` | Yes | Valid email format | Registered account email |
| `password` | `string` | Yes | Non-empty | Account password |

### Output

#### Registration Output

| Scenario | System Response |
|---|---|
| All fields valid + ToS accepted | Account created; verification email sent; user redirected to homepage (logged in) |
| Email already registered | Inline error on email field: generic error (does not reveal email exists — see BR-033) |
| Username already taken | Inline error on username field: `"Username is already taken"` |
| Password too weak | Inline error on password field with specific requirement not met |
| ToS not accepted | Submit blocked; checkbox highlighted |
| Any required field empty | Inline validation error on that field |

#### Login Output

| Scenario | System Response |
|---|---|
| Valid email + correct password | Redirect to homepage or previously attempted page |
| Invalid email or wrong password | Error message: `"Invalid email or password"` (generic — never reveals which is wrong) |
| Account locked (5 failed attempts) | Message: `"Your account has been temporarily locked. Please try again in 15 minutes."` |
| OAuth login success | Account created or linked; redirect to homepage (logged in) |

### Workflow

#### Happy Path — Registration

1. User navigates to `https://unsplash.com/join`
2. User fills in: First Name, Last Name, Email, Username, Password
3. User checks the ToS/Privacy Policy agreement checkbox
4. User clicks `"Join"`
5. System validates all fields (client-side + server-side)
6. On success: Account created; verification email sent; user is redirected to homepage as logged-in user
7. On failure: Specific inline validation errors shown on offending fields

#### Happy Path — Login

1. User navigates to `https://unsplash.com/login`
2. User enters Email and Password
3. User clicks `"Login"`
4. System validates credentials
5. On success: Redirect to homepage or previously attempted page
6. On failure: Generic error message `"Invalid email or password"` displayed

#### Alternative Flow — Account Lockout

1. User fails login 5 consecutive times
2. System locks the account for 15 minutes
3. System displays lockout message with time remaining

#### Alternative Flow — OAuth Login (Google)

1. User clicks `"Continue with Google"` on login or join page
2. System redirects to Google OAuth consent screen
3. User grants permission
4. System creates or links the account; user redirected to homepage (logged in)

### Business Rules

- **BR-021:** Email must be a valid format (RFC 5321 compliant)
- **BR-022:** Password must be at least 8 characters long
- **BR-023:** Password must contain at least one uppercase letter, one lowercase letter, and one number
- **BR-024:** Email address must be unique — no two accounts can share the same email
- **BR-025:** Username must be unique across the platform
- **BR-026:** Username must be between 3 and 30 characters
- **BR-027:** Username can only contain letters, numbers, and underscores — no spaces, special characters, or URLs
- **BR-028:** Username cannot be a trademarked brand name
- **BR-029:** Username cannot impersonate another person or public figure
- **BR-030:** After 5 consecutive failed login attempts, account is temporarily locked for 15 minutes
- **BR-031:** Users can sign up/login via Google OAuth or Facebook OAuth
- **BR-032:** Upon successful registration, system sends a confirmation email to the registered address
- **BR-033:** System must not reveal whether an email address is registered in login error messages (anti-enumeration)
- **BR-034:** Login error message must be generic: `"Invalid email or password"`
- **BR-035:** Users must agree to Terms of Service and Privacy Policy before completing registration

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| Email: `"  user@test.com  "` (leading/trailing spaces) | Trimmed before validation; treated as `user@test.com` |
| Email: `"user@"` (malformed) | Inline error: `"Please enter a valid email address"` |
| Password: exactly 8 characters | Accepted |
| Password: 7 characters | Rejected: `"Password must be at least 8 characters"` |
| Password: 8 chars but no digit (e.g., `"Password"`) | Rejected: missing digit requirement |
| Username: `"ab"` (2 chars) | Rejected: below 3-char minimum |
| Username: 31 characters | Rejected: exceeds 30-char maximum |
| Username: `"hello world"` (contains space) | Rejected: spaces not allowed |
| Username: `"__admin__"` (leading/trailing underscores) | Rejected per format rules |
| Login: rapid 6th attempt immediately after lockout | Blocked until 15-minute window expires |
| OAuth account + same email as existing password account | System links accounts or prompts user to choose |
| Registration with ToS unchecked | Submit button inactive or server returns 400 |

---

## 5. Feature: Photo Upload (Contributors)

### Description

Registered users can submit photos to Unsplash. Uploaded photos are reviewed against submission guidelines. Approved photos become part of the public library under the Unsplash license.

### Input

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `photo_file` | `file` | Yes | JPEG or PNG; max 50 MB; min 5 megapixels (≈ 2500×2000 px) | The photo to upload |
| `description` | `string` | No | Max 500 characters | Photo description |
| `alt_text` | `string` | No | Max 500 characters | Accessibility alt text |
| `location` | `string` | No | Max 100 characters | Location where photo was taken |
| `tags` | `list[string]` | No | Max 30 tags; each tag max 50 chars; no duplicates | Tags for discoverability |
| `user_role` | `enum` | — | Must be `Registered User` | Only authenticated users can upload |

### Output

| Scenario | System Response |
|---|---|
| File valid (format, size, resolution) | Preview displayed; optional fields shown; user can add metadata and submit |
| File invalid format (not JPEG/PNG) | Error: `"Only JPEG and PNG files are accepted"` |
| File size > 50 MB | Error: `"File size exceeds the 50 MB limit"` |
| Resolution < 5 megapixels | Error: `"Image resolution must be at least 5 megapixels (2500 × 2000 px minimum)"` |
| Submission successful | Photo uploaded with status `"Under Review"` on contributor's profile; confirmation notification sent |
| Daily limit reached (10 photos/day) | Upload blocked: `"You've reached your daily upload limit of 10 photos"` |
| Photo approved by admin | Photo published to public library; status changes to `"Published"` |
| Photo rejected by admin | Rejection email sent; photo removed from profile |
| Guest tries to upload | Redirect to login/join page |

### Workflow

#### Happy Path — Photo Upload

1. Authenticated user clicks `"Submit a photo"` in the top-right navigation
2. User is redirected to `https://unsplash.com/submit`
3. User selects a photo file from their device
4. System validates: file format (JPEG/PNG), file size (≤ 50 MB), resolution (≥ 5 megapixels)
5. On validation pass: System shows a preview of the photo
6. User fills in optional fields: Description, Alt Text, Location, Tags (up to 30)
7. User clicks `"Submit"` to confirm
8. System uploads the photo and sets its status to `"Under Review"`
9. System sends a confirmation notification to the contributor

#### Alternative Flow — Admin Approval

1. Admin reviews the submitted photo
2. Admin approves the photo
3. System publishes the photo to the public library; status changes to `"Published"`

#### Alternative Flow — Admin Rejection

1. Admin reviews the submitted photo and determines it violates submission guidelines
2. System sends an explanatory rejection email to the contributor
3. Photo is removed from the contributor's profile

### Business Rules

- **BR-036:** Only registered (authenticated) users can upload photos
- **BR-037:** Uploaded photos must have a minimum resolution of 5 megapixels (e.g., 2500 × 2000 pixels minimum)
- **BR-038:** Accepted file formats: JPEG and PNG only
- **BR-039:** Maximum file size per upload: 50 MB
- **BR-040:** Users can upload up to 10 photos per day (daily limit for non-verified accounts)
- **BR-041:** By uploading, contributors grant Unsplash and users a royalty-free, irrevocable license to use the photo
- **BR-042:** Contributors must own the rights to the uploaded image or have permission from the rights holder
- **BR-043:** Photos must not contain: nudity, violence, hate content, PII of individuals without consent, copyrighted logos or trademarked brand elements
- **BR-044:** Photos depicting identifiable people in editorial or commercial contexts require a valid model release
- **BR-045:** Submitted photos are subject to review — non-compliant photos are rejected with an explanatory email
- **BR-046:** Upon successful upload, photo appears on contributor's profile with status `"Under Review"` until approved
- **BR-047:** Contributors can add a description, alt text, and location to their uploaded photos
- **BR-048:** Contributors can add up to 30 tags per photo for discoverability

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| File is JPEG with `.png` extension | Validated by actual MIME type — accepted if truly JPEG |
| File is GIF / WEBP / HEIC | Rejected: `"Only JPEG and PNG files are accepted"` |
| File is exactly 50 MB | Accepted |
| File is 50.01 MB | Rejected: `"File size exceeds the 50 MB limit"` |
| Resolution is exactly 5 megapixels (e.g., 2500×2000) | Accepted |
| Resolution is 4.9 megapixels | Rejected: below minimum |
| 31 tags submitted | Error: `"Maximum 30 tags allowed"` |
| Duplicate tag in same upload (e.g., `"sunset"`, `"sunset"`) | Duplicate silently de-duplicated or error shown |
| Tag longer than 50 characters | Error: `"Tag cannot exceed 50 characters"` |
| Upload attempt on 11th photo (daily limit hit) | Blocked: `"Daily upload limit of 10 photos reached"` |
| File upload interrupted mid-stream | Error shown; upload not saved; user can retry |
| Guest attempts to access `/submit` directly | Redirect to `/login` |

---

## 6. Feature: Collections Management

### Description

Registered users can create collections to organize and save photos. Collections can be public or private. Other users can follow public collections.

### Input

#### Create Collection

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `name` | `string` | Yes | 1–60 characters; cannot be whitespace-only | Collection name |
| `visibility` | `enum` | Yes | `Public` \| `Private` | Who can see the collection |
| `photo_id` | `string` | No | Valid Unsplash photo ID | First photo to add (optional at creation) |
| `user_role` | `enum` | — | Must be `Registered User` | Only authenticated users can create collections |

#### Add Photo to Collection

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `photo_id` | `string` | Yes | Valid Unsplash photo ID | Photo to add |
| `collection_id` | `string` | Yes | Must belong to the authenticated user | Target collection |

### Output

| Scenario | System Response |
|---|---|
| Collection created successfully | Collection created; first photo added (if provided); success message: `"Photo added to [Collection Name]"` |
| Photo added to existing collection | Checkmark appears on the collection in dropdown; contributor notified |
| Photo already in collection | Silently prevented; system shows `"Already in this collection"` |
| Collection deleted | Collection and all associations removed; photos remain on Unsplash; confirmation dialog shown first |
| Guest tries to collect a photo | Login/signup prompt: `"Sign up to collect photos"` |
| Collection name is empty or whitespace | Error: collection name is required |

### Workflow

#### Happy Path — Create a New Collection

1. Authenticated user views a photo they want to save
2. User clicks the `"Collect"` (bookmark) icon on the photo
3. System displays a dropdown: existing collections with checkmarks + `"Create new collection"` option
4. User clicks `"Create new collection"`
5. User enters collection name and selects Public or Private
6. User clicks `"Create"`
7. System creates the collection and adds the current photo to it
8. System displays: `"Photo added to [Collection Name]"`

#### Happy Path — Add Photo to Existing Collection

1. Authenticated user views a photo
2. User clicks the `"Collect"` icon
3. System displays dropdown list of user's collections; collections already containing this photo show a checkmark
4. User clicks a collection (without checkmark)
5. System adds the photo; checkmark appears; photo contributor receives a notification

#### Alternative Flow — Delete a Collection

1. User navigates to their profile → Collections tab
2. User selects the collection to delete
3. User clicks `"Delete collection"`
4. System shows confirmation: `"Are you sure you want to delete [Collection Name]? This cannot be undone."`
5. User confirms
6. System deletes the collection and all photo associations; photos remain on Unsplash

### Business Rules

- **BR-049:** Only authenticated users can create collections
- **BR-050:** A user can create an unlimited number of collections
- **BR-051:** Collection names must be between 1 and 60 characters
- **BR-052:** Collections can be set as Public (visible to all) or Private (visible only to the owner)
- **BR-053:** Users can add any photo (free or Unsplash+) to their collections without a subscription
- **BR-054:** A user cannot add the same photo to the same collection twice (duplicate prevention)
- **BR-055:** The photo contributor receives a notification when their photo is added to another user's collection
- **BR-056:** Collections containing offensive content are removed by admins; email sent to collection creator
- **BR-057:** Users can delete their collections — deleting a collection does not delete the photos
- **BR-058:** Users can reorder photos within a collection
- **BR-059:** A collection displays a cover photo (default: first photo added; user can change)
- **BR-060:** Other users can follow public collections to receive updates

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| Collection name: 1 character | Accepted |
| Collection name: 60 characters | Accepted |
| Collection name: 61 characters | Rejected: `"Collection name cannot exceed 60 characters"` |
| Collection name: `"   "` (whitespace only) | Rejected: cannot be empty or whitespace-only |
| Adding same photo to same collection twice | System shows `"Already in this collection"`; no duplicate created |
| Deleting a collection that others follow | Collection deleted; followers lose the collection from their following list |
| Private collection URL accessed by another user | 404 or access denied |
| Reordering photos: drag-and-drop during concurrent session | Last save wins; no data corruption |
| Changing cover photo to a photo not in the collection | Blocked; only photos already in the collection can be set as cover |

---

## 7. Feature: Like / Save Photos

### Description

Authenticated users can "like" (heart) photos to show appreciation to photographers and to save them to a personal likes list.

### Input

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `photo_id` | `string` | Yes | Valid Unsplash photo ID | The photo to like |
| `action` | `enum` | — | `like` \| `unlike` (toggle) | Whether to add or remove the like |
| `user_role` | `enum` | — | Must be `Registered User` | Guests cannot like |

### Output

| Scenario | System Response |
|---|---|
| Like added (authenticated user) | Like count increments by 1; heart icon fills (red); photo appears in user's `Likes` tab; contributor notified |
| Unlike (user clicks filled heart again) | Like count decrements by 1; heart icon becomes hollow; no notification sent |
| Guest clicks like button | Login/signup prompt: `"Sign up to like this photo"` |
| User likes a photo they already liked | System ignores the action (idempotent) or shows heart already filled |

### Workflow

#### Happy Path — Like

1. Authenticated user hovers over or opens a photo
2. User clicks the heart icon (Like button)
3. System increments the like count by 1 and highlights the heart icon (red/filled)
4. Photo contributor receives notification: `"[Username] liked your photo"`
5. The liked photo appears in the user's profile under `"Likes"` tab

#### Alternative Flow — Unlike

1. User clicks the filled heart icon again on a previously liked photo
2. System decrements the like count by 1 and shows the hollow heart icon
3. No notification is sent to the contributor

#### Alternative Flow — Unauthenticated User

1. Guest user clicks the heart icon
2. System displays: `"Sign up to like this photo"`
3. Guest is redirected to the login/join page

### Business Rules

- **BR-061:** Only authenticated users can like a photo
- **BR-062:** A user can like any photo (free or Unsplash+) without a subscription
- **BR-063:** A user can only like a photo once — the like button toggles (click again to unlike)
- **BR-064:** The photo contributor receives a notification when their photo is liked
- **BR-065:** A user's liked photos are visible on their profile under the `"Likes"` tab (public by default)
- **BR-066:** The like count on each photo is displayed publicly
- **BR-067:** Unlike does not notify the contributor

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| User likes a photo then logs out and logs back in | Like state is preserved; heart still filled |
| User rapidly clicks like/unlike toggle (double-click) | Final state is what counts; no race condition; count accurate |
| User likes own uploaded photo | Technically allowed; no self-notification sent |
| Photo is deleted after user liked it | Liked photo removed from user's Likes tab; no broken reference |
| Guest navigates directly to a liked photo URL | No like state shown; heart icon available but triggers login prompt on click |

---

## 8. Feature: User Profile

### Description

Every registered user has a public profile page showing their uploaded photos, liked photos, and collections. Users can edit their profile information.

### Input

#### View Profile

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `username` | `string` | Yes | Valid registered username | Whose profile to view |
| `tab` | `enum` | No | `Photos` (default) \| `Likes` \| `Collections` | Which tab to display |
| `user_role` | `enum` | — | `Guest` \| `Registered User` | Both can view public profiles |

#### Edit Profile

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `first_name` / `last_name` | `string` | Yes | 1–50 characters each | Display name |
| `username` | `string` | Yes | 3–30 chars; alphanumeric + underscore; unique; max 1 change per 30 days | Public username |
| `bio` | `string` | No | Max 200 characters | Short biography |
| `location` | `string` | No | Max 100 characters | User's location |
| `website` | `string` | No | Must start with `http://` or `https://` | Portfolio or personal website |
| `portfolio_url` | `string` | No | Valid URL format | Portfolio URL |
| `social_twitter` | `string` | No | Twitter/X handle (without @) | Social link |
| `social_instagram` | `string` | No | Instagram handle (without @) | Social link |
| `avatar` | `file` | No | JPEG or PNG; max 5 MB; minimum 200×200 px | Profile picture |

### Output

| Scenario | System Response |
|---|---|
| View valid profile URL | Public profile page with avatar, name, bio, location, website, social links, follower/following count, stats |
| View non-existent username | 404 page |
| Edit profile — all fields valid | Profile updated; user redirected to their profile page |
| Edit profile — validation error | Inline errors on offending fields |
| Username change within 30 days | Error: `"You can only change your username once every 30 days"` |
| Avatar format/size invalid | Error on avatar field |

### Workflow

#### Happy Path — View Profile

1. User navigates to `https://unsplash.com/@{username}`
2. System displays the public profile page (Photos tab active by default)
3. User can switch between Photos, Likes, Collections tabs

#### Happy Path — Edit Profile

1. Authenticated user clicks their avatar → `"Edit profile"`
2. System displays the profile edit form with current values pre-filled
3. User updates desired fields
4. User clicks `"Save changes"`
5. System validates all fields (client-side + server-side)
6. On success: Profile updated; user redirected to their profile page
7. On failure: Specific inline validation errors displayed

### Business Rules

- **BR-068:** Profile URL format: `https://unsplash.com/@{username}`
- **BR-069:** Profile page is publicly visible (no login required to view)
- **BR-070:** Profile displays: avatar, name, bio, location, website, social links, followers/following count
- **BR-071:** Profile tabs: Photos, Likes, Collections
- **BR-072:** Users can edit: name, username, bio, location, website, portfolio URL, social links
- **BR-073:** Username changes are allowed but limited to once per 30 days
- **BR-074:** Bio max length: 200 characters
- **BR-075:** Website URL must be a valid URL format (starting with `http://` or `https://`)
- **BR-076:** Users can upload an avatar image (JPEG or PNG, max 5 MB, minimum 200×200 px)
- **BR-077:** Profile stats visible: total photos uploaded, total downloads received, total views received
- **BR-078:** Admins can suspend or deactivate accounts violating community guidelines
- **BR-079:** Inactive accounts with 0 photos may have their username reclaimed after 12 months of inactivity
- **BR-080:** Inactive accounts with 1+ photos may have their username reclaimed after 24 months of inactivity

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| Username: `"ab"` (2 chars) | Rejected |
| Username: `"user name"` (space) | Rejected |
| Username change on day 29 after last change | Rejected: `"You can only change your username once every 30 days"` |
| Username change on day 30 | Accepted |
| Bio: exactly 200 characters | Accepted |
| Bio: 201 characters | Rejected or truncated |
| Website: `"unsplash.com"` (no protocol) | Rejected: must start with `http://` or `https://` |
| Avatar: 199×200 px (one dimension below minimum) | Rejected: `"Avatar must be at least 200×200 pixels"` |
| Avatar: 5 MB exactly | Accepted |
| Avatar: 5.1 MB | Rejected: `"Avatar cannot exceed 5 MB"` |
| Suspended user visits their own profile | Account suspended notice shown; profile not publicly visible |

---

## 9. Feature: Unsplash+ Subscription

### Description

Unsplash+ is a premium subscription tier that gives users access to exclusive licensed photos not available for free. Unsplash+ photos are marked with a lock icon.

### Input

#### Subscribe

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `plan` | `enum` | Yes | `Monthly` \| `Annual` | Billing plan selected |
| `payment_card` | `object` | Yes | Card number (Luhn-valid), expiry (MM/YY), CVV, billing name | Payment details via Stripe |
| `billing_address` | `object` | Yes | All fields required for subscription activation | Billing address |
| `user_role` | `enum` | — | Must be `Registered User` | Guests must register before subscribing |

#### Cancel Subscription

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `user_role` | `enum` | — | Must be active `Unsplash+ Subscriber` | Only active subscribers can cancel |

### Output

| Scenario | System Response |
|---|---|
| Subscription activated (payment success) | Unsplash+ access unlocked; confirmation email with billing details sent; redirected to library |
| Payment fails (declined card) | Error: `"Payment failed — please check your card details"`; no subscription created |
| Subscription cancelled | Auto-renewal cancelled; access retained until end of current billing period; cancellation confirmation email sent |
| Premium download after subscription expires | Download gate shown; user redirected to Unsplash+ CTA |
| Non-subscriber attempts premium download | Lock icon shown; redirect to subscription page |
| Invoice download from billing settings | PDF invoice downloaded |

### Workflow

#### Happy Path — Subscribe

1. User clicks `"Get Unsplash+"` (in header or on a locked photo)
2. System displays subscription plans page (Monthly vs. Annual pricing)
3. User selects a plan and clicks `"Subscribe"`
4. System redirects to Stripe checkout page
5. User enters payment details (card number, expiry, CVV, billing address)
6. User confirms payment
7. Stripe processes the payment
8. On success: Unsplash+ subscription activated; confirmation email sent; user redirected to premium library
9. On failure: Stripe error returned; system displays `"Payment failed — please check your card details"`

#### Alternative Flow — Cancel Subscription

1. Authenticated Unsplash+ subscriber goes to Account Settings → Billing
2. User clicks `"Cancel subscription"`
3. System displays: `"Your Unsplash+ access will remain active until [billing cycle end date]. Are you sure?"`
4. User confirms
5. System cancels auto-renewal; user retains access until end of current billing period
6. System sends cancellation confirmation email

### Business Rules

- **BR-081:** Unsplash+ is a paid subscription with monthly or annual billing option
- **BR-082:** Unsplash+ photos are identified by a lock icon overlay on the thumbnail
- **BR-083:** Only active Unsplash+ subscribers can download premium (locked) photos without watermark
- **BR-084:** A non-subscriber clicking a locked photo is redirected to the Unsplash+ subscription page
- **BR-085:** Unsplash+ subscription is auto-renewed at the end of each billing cycle unless cancelled
- **BR-086:** Users can cancel the Unsplash+ subscription at any time; access remains until the end of the current billing period
- **BR-087:** Unsplash+ photos cannot be downloaded after subscription expires — access reverts to free content only
- **BR-088:** Payment is processed securely via Stripe; Unsplash does not store raw card data
- **BR-089:** Upon successful subscription activation, the user receives a confirmation email with billing details
- **BR-090:** Users can view and download their invoices from the billing settings page

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| Card number fails Luhn check | Rejected by Stripe before charge; error shown immediately |
| Expired card (expiry date in past) | Rejected: `"Your card has expired"` |
| Billing address missing required field | Error on that field; submission blocked |
| User subscribes then immediately cancels | Access active until billing period end; no refund shown |
| Subscription expires while user is mid-download of premium photo | Download allowed to complete; gate enforced on next premium download |
| User subscribes on Annual plan, then tries to change to Monthly mid-cycle | Plan change applied at next billing cycle |
| Invoice PDF for month where no charge occurred | Invoice shows $0.00 (e.g., during trial period if applicable) |
| Stripe webhook delayed (subscription active but system not updated) | System retries webhook; access granted once confirmed |

---

## 10. Feature: Visual Search

### Description

Unsplash provides a Visual Search tool that allows users to search by image rather than keyword. Users can upload an image or paste a URL to find visually similar photos.

### Input

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `image_file` | `file` | One of file/URL required | JPEG or PNG; max 20 MB | Reference image to search by |
| `image_url` | `string` | One of file/URL required | Must be publicly accessible URL | URL of reference image |
| `keyword_filter` | `string` | No | Optional; 1–200 characters | Keyword to further refine visual results |
| `user_role` | `enum` | — | `Guest` \| `Registered User` | Both can use visual search |

### Output

| Scenario | System Response |
|---|---|
| Valid image uploaded / URL provided | Grid of up to 20 visually similar photos, ranked by visual similarity score |
| No similar photos found | Message: `"No similar photos found. Try a different image."` |
| Image exceeds 20 MB | Error: `"Image must be 20 MB or smaller"` |
| Invalid file format (not JPEG/PNG) | Error: `"Only JPEG and PNG images are supported"` |
| URL not publicly accessible | Error: `"Image URL is not accessible. Please try uploading the image directly."` |
| Keyword filter applied after visual search | Results narrowed by both visual similarity and keyword relevance |

### Workflow

#### Happy Path — Upload Image

1. User navigates to Visual Search page or clicks the camera icon in the search bar
2. User uploads a reference image from their device
3. System processes the image using image similarity analysis (AI/ML engine)
4. System displays up to 20 visually similar photos in a grid
5. User can click any result to open the Photo Detail page and download
6. User can type a keyword to further filter the visual search results

#### Happy Path — URL-Based Search

1. User clicks the camera icon and selects `"Paste image URL"` option
2. User pastes a publicly accessible image URL
3. System fetches and processes the image
4. System displays up to 20 visually similar photos

### Business Rules

- **BR-091:** Visual Search is available to all users (authenticated and unauthenticated)
- **BR-092:** Users can initiate Visual Search by uploading an image (JPEG, PNG) or by entering a public image URL
- **BR-093:** Uploaded images for Visual Search must be at most 20 MB
- **BR-094:** Visual Search returns the top 20 most visually similar photos from the Unsplash library
- **BR-095:** Visual Search results can be further refined by keyword after the initial image-based search
- **BR-096:** Visual Search is powered by an AI/ML image similarity engine — results ranked by visual similarity score
- **BR-097:** If no visually similar photos are found, system displays: `"No similar photos found. Try a different image."`

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| Uploaded image: exactly 20 MB | Accepted |
| Uploaded image: 20.1 MB | Rejected: `"Image must be 20 MB or smaller"` |
| Image URL requires authentication (behind login wall) | Error: `"Image URL is not accessible. Please try uploading the image directly."` |
| Image URL resolves to a non-image file (e.g., a PDF) | Error: `"URL does not point to a valid image"` |
| Very abstract/plain image (solid color block) | Processed normally; results may be visually generic |
| Visual search + keyword filter returns 0 results | Empty state message shown for combined filters |
| Same image uploaded twice in same session | Second search runs normally; results may differ if library updated |

---

## 11. Feature: API Access (Developers)

### Description

Unsplash provides a public REST API for developers to integrate Unsplash photos into applications. API access requires registration and app approval.

### Input

#### API Registration

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `app_name` | `string` | Yes | 3–100 characters | Name of the developer's application |
| `description` | `string` | Yes | 20–500 characters | Description of how the API will be used |
| `redirect_uris` | `list[string]` | Yes | Valid URL format; `localhost` permitted for development | OAuth redirect URIs |
| `api_terms_accepted` | `boolean` | Yes | Must be `true` | Agreement to API Terms of Use |

#### API Photo Search Request

| Element | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `Authorization` | `header` | Yes | `Client-ID {ACCESS_KEY}` format | API key authentication |
| `query` | `query param` | Yes | Non-empty string | Search keyword |
| `page` | `query param` | No | Integer ≥ 1; default: 1 | Result page number |
| `per_page` | `query param` | No | Integer 1–30; default: 10 | Results per page |

### Output

| Scenario | System Response |
|---|---|
| Valid API request (within rate limit) | HTTP 200 with JSON response: photo results array, total count, pagination info |
| Missing or invalid Authorization header | HTTP 401 Unauthorized |
| Rate limit exceeded (Demo: 50 req/hr; Prod: 5,000 req/hr) | HTTP 429 with header `X-Ratelimit-Remaining: 0` |
| `per_page` out of range (< 1 or > 30) | HTTP 400 Bad Request |
| `page` ≤ 0 (negative or zero) | HTTP 400 Bad Request |
| App violates API terms | API keys revoked; HTTP 403 on subsequent requests |
| New app created in Demo mode | Access Key and Secret Key issued; rate limit set to 50 req/hr |
| Production approval granted | Rate limit upgraded to 5,000 req/hr |

### Workflow

#### Happy Path — API App Registration

1. Developer navigates to `https://unsplash.com/developers`
2. Developer clicks `"Register as a developer"`
3. System prompts login or registration (if not authenticated)
4. Developer clicks `"New Application"` and fills in: App Name, Description, Redirect URIs; agrees to API Terms
5. System creates the application and provides **Access Key** and **Secret Key**
6. Developer uses the Access Key in all API requests (Demo mode: 50 req/hr)

#### Happy Path — API Photo Search

1. Developer sends `GET /search/photos?query={keyword}&page=1&per_page=10`
2. Request includes header: `Authorization: Client-ID {ACCESS_KEY}`
3. System validates the access key
4. System returns JSON response: photo results array, total count, pagination info
5. Developer parses response and renders photos in their application

#### Alternative Flow — Rate Limit Exceeded

1. Developer has made 50 requests within the current hour (Demo mode)
2. On the 51st request, system returns HTTP 429 with `X-Ratelimit-Remaining: 0`
3. Developer must wait until the next hourly window to resume API calls

### Business Rules

- **BR-098:** To use the API, developers must register a free Unsplash developer account
- **BR-099:** Each app starts in Demo mode with a rate limit of 50 requests per hour
- **BR-100:** Production API access requires an application review and approval by the Unsplash team
- **BR-101:** Production rate limit: up to 5,000 requests per hour
- **BR-102:** API responses for list endpoints return 10 items per page by default; max 30 items per page
- **BR-103:** API calls must include a valid `Authorization: Client-ID {ACCESS_KEY}` header
- **BR-104:** Unauthenticated API calls return HTTP 401 Unauthorized
- **BR-105:** Rate-limited API calls return HTTP 429 Too Many Requests with header `X-Ratelimit-Remaining: 0`
- **BR-106:** API applications must display attribution (`"Photo by [Photographer Name] on Unsplash"`) for each photo used
- **BR-107:** API applications must call the Unsplash download endpoint to trigger a download event when a user downloads/uses a photo
- **BR-108:** Applications violating API terms may have their API keys revoked without notice

### Edge Cases & Validation Rules

| Scenario | Expected Behavior |
|---|---|
| App Name: 2 characters | Rejected: below 3-char minimum |
| Description: 19 characters | Rejected: below 20-char minimum |
| Redirect URI: `"not-a-url"` | Rejected: invalid URL format |
| Redirect URI: `"http://localhost:3000"` | Accepted for development |
| `per_page=0` | HTTP 400 |
| `per_page=31` | HTTP 400 |
| `page=-1` | HTTP 400 |
| `Authorization: Client-ID ` (empty key) | HTTP 401 |
| Authorization header present but key is revoked | HTTP 403 |
| Request exactly at rate limit (50th request in Demo) | HTTP 200 — succeeds; `X-Ratelimit-Remaining: 0` returned |
| 51st request in Demo mode in same hour | HTTP 429 returned |
| Production app exceeds 5,000 req/hr | HTTP 429; automatic reset at next hourly window |

---

## Appendix A — Summary of Business Rules

| ID | Feature | Rule Summary |
|---|---|---|
| BR-001 | Search | Search bar accessible to all users |
| BR-002 | Search | Results split into Photos / Collections / Users tabs |
| BR-003 | Search | Tag suggestions displayed on results page |
| BR-004 | Search | Infinite scroll on homepage photo grid |
| BR-005 | Search | 20 photos per page default; pagination supported |
| BR-006 | Search | Empty state message when no results found |
| BR-007 | Search | Trending searches displayed on homepage |
| BR-008 | Search | Category navigation filters by topic |
| BR-009 | Search | Sort by Relevance or Latest |
| BR-010 | Search | Keyword max 200 characters |
| BR-011 | Download | Guests can download free photos without account |
| BR-012 | Download | No attribution required (encouraged) |
| BR-013 | Download | Premium photos show lock icon |
| BR-014 | Download | Size options: Small / Medium / Large / Original |
| BR-015 | Download | Unsplash+ subscription required for premium photos |
| BR-016 | Download | Download increments photo's download counter |
| BR-017 | Download | JPEG format provided |
| BR-018 | Download | Guest downloads not linked to user accounts |
| BR-019 | Download | Cannot resell photos on competing platforms |
| BR-020 | Download | Cannot redistribute as competing stock service |
| BR-021 | Auth | Email must be valid format |
| BR-022 | Auth | Password minimum 8 characters |
| BR-023 | Auth | Password must include uppercase, lowercase, number |
| BR-024 | Auth | Email must be unique per account |
| BR-025 | Auth | Username must be unique platform-wide |
| BR-026 | Auth | Username 3–30 characters |
| BR-027 | Auth | Username: letters, numbers, underscores only |
| BR-028 | Auth | Username cannot be trademarked brand |
| BR-029 | Auth | Username cannot impersonate others |
| BR-030 | Auth | Account locked 15 min after 5 failed logins |
| BR-031 | Auth | Google and Facebook OAuth supported |
| BR-032 | Auth | Verification email sent on registration |
| BR-033 | Auth | Error message does not reveal if email is registered |
| BR-034 | Auth | Generic error: `"Invalid email or password"` |
| BR-035 | Auth | ToS acceptance required at registration |
| BR-036 | Upload | Only authenticated users can upload |
| BR-037 | Upload | Minimum 5 megapixel resolution |
| BR-038 | Upload | JPEG and PNG only |
| BR-039 | Upload | Max 50 MB per file |
| BR-040 | Upload | 10 photos/day daily upload limit |
| BR-041 | Upload | Royalty-free irrevocable license granted on upload |
| BR-042 | Upload | Contributor must own rights to the image |
| BR-043 | Upload | No nudity, violence, hate content, PII without consent |
| BR-044 | Upload | Identifiable people require model release |
| BR-045 | Upload | Photos reviewed before publishing |
| BR-046 | Upload | Photo shows `"Under Review"` status after upload |
| BR-047 | Upload | Description, alt text, location fields available |
| BR-048 | Upload | Up to 30 tags per photo |
| BR-049 | Collections | Only authenticated users can create collections |
| BR-050 | Collections | Unlimited collections per user |
| BR-051 | Collections | Collection name 1–60 characters |
| BR-052 | Collections | Collections can be Public or Private |
| BR-053 | Collections | Any photo can be added without subscription |
| BR-054 | Collections | No duplicate photo in same collection |
| BR-055 | Collections | Contributor notified when photo added to collection |
| BR-056 | Collections | Offensive collections removed; email sent |
| BR-057 | Collections | Deleting collection does not delete photos |
| BR-058 | Collections | Photos can be reordered in a collection |
| BR-059 | Collections | Collection cover photo is configurable |
| BR-060 | Collections | Users can follow public collections |
| BR-061 | Like | Only authenticated users can like |
| BR-062 | Like | Liking does not require subscription |
| BR-063 | Like | Like is a toggle (one like per user per photo) |
| BR-064 | Like | Contributor notified when photo is liked |
| BR-065 | Like | Liked photos visible on profile Likes tab |
| BR-066 | Like | Like count displayed publicly |
| BR-067 | Like | Unlike does not notify contributor |
| BR-068 | Profile | URL: `unsplash.com/@{username}` |
| BR-069 | Profile | Public profiles visible without login |
| BR-070 | Profile | Displays: avatar, name, bio, location, stats |
| BR-071 | Profile | Tabs: Photos, Likes, Collections |
| BR-072 | Profile | Editable: name, username, bio, location, website, socials |
| BR-073 | Profile | Username change limited to once per 30 days |
| BR-074 | Profile | Bio max 200 characters |
| BR-075 | Profile | Website must be valid URL |
| BR-076 | Profile | Avatar: JPEG/PNG, max 5 MB, min 200×200 px |
| BR-077 | Profile | Stats: total uploads, downloads received, views received |
| BR-078 | Profile | Admins can suspend accounts |
| BR-079 | Profile | 0-photo accounts reclaimed after 12 months inactivity |
| BR-080 | Profile | 1+-photo accounts reclaimed after 24 months inactivity |
| BR-081 | Unsplash+ | Paid subscription (monthly or annual) |
| BR-082 | Unsplash+ | Premium photos identified by lock icon |
| BR-083 | Unsplash+ | Active subscribers download premium photos |
| BR-084 | Unsplash+ | Non-subscribers redirected to subscription page |
| BR-085 | Unsplash+ | Auto-renews each billing cycle |
| BR-086 | Unsplash+ | Can cancel anytime; access until period end |
| BR-087 | Unsplash+ | Premium download blocked after subscription expires |
| BR-088 | Unsplash+ | Payment via Stripe; no raw card data stored |
| BR-089 | Unsplash+ | Confirmation email on subscription activation |
| BR-090 | Unsplash+ | Invoices downloadable from billing settings |
| BR-091 | Visual Search | Available to all users |
| BR-092 | Visual Search | Upload image or paste public URL |
| BR-093 | Visual Search | Max 20 MB for uploaded image |
| BR-094 | Visual Search | Returns top 20 visually similar photos |
| BR-095 | Visual Search | Can refine results with keyword |
| BR-096 | Visual Search | Powered by AI/ML image similarity engine |
| BR-097 | Visual Search | `"No results"` message when no similar photos found |
| BR-098 | API | Developer account required |
| BR-099 | API | Demo mode: 50 req/hour rate limit |
| BR-100 | API | Production requires app approval |
| BR-101 | API | Production: 5,000 req/hour |
| BR-102 | API | Pagination: 10 items/page default, max 30 |
| BR-103 | API | Authorization header required |
| BR-104 | API | Missing/invalid key → HTTP 401 |
| BR-105 | API | Rate limit exceeded → HTTP 429 |
| BR-106 | API | Attribution required in API apps |
| BR-107 | API | Must trigger download endpoint when photo is used |
| BR-108 | API | Violating apps may have API keys revoked |

---

## Appendix B — Key Pages & Endpoints

| Page | URL |
|---|---|
| Homepage | https://unsplash.com/ |
| Search Results | https://unsplash.com/s/photos/{keyword} |
| Photo Detail | https://unsplash.com/photos/{photo-id} |
| User Profile | https://unsplash.com/@{username} |
| Login | https://unsplash.com/login |
| Join / Register | https://unsplash.com/join |
| Submit Photo | https://unsplash.com/submit |
| Collections | https://unsplash.com/collections |
| Unsplash+ | https://unsplash.com/plus |
| Visual Search | https://unsplash.com/visual-search |
| API Developer Portal | https://unsplash.com/developers |
| API Documentation | https://unsplash.com/documentation |

---

*End of SPEC_UNSPLASH.md — v2.0.0*
