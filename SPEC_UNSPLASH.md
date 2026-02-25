# SPEC_UNSPLASH.md — Feature Specification: Unsplash.com

**Version:** 1.1.0
**Author:** QC Team — Cook A Skill
**Last Updated:** 2026-02-26
**Target Site:** https://unsplash.com/
**Spec Type:** Product Feature Specification (PRD)

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

### Input Definition

| Input Group | Fields | Required | Validation / Constraint | Notes |
|---|---|---|---|---|
| Search Input | `keyword`, `sort`, `tab`, `category` | `keyword`: Yes, others: No | `keyword` length 1–200 chars; supports special characters; trim whitespace | Used in Search & Discovery |
| Authentication Input | `email`, `password`, `oauth_provider`, `tos_consent` | Depends on flow | Email format valid, password >= 8 chars + complexity, ToS consent required | Used in Signup/Login |
| Content Upload Input | `file`, `file_size`, `resolution`, `format`, `tags`, `description`, `location`, `model_release` | `file`: Yes | 5MP min, <=50MB, JPEG/PNG, tags <=30, rights ownership required | Used in Contributor upload flow |
| Collection/Like/Profile Input | `collection_name`, `visibility`, `photo_id`, `profile_fields`, `avatar` | Depends on action | Name 1–60 chars, avatar JPEG/PNG <=5MB, per-rule permission checks | Used in social/profile management |
| Subscription & API Input | `plan`, `payment_method`, `api_key`, `authorization_header`, `page`, `per_page` | Depends on action | Stripe payment flow, API key required, pagination limits enforced | Used in Unsplash+ and Developer API |

### Output Definition

| Output Group | Output Examples | Success Criteria | Failure / Error Behavior |
|---|---|---|---|
| UI Output | Search result grid, tabs, empty state, lock icon, upload status, profile stats | UI state matches business rule for each feature | Show clear error message, keep safe state (no broken action) |
| Data/API Output | Download event tracked, HTTP 200/401/429, pagination response, analytics counters | Response format/status is correct and auditable | Return standardized error codes and deny unauthorized access |
| Notification Output | Confirmation email, moderation rejection email, contributor like/collection notifications | Triggered exactly by qualifying events | No notification leakage to unauthorized users |
| Access Control Output | Gated premium access, protected upload/collection actions, admin moderation effects | Correct role receives correct access | Unauthorized attempts are blocked and logged |

### End-to-End Workflow (Input → Process → Output)

1. **Input Capture:** User or API client submits request data (search query, login credentials, upload payload, API params).
2. **Validation & Authorization:** System validates field constraints, session/auth state, and role permissions according to BR-001..BR-108.
3. **Business Processing:** Core feature logic executes (search, download, register/login, upload review, collection/like/profile updates, subscription, API serving).
4. **Output Rendering/Response:** UI and API response are returned with expected content, status code, counters, and side effects (emails/notifications/logs).
5. **Post-Action Tracking:** Analytics, rate limit counters, moderation records, and audit events are updated for traceability and security.

---

## 2. Feature: Photo Search & Discovery

### Description

Users can search for photos by keyword using the search bar. Results include photos, collections, and user profiles. The homepage displays an infinite-scroll grid of curated photos. Discovery is supported through trending tags and themed category navigation.

### Business Rules

- **BR-001:** The search bar is accessible on all pages (header navigation) — authenticated and unauthenticated users
- **BR-002:** Search results are filtered into 3 tabs: Photos, Collections, Users
- **BR-003:** Each search result page displays a set of relevant tag suggestions at the top to guide users toward related searches
- **BR-004:** The homepage features an infinite-scroll photo grid — new photos load automatically when the user scrolls to the bottom
- **BR-005:** Search results must return at most 20 photos per page by default; pagination is supported
- **BR-006:** If the search returns 0 results, the system must display an empty-state message: "We couldn't find anything for [keyword]" along with suggested alternative search terms
- **BR-007:** Trending/popular searches are displayed below the search bar on the homepage to assist discovery
- **BR-008:** Category navigation links (e.g., "Nature", "Travel", "Architecture") are available in the header and filter the photo grid by category
- **BR-009:** Users can sort search results by: Relevance (default) or Latest
- **BR-010:** Search keyword must be between 1 and 200 characters

