# daily-pr-email

This repository contains a GitHub Actions workflow that periodically collects open Pull Requests (PRs) from one or more GitHub repositories and publishes a digest as a **new GitHub Issue** each run.

GitHub’s own notification system can then deliver that Issue to your inbox (e.g., Outlook) as email—no SMTP setup required.

## What it does

On a schedule (currently every 5 minutes for testing), the workflow:

1. Queries a configured list of repositories for **open PRs**
2. Filters out:
   - **Draft PRs**
   - PRs whose **title starts with `WIP`** (case-insensitive)
3. Generates a short Markdown digest containing only:
   - PR title
   - Link to PR
4. Creates a **new Issue** in this repository containing the digest

Because each run creates a new Issue, you can receive **distinct emails** (one per run) via GitHub Notifications.

## Why Issues?

GitHub can email you about Issue activity using your GitHub notification settings. This avoids having to configure SMTP credentials or a third-party email provider inside Actions.

## How to use

### 1) Configure which repos to scan

Edit the workflow file:

- `.github/workflows/pr-digest.yml`

Find the `REPOS` environment variable and add repositories as a **space-separated** list in `owner/repo` format:

    env:
      REPOS: "uclibs/staff-directory-23 uclibs/ucrate"

### 2) Configure how often it runs

In the same workflow file, update the cron schedule:

    on:
      schedule:
        - cron: "*/5 * * * *"  # every 5 minutes (testing)

Notes:

- Cron is interpreted in **UTC**.
- GitHub scheduled workflows are not a real-time cron service; some delay/jitter is normal.

When you’re done testing, change this to a daily/weekday schedule (example: weekdays at 13:00 UTC):

    - cron: "0 13 * * 1-5"

### 3) Enable email delivery (GitHub Notifications → your Outlook inbox)

This workflow itself does not send email. GitHub does.

To receive emails:

1. In GitHub, go to **Settings → Emails**
   - Add your work email address and verify it
   - (Recommended) enable **Keep my email addresses private**
2. Go to **Settings → Notifications**
   - Enable **Email notifications**
   - Select your work email address as the destination
3. In this repository, click **Watch → Custom → Issues**
   - This makes “new Issue created” generate a notification email.

If your organization filters GitHub emails, you may need to check Outlook quarantine/junk folders or allowlist GitHub notifications.

### 4) Run it manually (for testing)

1. Go to this repo on GitHub
2. Click **Actions**
3. Choose workflow **Daily PR Email**
4. Click **Run workflow**

A successful run should create a new Issue in the **Issues** tab.

## Output format

Each Issue contains a digest like:

- A header with the run timestamp (UTC)
- A section per repo
- A bullet list of PR links:

    ### uclibs/ucrate
    - [Fix build pipeline](https://github.com/uclibs/ucrate/pull/123)
    - [Update dependencies](https://github.com/uclibs/ucrate/pull/130)

## Notes / limitations

- This repository must have **GitHub Actions enabled**.
- Scheduled workflows only run based on the workflow file on the repo’s **default branch** (`main`).
- Creating a new Issue every run is great for testing, but will generate a lot of Issues and emails if the schedule is frequent. Reduce the cadence once confirmed.

## Customization ideas

- Include PR author, age, or labels
- Exclude additional title prefixes (e.g., `DO NOT MERGE`, `[WIP]`)
- Only create an Issue if the digest changes (reduces inbox noise)
- Switch from “new Issue per run” to “update a single rolling Issue”

## Repository contents

- `.github/workflows/pr-digest.yml` — scheduled workflow that generates the digest and creates an Issue

## Security / privacy

- No email addresses are stored in this repository.
- Email delivery is handled by GitHub Notifications, configured in your GitHub account settings.
- If this is a public repo and you commit from your local machine, consider using GitHub’s `noreply` email address for commits to avoid exposing your work email in commit metadata:
  - https://github.com/settings/emails