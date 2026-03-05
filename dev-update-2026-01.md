<!-- Title: xyOps Dev Update: February 2026 -->
<!-- Summary: A quick development update on xyOps, including what was completed in January 2026. -->
<!-- Author: jhuckaby -->
<!-- Date: 2026/02/01 -->
<!-- Tags: Announcements, Updates -->

# Overview

What an exciting month!  First of all, let me say **thank you** to all the beta testers and early adopters!  Your bug reports and feature requests have been incredibly helpful, and they are making xyOps better every day.

January was all about polishing the beta and shipping a lot of user-facing improvements.  The two biggest features are now done: the **Configuration Editor** and the **Log Viewer** in the UI.  I also squeezed in a solid batch of API and workflow enhancements, plus a long list of bug fixes and doc updates.

# Release Overview

We shipped **44 point releases** of xyOps in January, from [v0.9.6](https://github.com/pixlcore/xyops/releases/tag/v0.9.6) through [v0.9.49](https://github.com/pixlcore/xyops/releases/tag/v0.9.49).  Most of the work focused on UI features, operational polish, and API expansion.  See the [Changelog](https://github.com/pixlcore/xyops/blob/main/CHANGELOG.md) for full release details.

In general I'm really happy with the state of the app.  It's stable, performant, and has tons of great features.  Here are a few we just added:

# Major Features

- **New Configuration Editor** -- Edit your xyOps configuration right in the UI.
- **New Log Viewer** -- View or search through any xyOps log or archives in the UI.
- **Schedule Trigger Tags and Params** -- Attach tags and/or event params onto each trigger for an event.
- **Bulk export for job, ticket, alert, and snapshot searches**, with CSV, TSV, NDJSON, and optional gzip
- **Bucket Menu** for dynamic parameter options sourced from storage buckets
- **Media slideshow** for job outputs that include images, video, or audio
- **Job search and tagging enhancements**, including new system tags and a selectable property list for API searches
- **Event and workflow UX upgrades**, with better summaries, locale-aware timing display, and workflow map improvements

# Other Notable Changes

- New configuration and log search APIs: [admin_get_config](https://docs.xyops.io/#Docs/api/admin_get_config), [admin_update_config](https://docs.xyops.io/#Docs/api/admin_update_config), [admin_search_logs](https://docs.xyops.io/#Docs/api/admin_search_logs)
- New data APIs: [write_bucket_data](https://docs.xyops.io/#Docs/api/write_bucket_data) and [bulk_search_export](https://docs.xyops.io/#Docs/api/bulk_search_export)
- Search improvements: optional `select` parameter for [search_jobs](https://docs.xyops.io/#Docs/api/search_jobs)
- Programmatic bucket data access via [write_bucket_data](https://docs.xyops.io/#Docs/api/write_bucket_data)
- Log archive auto-delete via [log_archive_keep](https://docs.xyops.io/#Docs/config/log_archive_keep)
- API hardening and parameter naming cleanup for all APIs that handle file uploads.
- New system tag for job with files ("Has Files"), so you can search for all jobs that have input or output files attached.
- Job details now include an **Edit Event** button with consolidated ticket actions
- Event List now shows a **Tags** column
- Send a test email from the system page, with full debug output

# Documentation and Ops

- Expanded hosting docs, config docs, and API docs, plus a batch of formatting and clarity fixes
- Satellite install and upgrade scripts were redesigned to behave as first-class systemd services

# New Marketplace Plugins

The xyOps Marketplace now has plugins!  Thank you to all the authors who contributed their work:

- **[On Holiday](http://github.com/UNICodehORN/xyplug-on-holiday)** by UNICodehORN: Skip or execute xyOps schedules on public holidays using OpenHolidaysAPI.
- **[Sun Trigger](http://github.com/UNICodehORN/xyplug-sun-trigger)** by UNICodehORN: Execute xyOps schedules based on sun using public sunset API.
- **[PowerShell Plugin](http://github.com/nickdollimount/xyplug-powershell)** by Nick Dollimount: Execute PowerShell script code using xyOps events.
- **[MSSQL Query](https://github.com/talder/xyOps-MSSQL-Query)** by Tim Alderweireldt: Execute SQL queries against Microsoft SQL Server databases using PowerShell and dbatools. Supports encrypted connections, certificate validation, CSV/JSON export, and automatic row limiting.
- **[MSSQL Information](https://github.com/talder/xyOps-MSSQL-Information)** by Tim Alderweireldt: Collect comprehensive information from Microsoft SQL Server instances including server details, Availability Groups, database inventory, and user accounts. Outputs multiple tables and structured JSON for workflows.
- **[Bluesky Social](http://github.com/pixlcore/xyplug-bluesky)** by PixlCore: Access your Bluesky social profile, read your timeline, make posts, leave likes, and more.
- **[Stagehand](http://github.com/pixlcore/xyplug-stagehand)** by PixlCore: An AI-powered browser automation framework for xyOps.  Drive a headless browser with simple English instructions, take actions, extract data, capture network requests, and even record a video of the whole session.
- **[AI Generation](http://github.com/pixlcore/xyplug-ai)** by PixlCore: An AI plugin which lets you prompt any AI provider and LLM, including local models, and return the result.
- **[Extract Anything](http://github.com/pixlcore/xyplug-extract)** by PixlCore: Extract structured data out of any file, using the amazing Kreuzberg library.
- **[Weather Forecast](http://github.com/pixlcore/xyplug-weather)** by PixlCore: Fetch current conditions and forecasts from the free Open-Meteo API.
- **[Replicate AI](http://github.com/pixlcore/xyplug-replicate)** by PixlCore: Generate high-quality AI images, video, or audio using the Replicate API.

# Fun Stats

- **xyOps Git Commits in January:** 333
- **Lines changed:** 10,762 total lines, 8,874 added, 1,888 removed, net +6,986
- **Total line counts:**
	- JavaScript: 60,335 lines
	- JSON: 23,300 lines
	- Markdown: 14,622 lines
	- CSS: 4,603 lines
	- HTML: 435 lines
	- Other: 989 lines
- **Grand Total:** 104,284 lines

# Looking Ahead

My top priorities for February are to finish and release v1.0, launch the paid support tiers, and start a big marketing push.

Also, of course, continue tightening the UI and API based on beta feedback.

# Community

Please join the community and help improve xyOps:

- **GitHub Discussions:** [Discussion Board](https://github.com/pixlcore/xyops/discussions)
- **Reporting Bugs:** [GitHub Issues](https://github.com/pixlcore/xyops/issues) (please search existing issues first)
- **Reddit:** [r/xyOps](https://reddit.com/r/xyOps/)
- **Discord:** [Join the Server](https://discord.gg/FTzqmbGbdd)
- **Social Media:** [Bluesky](https://pixlcore.bsky.social/), [Mastodon](https://mastodon.social/@pixlcore), [LinkedIn](https://linkedin.com/company/pixlcore)
- **RSS Feed:** [pixlcore.com/feed.rss](https://pixlcore.com/feed.rss)

Hope to see you there!  Come and say hi!
