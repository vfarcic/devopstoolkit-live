---
name: generate-analytics
description: Refresh the channel analytics block on content/sponsor/_index.md by fetching live numbers from the YouTube Data + YouTube Analytics APIs.
user-invocable: true
---

# Generate Sponsor Page Analytics

Refresh the analytics block in `content/sponsor/_index.md` (between `<!-- SPONSOR_ANALYTICS_START -->` and `<!-- SPONSOR_ANALYTICS_END -->`) with the latest channel stats, demographics, geography, and engagement numbers.

The data comes from the YouTube Data API (channel statistics) and YouTube Analytics API (demographics, geography, engagement). Both require an OAuth2 access token with the `yt-analytics.readonly` and `youtube.readonly` scopes — the cached token from the `youtube-automation` tool is reused.

This skill replicates the logic in `~/code/youtube-automation/internal/publishing/sponsor_charts.go` so the output format stays consistent with what that tool produces. If the format on the live page ever diverges from this skill's output, re-check that file.

## Inputs (must exist on disk)

| Path | What it is |
|------|------------|
| `.claude/skills/generate-analytics/client_secret.json` | Google OAuth client credentials (`client_id`, `client_secret`). Needed to refresh the access token. Gitignored. Extracted from cluster Secret `youtube-automation-oauth-client` (see "Restoring credentials" below). |
| `~/.credentials/youtube-go.json` | Cached OAuth2 token (`access_token`, `refresh_token`, `expiry`, …). Lives outside this repo because `youtube-automation` writes back to it on refresh — moving it would break that tool. Shared single source of truth. |

The channel ID is fixed: `UCfz8x0lVzJpb_dgWm9kPVrw` (DevOps Toolkit).

If `client_secret.json` is missing, see "Restoring credentials" at the bottom. If `youtube-go.json` is missing or its `refresh_token` is revoked, stop and ask the user to re-run the `youtube-automation` OAuth flow.

## Step 1: Refresh the access token

The cached `access_token` is almost certainly expired (Google access tokens last ~1h). Exchange the `refresh_token` for a new access token:

```bash
SECRET_FILE=.claude/skills/generate-analytics/client_secret.json
CLIENT_ID=$(jq -r '.installed.client_id // .web.client_id' "$SECRET_FILE")
CLIENT_SECRET=$(jq -r '.installed.client_secret // .web.client_secret' "$SECRET_FILE")
REFRESH_TOKEN=$(jq -r '.refresh_token' ~/.credentials/youtube-go.json)

ACCESS_TOKEN=$(curl -s -X POST https://oauth2.googleapis.com/token \
  -d "client_id=${CLIENT_ID}" \
  -d "client_secret=${CLIENT_SECRET}" \
  -d "refresh_token=${REFRESH_TOKEN}" \
  -d "grant_type=refresh_token" | jq -r '.access_token')
```

If the response has no `access_token` (e.g. `invalid_grant`), the refresh token has been revoked. Stop and ask the user to re-run the `youtube-automation` OAuth flow.

**Do not write the new access token back to the cache file** — `youtube-automation` manages that. Use it in-memory for the API calls below.

## Step 2: Fetch the four datasets

Compute the 90-day window once: `END=$(date +%F)` and `START=$(date -v-90d +%F)` (macOS) — adjust for Linux if needed (`date -d '90 days ago' +%F`).

Channel ID is constant:

```bash
CHANNEL_ID=UCfz8x0lVzJpb_dgWm9kPVrw
```

### 2a. Channel statistics (lifetime — public, but use the token anyway)

```
GET https://youtube.googleapis.com/youtube/v3/channels?part=statistics&id=${CHANNEL_ID}
Authorization: Bearer ${ACCESS_TOKEN}
```

Take `items[0].statistics.{subscriberCount, viewCount, videoCount, hiddenSubscriberCount}`.

### 2b. Demographics (last 90 days)

```
GET https://youtubeanalytics.googleapis.com/v2/reports
  ?ids=channel==${CHANNEL_ID}
  &startDate=${START}
  &endDate=${END}
  &dimensions=ageGroup,gender
  &metrics=viewerPercentage
```

Response `rows` is an array of `[ageGroup, gender, viewerPercentage]`. Aggregate by summing `viewerPercentage` per `ageGroup` and per `gender` independently.

### 2c. Geographic distribution (last 90 days, top 10)

```
GET https://youtubeanalytics.googleapis.com/v2/reports
  ?ids=channel==${CHANNEL_ID}
  &startDate=${START}
  &endDate=${END}
  &dimensions=country
  &metrics=views
  &sort=-views
  &maxResults=10
```

Each row is `[countryCode, views]`. Compute total views across the 10 rows, then derive each country's percentage as `views / total * 100`.

### 2d. Engagement (last 90 days, regular videos only)

```
GET https://youtubeanalytics.googleapis.com/v2/reports
  ?ids=channel==${CHANNEL_ID}
  &startDate=${START}
  &endDate=${END}
  &dimensions=creatorContentType
  &metrics=averageViewDuration,likes,comments,shares,views
```