### User Flow

1. User opens https://unsplash.com/
2. User types a keyword (e.g., "mountain") into the search bar
3. User presses Enter or clicks the search icon
4. System displays search results under 3 tabs: Photos, Collections, Users
5. User can switch between tabs to filter result type
6. User clicks a tag suggestion to refine the search
7. System loads next batch of results when user scrolls to the bottom of the page
8. User clicks a photo thumbnail to open the photo detail page

### Validation Rules

- Search keyword cannot be empty (whitespace-only input is treated as empty)
- Search keyword exceeding 200 characters is truncated to 200 characters before submission
- Special characters are allowed in search keywords (used for tag searches)

---

## 3. Feature: Photo Download

### Description

Users can download free photos without creating an account. Each photo has a "Download free" button. Premium photos (Unsplash+) require an active subscription. Downloaded photos are provided in high resolution.

### Business Rules

- **BR-011:** Guest users (unauthenticated) can download free photos without an account
- **BR-012:** Free photo downloads do not require attribution (but attribution to photographer is appreciated)
- **BR-013:** Download button is labeled "Download free" for free photos and shows a lock icon for Unsplash+ premium photos
- **BR-014:** Users can select download size: Small, Medium, Large, or Original (for free photos)
- **BR-015:** Unsplash+ photos require an active Unsplash+ subscription to download without watermark
- **BR-016:** Downloading a photo triggers a download count increment on the photo's stats
- **BR-017:** Downloaded photos are provided in JPEG format at the selected resolution
- **BR-018:** The platform tracks download events for analytics (no user account linkage for guest downloads)
- **BR-019:** Users cannot resell downloaded photos as-is on competing stock photo platforms
- **BR-020:** Users cannot redistribute downloaded photos as part of a competing stock photo service

### User Flow

1. User finds a photo via search or browsing
2. User clicks on a photo thumbnail to open the photo detail page
3. System displays photo in full-view with photographer info, stats (downloads, likes, views), and EXIF data
4. User clicks the "Download free" button (or the download icon dropdown)
5. System shows size options: Small, Medium, Large, Original
6. User selects desired size
7. System initiates file download to the user's device
8. Download count on the photo increments by 1

**Alternative Flow — Unsplash+ Photo:**

4a. User clicks on a locked (Unsplash+) photo
4b. System displays "This photo requires Unsplash+" with a "Subscribe" CTA
4c. If user has active Unsplash+ subscription, they can download with full quality (no watermark)
4d. If user does not have subscription, they are redirected to the Unsplash+ subscription page

### Validation Rules

- If a photo has been deleted by the contributor, the download button is disabled and the system shows "Photo no longer available"
- Download size options vary by original photo resolution; size options that exceed the original are not shown

---

## 4. Feature: User Registration & Login

### Description

Users can create an account with email/password or via Google/Facebook OAuth. Registration unlocks additional features: liking photos, creating collections, uploading photos, and accessing Unsplash+.

### Business Rules

- **BR-021:** Email must be a valid format (RFC 5321 compliant)
- **BR-022:** Password must be at least 8 characters long
- **BR-023:** Password must contain at least one uppercase letter, one lowercase letter, and one number
- **BR-024:** Email address must be unique — no two accounts can share the same email
- **BR-025:** Username must be unique across the platform (first-come, first-served basis)
- **BR-026:** Username must be between 3 and 30 characters
- **BR-027:** Username can only contain letters, numbers, and underscores — no spaces, special characters, or URLs
- **BR-028:** Username cannot be a trademarked brand name
- **BR-029:** Username cannot impersonate another person or public figure
- **BR-030:** After 5 consecutive failed login attempts, the account is temporarily locked for 15 minutes
- **BR-031:** Users can sign up/login via Google OAuth or Facebook OAuth as alternative methods
- **BR-032:** Upon successful registration, the system sends a confirmation email to the registered address
- **BR-033:** The system must not reveal whether an email address is registered when returning login error messages (anti-enumeration)
- **BR-034:** Login error message must be generic: "Invalid email or password"
- **BR-035:** Users must agree to Terms of Service and Privacy Policy before completing registration

