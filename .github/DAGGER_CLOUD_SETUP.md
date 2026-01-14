# Dagger Cloud Setup for CI

This document explains how to configure Dagger Cloud for the auto-publish workflow.

## Prerequisites

- A Dagger Cloud account (sign up at https://dagger.cloud)
- Admin access to this GitHub repository

## Setup Steps

### 1. Obtain Your Dagger Cloud Token

1. Log in to [Dagger Cloud](https://dagger.cloud)
2. Navigate to your organization's settings page (click the cogwheel icon)
3. Select **Tokens** from the menu
4. Click the eye icon to view your token
5. Copy the token value

### 2. Add Token to GitHub Secrets

1. Go to your repository on GitHub: https://github.com/sylvester-francis/Sentry
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Set the following:
   - **Name**: `DAGGER_CLOUD_TOKEN`
   - **Secret**: Paste your Dagger Cloud token
5. Click **Add secret**

### 3. Verify Configuration

Once the secret is added, the workflow will automatically use Dagger Cloud on the next push to main.

You can verify it's working by:
1. Making a small change to any `.dagger/*.go` file
2. Committing and pushing to main
3. Checking the Actions tab for the workflow run
4. The workflow should show Dagger Cloud connection in the logs

## What Dagger Cloud Provides

- **Caching**: Faster builds with distributed caching
- **Telemetry**: Insights into your pipeline performance
- **Logs**: Centralized logs for all Dagger operations
- **Collaboration**: Share pipeline insights with your team

## Security Notes

- **Never commit the token**: The token must be stored as a GitHub Secret
- **Restrict access**: Only give the token to trusted CI environments
- **Rotate regularly**: Generate new tokens periodically for security

## Troubleshooting

### Workflow fails with authentication error
- Verify the secret name is exactly `DAGGER_CLOUD_TOKEN`
- Check that the token is valid in Dagger Cloud settings
- Ensure the token hasn't expired

### Dagger Cloud not showing data
- Confirm the `DAGGER_CLOUD_TOKEN` environment variable is set in all workflow steps
- Check Dagger Cloud organization settings for proper permissions

## Additional Resources

- [Dagger Cloud Documentation](https://docs.dagger.io/reference/configuration/cloud/)
- [GitHub Actions Secrets Documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Dagger Cloud Dashboard](https://dagger.cloud)
