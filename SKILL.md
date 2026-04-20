---
name: pinterest-image-source-recovery
description: Pinterest 图片源 URL 恢复 - 当自动化失败时使用合规优先的工作流进行浏览器辅助提取。处理反爬严格的情况。
---

# Pinterest Image Source Recovery

Use this skill when Pinterest page URLs cannot be reliably converted to direct image URLs because of anti-bot controls.

## Operating Principles

- Compliance first: do not provide captcha bypass, credential abuse, proxy evasion, or fingerprint spoofing playbooks.
- Use only user-authorized access (their own logged-in session or permitted content).
- Prefer deterministic extraction from already-loaded content before any heavier workflow.
- Always provide a fallback path so the task can still finish.

## Intake Checklist

Confirm before execution:

- `goal`: what the user needs (download, archive, dataset, inspiration board).
- `scope`: single Pin, list of Pin URLs, or board-level batch.
- `rights`: user confirms usage rights for target images.
- `target_format`: original URL list, downloaded files, or import-ready links.

If rights are unclear, switch to advisory mode and avoid bulk acquisition steps.

## Recovery Workflow (In Order)

1. Normalize inputs.
2. Attempt low-friction extraction from available page data.
3. If blocked, switch to browser-assisted extraction in a real user session.
4. If still blocked, provide non-automated fallback and a clear gap report.

### 1) Normalize Inputs

- Accept Pin links like `https://www.pinterest.com/pin/<id>/`.
- De-duplicate URLs and keep a stable index for result mapping.
- Mark invalid or non-Pinterest links early.

### 2) Low-Friction Extraction

Try these sources first (without bypass tactics):

- Existing direct `i.pinimg.com` links in user-provided text/HTML.
- `img`/`srcset` candidates from rendered page snapshots.
- Structured JSON blobs already present in fetched page content.

Validation rules:

- URL host should be `i.pinimg.com` (or other explicit image CDN host found in page data).
- Content type should be image-like when verifiable.
- Keep both `best_candidate` and `alternatives` when multiple sizes appear.

### 3) Browser-Assisted Extraction (Preferred Fallback)

When anti-bot blocks headless fetch, use user-controlled browser session:

- User opens Pin while logged in (if needed).
- Capture image request URLs from DevTools Network (filter by `img` or `pinimg`).
- Prefer highest-resolution successful response URL.
- Return URL + Pin source mapping.

This is semi-automatic, reproducible, and usually more stable than repeated anonymous scraping.

### 4) Non-Automated Fallback

If real URL extraction still fails:

- Record blocked items with reason (`auth_wall`, `rate_limited`, `dynamic_load_failed`, `missing_asset`).
- Offer acceptable alternatives:
  - user-provided screenshot archival,
  - manual export from authorized tools/workflows,
  - source-page reference list without direct binaries.

Do not fabricate "real image URLs" when extraction is not confirmed.

## Batch Strategy

For multi-link tasks:

- Use small batches (5-20) with delay between groups.
- Continue on partial failures.
- Produce per-item status (`ok`, `needs_browser`, `blocked`).

If failure ratio is high, stop early and switch to browser-assisted mode.

## Safety Rules

- No instructions for bypassing captchas or anti-abuse systems.
- No credential collection or account-sharing workflows.
- No claims of guaranteed full automation on protected platforms.

## Output Format

Return:

- `goal`: one-line intent.
- `scope`: total links and valid candidates.
- `method`: low-friction, browser-assisted, or fallback.
- `results`: resolved direct URLs with source mapping.
- `blocked`: items not resolved + concrete reason.
- `next_step`: one practical action to unblock the remaining work.

## Integration Notes

- If user also wants Eagle ingestion, hand off resolved direct URLs to `$eagle-save-image-to-library`.
- Keep unresolved items in a separate list; do not send page URLs as image URLs.