### User Flow — Registration

1. User navigates to https://unsplash.com/join
2. User fills in: First Name, Last Name, Email, Username, Password
3. User checks the "I agree to Terms of Service and Privacy Policy" checkbox
4. User clicks "Join"
5. System validates all fields
6. On success: System creates the account, sends a verification email, and redirects to the homepage (logged in)
7. On failure: System displays specific field-level validation errors inline

### User Flow — Login

1. User navigates to https://unsplash.com/login
2. User enters Email and Password
3. User clicks "Login"
4. System validates credentials
5. On success: System redirects to the homepage or the previously attempted page
6. On failure: System displays "Invalid email or password" error message
7. After 5 consecutive failures: System locks the account for 15 minutes and displays the lockout message

### User Flow — OAuth Login (Google)

1. User clicks "Continue with Google" on the login or join page
2. System redirects to Google OAuth consent screen
3. User grants permission
4. System creates or links the account and redirects to the homepage (logged in)

### Validation Rules

- Email: must match `^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$`
- Password: minimum 8 characters, must contain uppercase, lowercase, and digit
- Username: 3–30 chars, alphanumeric + underscore only, no leading/trailing underscores
- First Name / Last Name: required, 1–50 characters each
- All fields are required unless marked optional

---

## 5. Feature: Photo Upload (Contributors)

### Description

Registered users can submit photos to Unsplash. Uploaded photos are reviewed against submission guidelines. Approved photos become part of the public library under the Unsplash license.

### Business Rules

- **BR-036:** Only registered (authenticated) users can upload photos
- **BR-037:** Uploaded photos must have a minimum resolution of 5 megapixels (e.g., 2500 × 2000 pixels minimum)
- **BR-038:** Accepted file formats: JPEG and PNG only
- **BR-039:** Maximum file size per upload: 50 MB
- **BR-040:** Users can upload up to 10 photos per day (daily limit for non-verified accounts)
- **BR-041:** By uploading, contributors grant Unsplash and users a royalty-free, irrevocable license to use the photo
- **BR-042:** Contributors must own the rights to the uploaded image or have permission from the rights holder
- **BR-043:** Photos must not contain: nudity, violence, hate content, personally identifiable information (PII) of individuals without consent, copyrighted logos or trademarked brand elements
- **BR-044:** Photos depicting identifiable people in editorial or commercial contexts require a valid model release
- **BR-045:** Submitted photos are subject to review — non-compliant photos are rejected with an explanatory email to the contributor
- **BR-046:** Upon successful upload, the photo appears on the contributor's profile with status "Under Review" until approved
- **BR-047:** Contributors can add a description, alt text, and location to their uploaded photos
- **BR-048:** Contributors can add up to 30 tags per photo for discoverability

### User Flow

1. Authenticated user clicks "Submit a photo" button in the top-right navigation
2. User is redirected to https://unsplash.com/submit
3. User selects a photo file from their device
4. System validates: file format (JPEG/PNG), file size (≤ 50 MB), resolution (≥ 5 megapixels)
5. On validation pass: System shows a preview of the photo
6. User fills in optional fields: Description, Alt text, Location, Tags (up to 30)
7. User clicks "Submit" to confirm
8. System uploads the photo and sets its status to "Under Review"
9. System sends a confirmation notification to the contributor
10. Upon admin approval, the photo is published to the public library

**Rejection Flow:**

10a. Admin rejects the photo (violates guidelines)
10b. System sends an explanatory rejection email to the contributor
10c. Photo is removed from the contributor's profile

### Validation Rules

- File format must be JPEG or PNG (other formats are rejected with error: "Only JPEG and PNG files are accepted")
- File size must not exceed 50 MB (error: "File size exceeds the 50 MB limit")
- Image resolution must be at least 5 megapixels (error: "Image resolution must be at least 5 megapixels (2500 × 2000 px minimum)")
- Tags: max 30 tags; each tag max 50 characters; no duplicate tags
- Description: max 500 characters
- Location: max 100 characters

---

