# Test Cases — Generated from `SPEC_UNSPLASH.md`

## Run Configuration
- `feature_name`: Unsplash Platform (Multi-feature)
- `output_format`: markdown
- `coverage_level`: standard
- `enable_security`: true
- `mask_pii`: true

## Step 0 — PII Masking Report
No sensitive runtime/customer data patterns were detected in the spec.

| Pattern | Count masked |
|---|---:|
| Email | 0 |
| Phone | 0 |
| API key/token | 0 |
| JWT | 0 |
| National ID | 0 |
| Credit card | 0 |
| IP | 0 |
| Internal URL | 0 |

## Step 1 — Input Validation Result
- `spec_content` provided and valid.
- Contains user flows and business rules.
- Generation continued.

## Test Cases

| ID | Title | Type | Priority | Rule Ref | Precondition | Test Data | Steps | Expected Result |
|---|---|---|---|---|---|---|---|---|
| TC-001 | Search with valid keyword returns results | Happy Path | P1 - Critical | BR-001, BR-002, BR-005 | Unsplash homepage is available | keyword=`mountain` | 1) Open homepage 2) Enter keyword 3) Submit search | Results page opens with Photos/Collections/Users tabs and photo results list. |
| TC-002 | Search with whitespace-only keyword is rejected | Negative | P2 - High | BR-010 | Homepage loaded | keyword=`   ` | 1) Enter spaces only 2) Submit | Search is not executed as valid input; no invalid query request is sent. |
| TC-003 | Search keyword >200 chars is truncated | Edge Case | P2 - High | BR-010 | Homepage loaded | 230-char string | 1) Paste long keyword 2) Submit | Query length is capped at 200 chars before submission. |
| TC-004 | Empty search result shows fallback message | Negative | P3 - Medium | BR-006 | Search available | keyword=`asdasdnonexistenttagzzz` | 1) Submit rare keyword | Empty state shows: "We couldn't find anything for [keyword]" + suggested alternatives. |
| TC-005 | Download free photo as guest | Happy Path | P1 - Critical | BR-011, BR-014, BR-016, BR-017 | A free photo detail page is open | Size=`Medium` | 1) Click Download free 2) Select Medium | JPEG download starts and download count increments by 1. |
| TC-006 | Download option unavailable for deleted photo | Negative | P2 - High | BR-013 | Photo was deleted by contributor | n/a | 1) Open old/deleted photo URL 2) Observe action buttons | Download button is disabled and message "Photo no longer available" is shown. |
| TC-007 | Locked Unsplash+ photo redirects non-subscriber | Negative | P1 - Critical | BR-015 | User is logged in without Unsplash+ | premium photo | 1) Open locked photo 2) Attempt download | User sees Unsplash+ requirement and is redirected to subscription CTA/page. |
| TC-008 | Register with valid data succeeds | Happy Path | P1 - Critical | BR-021, BR-022, BR-023, BR-024, BR-025, BR-032, BR-035 | Guest at `/join` | Valid unique email/username and strong password | 1) Fill all required fields 2) Accept terms 3) Submit | Account is created, user is logged in, verification/confirmation email is sent. |
| TC-009 | Register with duplicate email is blocked | Negative | P1 - Critical | BR-024 | Existing account with same email | Existing email + new username | 1) Submit registration | Registration fails with field-level duplicate-email error. |
| TC-010 | Login failure error does not reveal account existence | Security | P1 - Critical | BR-033, BR-034 | Login page open | unknown email, wrong password | 1) Attempt login with unknown email 2) Attempt login with known email wrong password | Both cases return the same generic message: "Invalid email or password". |
| TC-011 | Account is locked after 5 failed login attempts | Security | P1 - Critical | BR-030 | Existing user account | wrong password x5 | 1) Fail login 5 times consecutively 2) Retry correct password during lock | Account remains temporarily locked for 15 minutes. |
| TC-012 | OAuth login with Google succeeds | Happy Path | P2 - High | BR-031 | Google account is available | OAuth consent granted | 1) Click Continue with Google 2) Grant permissions | User is authenticated and redirected to homepage. |
| TC-013 | Contributor uploads valid JPEG (>=5MP, <=50MB) | Happy Path | P1 - Critical | BR-036, BR-037, BR-038, BR-039, BR-046 | Authenticated contributor at `/submit` | JPEG 6MP, 8MB + tags | 1) Upload file 2) Fill metadata 3) Submit | Upload accepted and photo status is `Under Review`. |
| TC-014 | Reject upload with unsupported format | Negative | P2 - High | BR-038 | `/submit` opened | GIF/WEBP file | 1) Select unsupported file type 2) Submit | Upload blocked with "Only JPEG and PNG files are accepted". |
| TC-015 | Reject upload above 50MB | Edge Case | P2 - High | BR-039 | `/submit` opened | PNG 55MB | 1) Upload file | Upload blocked with 50MB limit error. |
| TC-016 | Reject upload below minimum resolution | Edge Case | P2 - High | BR-037 | `/submit` opened | JPEG <5MP | 1) Upload low-resolution image | Upload blocked with minimum 5MP resolution error. |
| TC-017 | Create new private collection and add current photo | Happy Path | P2 - High | BR-049, BR-051, BR-052 | Authenticated user viewing photo | Collection name=`Inspiration` visibility=`Private` | 1) Click Collect 2) Create new collection 3) Save | Collection created and photo added successfully. |
| TC-018 | Prevent duplicate photo add in same collection | Negative | P2 - High | BR-054 | Photo already in selected collection | same photo/collection pair | 1) Click Collect 2) Choose same collection again | System prevents duplicate and indicates already collected state. |
| TC-019 | Delete collection does not delete photos | Happy Path | P3 - Medium | BR-057 | User owns collection with photos | n/a | 1) Delete collection with confirmation | Collection removed; original photos remain on Unsplash. |
| TC-020 | Like photo toggles like state and count | Happy Path | P2 - High | BR-061, BR-063, BR-066 | Authenticated user on photo | n/a | 1) Click heart once 2) Click heart again | First click adds like/increments count; second click removes like/decrements count. |
| TC-021 | Guest trying to like is prompted to login | Negative | P2 - High | BR-061 | Guest user | n/a | 1) Guest clicks heart icon | Login/signup prompt is shown and guest can be redirected to auth page. |
| TC-022 | Like action sends contributor notification | Happy Path | P3 - Medium | BR-064 | Authenticated user likes someone else’s photo | n/a | 1) Like a photo | Contributor receives like notification. |
| TC-023 | Update profile with valid bio/location/social links | Happy Path | P3 - Medium | BR-068, BR-071, BR-072 | Logged-in user in profile settings | valid text and links | 1) Edit profile 2) Save | Profile updates are persisted and visible on public profile. |
| TC-024 | Username update to existing username is rejected | Negative | P2 - High | BR-070 | Another account already owns target username | duplicate username | 1) Attempt username change 2) Save | Update fails with uniqueness validation error. |
| TC-025 | Subscribe Unsplash+ with successful Stripe payment | Happy Path | P1 - Critical | BR-081, BR-083, BR-088, BR-089 | Logged-in non-subscriber | Valid card via Stripe | 1) Select plan 2) Complete checkout | Subscription activated and confirmation email sent. |
| TC-026 | Failed payment does not activate subscription | Negative | P1 - Critical | BR-088 | Logged-in non-subscriber | Declined card | 1) Submit payment 2) Observe response | Payment failure message is shown and no premium access is granted. |
| TC-027 | Cancel subscription keeps access until billing period end | Happy Path | P2 - High | BR-086 | Active Unsplash+ subscriber | n/a | 1) Cancel subscription in Billing 2) Check access immediately | Auto-renew is canceled, access remains active until period end. |
| TC-028 | Visual search by valid image upload returns similar photos | Happy Path | P2 - High | BR-091, BR-092, BR-094 | Visual search page available | JPEG 2MB | 1) Upload image 2) Run search | Up to 20 visually similar photos are returned. |
| TC-029 | Visual search rejects inaccessible image URL | Negative | P2 - High | BR-092, BR-097 | Visual search page | private/403 image URL | 1) Paste inaccessible URL 2) Submit | Error shown: image URL is not accessible. |
| TC-030 | API call without Authorization returns 401 | Security | P1 - Critical | BR-103, BR-104 | API endpoint available | GET `/search/photos?query=nature` without Client-ID | 1) Send request without auth header | Response HTTP 401 Unauthorized. |
| TC-031 | Demo API rate limit returns 429 with remaining=0 | Security | P1 - Critical | BR-099, BR-105 | Demo app key configured | exceed 50 req/hour | 1) Send requests over demo limit 2) Inspect response | HTTP 429 returned with `X-Ratelimit-Remaining: 0`. |
| TC-032 | Security: XSS payload in profile bio is neutralized | Security | P1 - Critical | BR-071 | Logged-in profile owner | `<script>alert(1)</script>` | 1) Save bio with script payload 2) Open public profile | Script is not executed; content is sanitized/escaped safely. |

