# spec_unflash.md — Unsplash Feature Spec (Refined Format)

**Version:** 1.0.0  
**Last Updated:** 2026-02-26  
**Source:** `SPEC_UNSPLASH.md`

## 1) Input Definition

| Input Group | Main Fields | Required | Validation |
|---|---|---|---|
| Search | `keyword`, `sort`, `tab`, `category` | `keyword` required | keyword 1–200 chars, trim whitespace |
| Auth | `email`, `password`, `oauth_provider`, `tos_consent` | required by flow | valid email, password policy, ToS required |
| Upload | `file`, `format`, `file_size`, `resolution`, `tags` | `file` required | JPEG/PNG, <=50MB, >=5MP, tags <=30 |
| Collection/Profile | `collection_name`, `visibility`, `photo_id`, `avatar` | by action | collection name 1–60 chars, avatar constraints |
| Subscription/API | `plan`, `payment_method`, `api_key`, `authorization_header`, `page`, `per_page` | by action | API auth required, pagination/rate-limit rules |

## 2) Output Definition

| Output Group | Output | Success Criteria | Error Criteria |
|---|---|---|---|
| UI | Search results, empty state, lock icon, upload status, profile tabs | Correct UI state per BR | Clear error shown, no unsafe action |
| API/Data | HTTP status, pagination payload, counters | Correct status and payload | 401/429 and policy-safe response |
| Notifications | email/like/collection notifications | Sent for valid trigger events | No leakage to unauthorized users |
| Access Control | gated features by role/subscription | only allowed role can proceed | unauthorized actions blocked |

## 3) Workflow Definition (Input → Process → Output)

1. User/API submits input for a feature flow.
2. System validates field constraints + auth/role.
3. System executes business logic for target feature.
4. System returns UI/API output + side effects (analytics/notifications).
5. System records audit/rate-limit/moderation logs.

## 4) Rule Coverage Scope

- This refined format keeps **Input/Output/Workflow** explicit.
- Detailed rule content remains in `SPEC_UNSPLASH.md` (`BR-001` → `BR-108`).