## 6. Feature: Collections Management

### Description

Registered users can create collections to organize and save photos. Collections can be public or private. Other users can follow public collections.

### Business Rules

- **BR-049:** Only authenticated users can create collections
- **BR-050:** A user can create an unlimited number of collections
- **BR-051:** Collection names must be between 1 and 60 characters
- **BR-052:** Collections can be set as Public (visible to all) or Private (visible only to the owner)
- **BR-053:** Users can add any photo (free or Unsplash+) to their collections — adding a premium photo does not require a subscription
- **BR-054:** A user cannot add the same photo to the same collection twice (duplicate prevention)
- **BR-055:** The photo contributor receives a notification when their photo is added to another user's collection
- **BR-056:** Collections containing offensive content are removed by admins immediately, and an email is sent to the collection creator
- **BR-057:** Users can delete their collections — deleting a collection does not delete the photos
- **BR-058:** Users can reorder photos within a collection
- **BR-059:** A collection displays a cover photo (by default: the first photo added; user can change this)
- **BR-060:** Other users can follow public collections to receive updates

### User Flow — Create a Collection

1. Authenticated user views a photo they want to save
2. User clicks the "Collect" (bookmark) icon on the photo
3. System displays a dropdown showing existing collections + "Create new collection" option
4. User clicks "Create new collection"
5. User enters a collection name and selects Public or Private
6. User clicks "Create"
7. System creates the collection and adds the current photo to it
8. System displays a success confirmation: "Photo added to [Collection Name]"

### User Flow — Add Photo to Existing Collection

1. Authenticated user views a photo
2. User clicks the "Collect" icon
3. System displays a dropdown list of the user's existing collections with checkmarks on collections that already contain this photo
4. User clicks on the desired collection
5. System adds the photo to the collection and shows a checkmark confirmation
6. Photo contributor receives a notification

### User Flow — Delete a Collection

1. User navigates to their profile → Collections tab
2. User selects the collection to delete
3. User clicks the "Delete collection" option
4. System shows a confirmation dialog: "Are you sure you want to delete [Collection Name]? This cannot be undone."
5. User confirms
6. System deletes the collection and all collection-photo associations (photos remain on Unsplash)

### Validation Rules

- Collection name: 1–60 characters, cannot be empty or whitespace-only
- Duplicate photo in same collection is silently prevented (system shows "Already in this collection")

---

## 7. Feature: Like / Save Photos

### Description

Authenticated users can "like" (heart) photos to show appreciation to photographers and to save them to a personal likes list.

### Business Rules

- **BR-061:** Only authenticated users can like a photo
- **BR-062:** A user can like any photo (free or Unsplash+) without a subscription
- **BR-063:** A user can only like a photo once — the like button toggles (click again to unlike)
- **BR-064:** The photo contributor receives a notification when their photo is liked
- **BR-065:** A user's liked photos are visible on their profile under the "Likes" tab (public by default)
- **BR-066:** The like count on each photo is displayed publicly
- **BR-067:** Unlike does not notify the contributor

### User Flow

1. Authenticated user hovers over or opens a photo
2. User clicks the heart icon (Like button)
3. System increments the like count by 1 and highlights the heart icon (red/filled)
4. Photo contributor receives a notification: "[Username] liked your photo"
5. The liked photo appears in the user's profile under "Likes" tab

**Unlike Flow:**

3a. User clicks the filled heart icon again
3b. System decrements the like count by 1 and shows the hollow heart icon
3c. No notification is sent to the contributor

**Unauthenticated Flow:**

2a. Guest user clicks the heart icon
2b. System displays a login/signup prompt: "Sign up to like this photo"
2c. Guest is redirected to the login/join page

---

## 8. Feature: User Profile

### Description

Every registered user has a public profile page showing their uploaded photos, liked photos, and collections. Users can edit their profile information.

### Business Rules