## Coverage Summary

| Type | Count |
|---|---:|
| Happy Path | 13 |
| Negative | 10 |
| Edge Case | 3 |
| Security | 6 |
| **Total** | **32** |

## Traceability Matrix

| Rule Ref | Covered by |
|---|---|
| BR-001..BR-010 | TC-001, TC-002, TC-003, TC-004 |
| BR-011..BR-020 | TC-005, TC-006, TC-007 |
| BR-021..BR-035 | TC-008, TC-009, TC-010, TC-011, TC-012 |
| BR-036..BR-048 | TC-013, TC-014, TC-015, TC-016 |
| BR-049..BR-060 | TC-017, TC-018, TC-019 |
| BR-061..BR-067 | TC-020, TC-021, TC-022 |
| BR-068..BR-080 | TC-023, TC-024, TC-032 |
| BR-081..BR-090 | TC-025, TC-026, TC-027 |
| BR-091..BR-097 | TC-028, TC-029 |
| BR-098..BR-108 | TC-030, TC-031 |

## Coverage Gap Notes (standard level)
- Some rules in each range are covered via grouped functional scenarios rather than 1:1 per-rule mapping.
- For `coverage_level=full`, expand to one explicit TC per individual rule (BR-001 ... BR-108).

## Test Execution Report Template

- Feature: Unsplash Platform
- Tester: ___________
- Date: ___________
- Environment: ___________
- Build/Version: ___________

| Total TCs | Pass | Fail | Blocked | Not Run |
|---|---|---|---|---|
| 32 |  |  |  |  |

| Bug ID | TC ID | Rule Ref | Description | Severity | Status |
|---|---|---|---|---|---|
|  |  |  |  |  |  |
