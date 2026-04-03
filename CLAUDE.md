# Claude Instructions for pyxproj

## Organization Context

- **Organization:** insitro (GitHub org)
- **Common usernames:**
  - joe-insitro (Joe Marrama)
  - zack-insitro (Zack Phillips)

## GitHub Operations

### Setup

**Token location:** `~/.claude/settings.json` as `GITHUB_TOKEN`

**Current configuration:**
- Account: `zack-insitro`
- Token scopes:
  - `repo` - Full repository access
  - `read:org` - Organization data
  - `read:user` - User profile
  - `user:email` - Email addresses
  - `gist` - Gist access

The token is automatically available as the `GITHUB_TOKEN` environment variable in all sessions.

### Finding PRs Merged by a Specific User

When asked to find PRs merged by someone (e.g., "show me PRs joe marrama merged"):

**Important:** The GitHub Issues Search API does NOT include the `merged_by` field. You must:

1. Use the Issues Search API to find merged PRs in a date range for specific repos
2. Then fetch each PR individually via the Pull Requests API to get the `merged_by` field

**Example workflow:**

```bash
# Step 1: Search for merged PRs in date range
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/issues?q=repo:insitro/core+is:pr+is:merged+merged:2026-03-27..2026-04-03&per_page=100"

# Step 2: For each PR, fetch details to get merged_by
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/insitro/core/pulls/{number}"
```

**Key repos to check for insitro org:**
- core (most active)
- insitro-data
- groot
- jupyter_notebooks
- target_explorers
- bulkRNA
- redun-private
- lims-brain

### Refreshing GitHub Token

If the token expires or needs regeneration:

1. Create new token: https://github.com/settings/tokens/new
2. Select required scopes (minimum: `repo`, `read:org`, `read:user`, `user:email`)
3. Update `~/.claude/settings.json` with the new token value

## Google Workspace Operations

### Setup

Google Workspace access uses OAuth 2.0 with a personal Google Cloud project to bypass organizational restrictions.

**Credentials location:** `~/.claude/google_credentials.json`
**Environment variable:** `GOOGLE_APPLICATION_CREDENTIALS`
**Project:** `teak-amphora-492213-i1`
**Account:** `zack@insitro.com`

**Enabled APIs:**
- Google Drive API
- Google Docs API
- Google Sheets API
- Google Slides API

**Why this setup?** The organization (insitro) blocks default gcloud application credentials for security reasons. This personal OAuth approach bypasses that restriction by using a personal Google Cloud project with "External" OAuth credentials.

### Finding Google Docs/Sheets/Slides

**Quick search using helper script:**

```bash
python3 /Users/zack/.local/bin/search_gdrive.py [search terms...]
```

**Examples:**
```bash
# Find Swimaging 2026Q2 planning doc
python3 /Users/zack/.local/bin/search_gdrive.py swimaging 2026q2

# Find any planning documents
python3 /Users/zack/.local/bin/search_gdrive.py planning 2026
```

The script searches both file names and full text content, returns results sorted by modification time (most recent first), and includes direct links.

**Common document patterns:**
- Swimaging planning docs: "Swimaging 2026Q[1-4]"
- ImagingML planning docs: "ImagingML 2026Q[1-4]"
- Allocation sheets: "Imaging 2026 Allocations", "ImagingML 2026Q[X] Allocations"

### Using Google APIs in Python

```python
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build
import os

# Load credentials
creds = Credentials.from_authorized_user_file(
    os.path.expanduser('~/.claude/google_credentials.json')
)

# Use the service
service = build('drive', 'v3', credentials=creds)
results = service.files().list(pageSize=10).execute()
```

### Refreshing Google Credentials

OAuth tokens auto-refresh when needed. If manual refresh is required, re-authenticate with the OAuth setup script (stored in project setup).

## AWS Bedrock Access

**Profile:** `bedrock`
**Region:** `us-west-2`
**Account:** `298579124006`

AWS credentials are managed via AWS SSO. The default profile in `~/.aws/config` points to the bedrock configuration for automatic discovery.

**Refresh session:**
```bash
aws sso login --profile bedrock
```

## Troubleshooting

### GitHub API Rate Limiting

Check rate limit status:
```bash
curl -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/rate_limit
```

### Google OAuth Consent Required

If you see "Access blocked" errors, verify:
1. Your email is added as a test user in the Google Cloud Console
2. OAuth consent screen is configured for "External" type
3. All required scopes are enabled in the consent configuration

### Session Expiration

- **AWS:** Re-run `aws sso login --profile bedrock`
- **Google:** Tokens auto-refresh, but if issues persist, re-run authentication script
- **GitHub:** Token doesn't expire unless manually revoked; regenerate if needed

## Summary

All credentials are stored in `~/.claude/settings.json` and loaded automatically:
- `GITHUB_TOKEN` for GitHub API access
- `GOOGLE_APPLICATION_CREDENTIALS` for Google Workspace access
- `AWS_PROFILE=bedrock` for AWS/Bedrock access

Agents can access these services directly without additional authentication steps.