- **BR-068:** Profile URL format: `https://unsplash.com/@{username}`
- **BR-069:** Profile page is publicly visible (no login required to view)
- **BR-070:** Profile displays: avatar, name, bio, location, website, social links, followers/following count
- **BR-071:** Profile tabs: Photos, Likes, Collections
- **BR-072:** Users can edit: name, username, bio, location, website, portfolio URL, social links (Twitter/Instagram)
- **BR-073:** Username changes are allowed but limited to once per 30 days
- **BR-074:** Bio max length: 200 characters
- **BR-075:** Website URL must be a valid URL format (starting with http:// or https://)
- **BR-076:** Users can upload an avatar image (JPEG or PNG, max 5 MB, minimum 200×200 px)
- **BR-077:** Profile stats are visible: total photos uploaded, total downloads received, total views received
- **BR-078:** Admins can suspend or deactivate accounts violating community guidelines
- **BR-079:** Inactive accounts with 0 photos may have their username reclaimed after 12 months of inactivity
- **BR-080:** Inactive accounts with 1+ photos may have their username reclaimed after 24 months of inactivity

### User Flow — View Profile

1. User navigates to `https://unsplash.com/@{username}`
2. System displays the public profile page with photo grid (Photos tab active by default)
3. User can switch between Photos, Likes, Collections tabs

### User Flow — Edit Profile

1. Authenticated user clicks their avatar → "Edit profile"
2. System displays the profile edit form
3. User updates desired fields (name, bio, location, website, avatar, social links)
4. User clicks "Save changes"
5. System validates all fields
6. On success: Profile is updated and user is redirected to their profile page
7. On failure: Specific field validation errors are displayed inline

### Validation Rules

- Username: 3–30 chars, alphanumeric + underscore only
- Bio: max 200 characters
- Website URL: must start with `http://` or `https://`
- Avatar: JPEG or PNG only, max 5 MB, minimum 200×200 px
- Username changes limited to 1 change per 30 days

---

## 9. Feature: Unsplash+ Subscription

### Description

Unsplash+ is a premium subscription tier that gives users access to exclusive licensed photos not available for free. Unsplash+ photos are marked with a lock icon.

### Business Rules

- **BR-081:** Unsplash+ is a paid subscription with a monthly or annual billing option
- **BR-082:** Unsplash+ photos are identified by a lock icon overlay on the thumbnail
- **BR-083:** Only active Unsplash+ subscribers can download premium (locked) photos without watermark
- **BR-084:** A non-subscriber clicking a locked photo is redirected to the Unsplash+ subscription page
- **BR-085:** Unsplash+ subscription is auto-renewed at the end of each billing cycle unless cancelled
- **BR-086:** Users can cancel the Unsplash+ subscription at any time; access remains until the end of the current billing period
- **BR-087:** Unsplash+ photos cannot be downloaded after subscription expires — access reverts to free content only
- **BR-088:** Payment is processed securely via Stripe; Unsplash does not store raw card data
- **BR-089:** Upon successful subscription activation, the user receives a confirmation email with billing details
- **BR-090:** Users can view and download their invoices from the billing settings page

### User Flow — Subscribe

1. User clicks "Get Unsplash+" (CTA in header or on a locked photo)
2. System displays the subscription plans page (monthly vs. annual pricing)
3. User selects a plan and clicks "Subscribe"
4. System redirects to a Stripe checkout page
5. User enters payment details (card number, expiry, CVV, billing address)
6. User confirms payment
7. Stripe processes the payment
8. On success: System activates the Unsplash+ subscription, sends a confirmation email, and redirects to the library
9. On failure: Stripe returns an error; System displays "Payment failed — please check your card details"

### User Flow — Cancel Subscription

1. Authenticated Unsplash+ subscriber goes to Account Settings → Billing
2. User clicks "Cancel subscription"
3. System displays a confirmation dialog: "Your Unsplash+ access will remain active until [billing cycle end date]. Are you sure?"
4. User confirms
5. System cancels the auto-renewal; user retains access until the end of the current billing period
6. System sends a cancellation confirmation email

### Validation Rules

- Payment card must be valid (Luhn algorithm check via Stripe)
- Billing address fields are required for subscription activation
- If a subscription is cancelled and the user tries to download a Unsplash+ photo after expiry, they are shown the subscription CTA again

---

## 10. Feature: Visual Search

### Description

Unsplash provides a Visual Search tool that allows users to search by image rather than keyword. Users can upload an image or paste a URL to find visually similar photos.

### Business Rules

- **BR-091:** Visual Search is available to all users (authenticated and unauthenticated)
- **BR-092:** Users can initiate Visual Search by uploading an image (JPEG, PNG) or by entering a public image URL
- **BR-093:** Uploaded images for Visual Search must be at most 20 MB
- **BR-094:** Visual Search returns the top 20 most visually similar photos from the Unsplash library
- **BR-095:** Visual Search results can be further refined by keyword after the initial image-based search
- **BR-096:** Visual Search is powered by an AI/ML image similarity engine — results are ranked by visual similarity score
- **BR-097:** If no visually similar photos are found, the system displays: "No similar photos found. Try a different image."

### User Flow

1. User navigates to the Visual Search page or clicks the camera icon in the search bar
2. User uploads a reference image from their device or pastes a public image URL
3. System processes the image using image similarity analysis
4. System displays up to 20 visually similar photos in a grid
5. User can click any result to open the photo detail page and download
6. User can type a keyword to further filter the visual search results

### Validation Rules

- Uploaded image must be JPEG or PNG
- Uploaded image file size must not exceed 20 MB
- Image URL must be publicly accessible (no authentication required); inaccessible URLs return error: "Image URL is not accessible. Please try uploading the image directly."

---

## 11. Feature: API Access (Developers)

### Description

Unsplash provides a public REST API for developers to integrate Unsplash photos into applications. API access requires registration and app approval.

### Business Rules

- **BR-098:** To use the API, developers must register a free Unsplash developer account
- **BR-099:** Each app starts in Demo mode with a rate limit of 50 requests per hour
- **BR-100:** Production API access requires an application review and approval by the Unsplash team
- **BR-101:** Production rate limit: up to 5,000 requests per hour
- **BR-102:** API responses for list endpoints return 10 items per page by default; max 30 items per page
- **BR-103:** API calls must include a valid `Authorization: Client-ID {ACCESS_KEY}` header
- **BR-104:** Unauthenticated API calls return HTTP 401 Unauthorized
- **BR-105:** Rate-limited API calls return HTTP 429 Too Many Requests with a `X-Ratelimit-Remaining: 0` header
- **BR-106:** API applications must display attribution ("Photo by [Photographer Name] on Unsplash") for each photo used
- **BR-107:** API applications must call the Unsplash download endpoint to trigger a download event when a user downloads/uses a photo
- **BR-108:** Applications violating API terms may have their API keys revoked without notice

### User Flow — API Registration

1. Developer navigates to https://unsplash.com/developers
2. Developer clicks "Register as a developer"
3. System prompts login or registration (if not already authenticated)
4. Developer clicks "New Application" and fills in: App Name, Description, Redirect URIs, and agrees to API Terms
5. System creates the application and provides Access Key and Secret Key
6. Developer uses the Access Key in all API requests

### User Flow — API Photo Search

1. Developer sends GET request: `GET /search/photos?query={keyword}&page=1&per_page=10`
2. Request includes header: `Authorization: Client-ID {ACCESS_KEY}`
3. System validates the access key
4. System returns JSON response with photo results array, total count, and pagination info
5. Developer parses the response and renders photos in their application

**Rate Limit Exceeded Flow:**

3a. If the developer has exceeded 50 requests/hour (Demo mode) or 5,000/hour (Production):
3b. System returns HTTP 429 with header `X-Ratelimit-Remaining: 0`
3c. Developer must wait until the next hourly window to resume API calls

### Validation Rules

- App Name: 3–100 characters, required
- Description: 20–500 characters, required
- Redirect URIs: must be valid URL format; localhost is permitted for development
- `per_page` parameter: minimum 1, maximum 30; values outside range return HTTP 400
- `page` parameter: minimum 1; negative or zero values return HTTP 400

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
| BR-034 | Auth | Generic error: "Invalid email or password" |
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
| BR-046 | Upload | Photo shows "Under Review" status after upload |
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
| BR-097 | Visual Search | "No results" message when no similar photos found |
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

*End of SPEC_UNSPLASH.md*
