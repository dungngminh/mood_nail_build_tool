# Webhook Setup Instructions

To trigger this workflow when the `dungngminh/mood_nail_build` repository has new commits to the `develop` branch, you need to set up a webhook or GitHub Action in the source repository.

## Option 1: Repository Dispatch from Source Repository (Recommended)

Add this workflow to the `dungngminh/mood_nail_build` repository in `.github/workflows/trigger-deploy.yml`:

```yaml
name: Trigger Deployment

on:
  push:
    branches:
      - develop

jobs:
  trigger-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger deployment workflow
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.DEPLOY_TOKEN }}
          repository: dungngminh/moon_nail_build_tool
          event-type: develop-push
          client-payload: |
            {
              "ref": "${{ github.ref }}",
              "sha": "${{ github.sha }}",
              "repository": "${{ github.repository }}"
            }
```

## Option 2: Webhook Configuration

1. Go to the source repository: https://github.com/dungngminh/mood_nail_build
2. Navigate to Settings → Webhooks
3. Click "Add webhook"
4. Set Payload URL to: `https://api.github.com/repos/dungngminh/moon_nail_build_tool/dispatches`
5. Set Content type to: `application/json`
6. Set Secret to your webhook secret (store in GitHub Secrets as `WEBHOOK_SECRET`)
7. Select "Let me select individual events" and choose "Push"
8. Set Active to true

## Required Secrets

You need to set up these secrets in the **build tool repository** (`moon_nail_build_tool`):

### GitHub Secrets (Repository Settings → Secrets and variables → Actions)

1. `VERCEL_TOKEN` - Your Vercel API token
2. `VERCEL_PROJECT_ID` - Your Vercel project ID
3. `VERCEL_ORG_ID` - Your Vercel organization ID
4. `PRIVATE_REPO_TOKEN` - GitHub Personal Access Token with `repo` scope to access the private repository
5. `DEPLOY_TOKEN` - GitHub Personal Access Token with `repo` scope (for Option 1, can be the same as PRIVATE_REPO_TOKEN)

### For the Source Repository

If using Option 1, add this secret to `dungngminh/mood_nail_build`:

1. `DEPLOY_TOKEN` - GitHub Personal Access Token with `repo` scope to trigger workflows in other repositories

## Testing

You can manually trigger the workflow using the "workflow_dispatch" trigger:

1. Go to Actions tab in this repository
2. Select "Deploy Flutter Web to Vercel" workflow
3. Click "Run workflow"
4. Optionally specify a git ref (branch, tag, or commit SHA)

## Notes

- The workflow will checkout the `develop` branch from the source repository by default
- Build artifacts are archived on failure for debugging
- The workflow includes deployment status notifications
- **Important**: Since `dungngminh/mood_nail_build` is a private repository, you must use a Personal Access Token with `repo` scope to access it

## Creating a Personal Access Token

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Give it a descriptive name like "Moon Nail Build Tool Access"
4. Select the `repo` scope (full control of private repositories)
5. Set an appropriate expiration date
6. Copy the token and add it as `PRIVATE_REPO_TOKEN` secret in your build tool repository