Rows are `[creatorContentType, averageViewDuration, likes, comments, shares, views]`. **Keep only the row where `creatorContentType == "videoOnDemand"`** (this excludes Shorts and live streams).

## Step 3: Format the markdown block

The block must match this exact structure (whitespace included). `{Month D, YYYY}` is today's date in `February 3, 2026` style — use the actual current date, not a hardcoded one.

```markdown
<!-- SPONSOR_ANALYTICS_START -->
## Channel Analytics

*Last updated: {Month D, YYYY}.*

### Overview (All Time)

| Metric | Value |
|--------|-------|
| Subscribers | {N} |
| Total Views | {N} |
| Videos | {N} |

### Audience Demographics (Last 90 Days)

```mermaid
xychart-beta horizontal
    title "Age Distribution"
    x-axis [{labels}]
    y-axis "Percentage" 0 --> {yMax}
    bar [{values}]
```

```mermaid
pie showData title Gender Distribution
    "{Label}" : {X.X}
    ...
```

### Geographic Distribution (Last 90 Days)

```mermaid
xychart-beta horizontal
    title "Top Countries by Views"
    x-axis [{labels}]
    y-axis "Percentage" 0 --> {yMax}
    bar [{values}]
```

### Engagement (Last 90 Days, Regular Videos Only)

| Metric | Value |
|--------|-------|
| Avg Watch Time | {Xm Ys} |
| Likes | {N} |
| Comments | {N} |
| Shares | {N} |
| Engagement Rate | {X.XX}% |

<!-- SPONSOR_ANALYTICS_END -->
```

### Formatting rules

- **`formatNumber`** — thousand separators: `93900` → `93,900`. If `hiddenSubscriberCount` is true, render Subscribers as `N/A`.
- **Age groups** — drop entries `< 0.5%`. Order: `13-17, 18-24, 25-34, 35-44, 45-54, 55-64, 65+`. Strip `age` prefix; convert trailing `-` to `+` (`age65-` → `65+`). Percentages with one decimal: `34.4`.
- **Gender** — drop entries `< 0.5%`. Map `male`→`Male`, `female`→`Female`, `user_specified`→`Other`. Percentages with one decimal.
- **Countries** — friendly names only for: `US: United States, IN: India, GB: United Kingdom, DE: Germany, CA: Canada, BR: Brazil, AU: Australia, NL: Netherlands, FR: France, PL: Poland`. **For any other code, keep the raw two-letter code as-is** (e.g. `IL`, `JP`). Percentages with one decimal.
- **yMax for both horizontal charts** — `((floor(maxPercentage/10) + 1) * 10)`, with a floor of 10. So 34.4 → 40, 7.5 → 10, 22.0 → 30.
- **Avg Watch Time** — `averageViewDuration` is seconds (float). Render as `{m}m {s}s` (e.g. `5m 34s`). If minutes is 0, render `{s}s`.
- **Engagement Rate** — `(likes + comments) / views * 100`, two decimals, trailing `%`.

## Step 4: Replace the block in the sponsor page

Use the Edit tool on `content/sponsor/_index.md` to replace everything from `<!-- SPONSOR_ANALYTICS_START -->` through `<!-- SPONSOR_ANALYTICS_END -->` (inclusive) with the new block. Leave the rest of the file untouched.

## Step 5: Verify and report

- Run `git diff content/sponsor/_index.md` and show the user a short summary: subscriber delta, view delta, "Last updated" date, anything that looks anomalous (e.g. a country dropping out of the top 10, large engagement-rate swing).
- **Do not commit or push.** The user will review and commit themselves.

## Common failures

- `invalid_grant` on token refresh → refresh token revoked; user must re-auth via `youtube-automation`.
- Empty `rows` in an analytics response → window too short, channel ID wrong, or analytics-readonly scope missing. Stop and surface the raw response.
- Missing `client_secret.json` → see "Restoring credentials" below.

## Restoring credentials

`client_secret.json` is gitignored. To restore it, extract the contents of the cluster Secret `youtube-automation-oauth-client` (mounted in production at `/client_secret.json`):

```bash
PATH="/nix/store/$(ls /nix/store | grep '^[a-z0-9]\+-gke-gcloud-auth-plugin-' | head -1)/bin:$PATH" \
KUBECONFIG=~/code/dot-ai-infra/kubeconfig.yaml \
  kubectl -n youtube-automation get secret youtube-automation-oauth-client \
    -o jsonpath='{.data.client_secret\.json}' | base64 -d \
    > .claude/skills/generate-analytics/client_secret.json
```

The `gke-gcloud-auth-plugin` PATH dance is needed because the kubeconfig auths against GKE via an exec plugin that isn't on the default macOS PATH but is available through the nix store via `dot-ai-infra`'s devbox.

If the cluster is also gone, restore from Google Cloud Console (project listed in `client_secret.json`'s `project_id` field): OAuth client → Download JSON.
