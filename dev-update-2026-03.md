<!-- Title: xyOps Dev Update: March / April 2026 -->
<!-- Summary: A development update on xyOps, covering March 2026 and the first part of April 2026. -->
<!-- Author: jhuckaby -->
<!-- Date: 2026/04/12 -->
<!-- Tags: Announcements, Updates -->

# Overview

This update covers **March 1, 2026 through April 10, 2026**, and it has been a very busy stretch for xyOps.

The big theme since the v1.0 launch has been operational maturity.  I spent most of this period tightening security, improving upgrades and diagnostics, expanding storage and deployment options, polishing scheduling and workflow behavior, and continuing to grow the Marketplace.  We also shipped several new first-party Plugins that I am really happy with.

# Release Overview

During this window we shipped:

- **33 xyOps releases**, from [v1.0.15](https://github.com/pixlcore/xyops/releases/tag/v1.0.15) through [v1.0.47](https://github.com/pixlcore/xyops/releases/tag/v1.0.47)
- **8 xySat releases**, from [v1.0.11](https://github.com/pixlcore/xysat/releases/tag/v1.0.11) through [v1.0.18](https://github.com/pixlcore/xysat/releases/tag/v1.0.18)

For the xyOps repo alone, this period included **238 commits** and **120 changed files**, with **7,449 lines added** and **8,988 lines removed**.  A lot of that churn was cleanup, hardening, and internal refactoring after the v1.0 release.

As always, see the full changelogs for complete details:

- https://github.com/pixlcore/xyops/blob/main/CHANGELOG.md
- https://github.com/pixlcore/xysat/blob/main/CHANGELOG.md

# Major Improvements

Here are the biggest themes from March and early April:

- **Security and install hardening** got a major pass, including the new CSRF token system, automatic secret key generation on first install, stricter config validation at startup, tighter config file permissions, XSS protection fixes, secret redaction improvements, and stronger safeguards against bad parameter names such as `__proto__`.
- **Security documentation** also improved, with new [THREAT_MODEL.md](https://github.com/pixlcore/xyops/blob/main/THREAT_MODEL.md) and [SECURITY_OVERVIEW.md](https://github.com/pixlcore/xyops/blob/main/SECURITY_OVERVIEW.md) documents added to the repo.
- **Auth and access workflows** got better too, with customizable Magic Link forms, a new custom SSO pre-processing command hook, and an emergency Docker-friendly admin recovery path for containerized installs.
- **Upgrades and diagnostics** are much nicer now.  xyOps can visually flag outdated conductors and workers right in the UI, show version details in upgrade dialogs, generate a diagnostic report for GitHub issues, and attach xyOps / xySat version information into each launched job.
- **Scheduling and queue control** got smarter.  New features include the **Every Nth** schedule modifier, optional job priority to jump the queue, per-server max jobs, group-level default max jobs, and new server selection algorithms (`prefer_first_natural` and `prefer_last_natural`).
- **Workflow behavior** continues to improve, especially around target expressions and queue handling.  Multiplex workflows can now use job target expressions, queued jobs re-evaluate target expressions while waiting, workflow queue namespaces are cleaner, and jobs in the `finishing` state can now be aborted.
- **Revision history and export workflows** got several useful improvements.  Search results and job details can now show previous event titles and revision numbers, historical revisions can be exported directly in XYPDF format, and event status labels can show the last completed run time on hover.
- **Marketplace management** is more polished.  Installed Marketplace plugins are now intentionally clone-only for local customization, the UI can quickly set up Secret Vaults for installed plugins, Marketplace listings render better markdown, mobile layout improved, and GitHub thumbnails now lazy-load to avoid rate-limit issues.
- **Storage and deployment options** expanded further.  xyOps now includes support for the new Postgres storage engine in `pixl-server-storage`, along with sample config and documentation updates.  Docker and satellite configuration docs also received a lot of attention, especially around persistent config mounts and managed satellite keys.

# xySat Highlights

xySat had a strong month as well, with a focus on reliability, configurability, and future monitoring capabilities:

- Basic startup config validation was added, so bad configurations fail loudly and earlier.
- Satellite config handling is more flexible now, with support for custom config file locations and conductor-managed key allowlists.
- Container behavior improved, including better handling of temp directories and custom config paths.
- The HTTP Plugin gained configurable connect and idle timeouts.
- Very basic **GPU / graphics monitoring** landed in v1.0.17, currently disabled by default for now.

Combined with the xyOps-side upgrade and diagnostics work, the overall cluster management story is getting a lot stronger.

# New Marketplace Plugins

I also published four new Marketplace Plugins:

- **[Auto Upgrade](https://github.com/pixlcore/xyplug-upgrade)**: Automatically checks for xyOps and xySat upgrades, generates a markdown report, optionally emails results, and can queue background upgrades using the official documented admin APIs.
- **[Remote SSH](https://github.com/pixlcore/xyplug-ssh)**: A pure Node.js SSH transport plugin that can run remote scripts or forward full xyOps job JSON to remote workers, with support for private keys, passwords, `ssh-agent`, and host fingerprint pinning.
- **[S3 Toolbox](https://github.com/pixlcore/xyplug-s3)**: A full AWS S3 utility plugin for uploading, downloading, copying, moving, listing, grepping, and deleting files, designed to work naturally with xyOps job input and output files.
- **[Whisper Transcription](https://github.com/pixlcore/xyplug-whisper)**: An offline transcription plugin powered by local `whisper.cpp` inside Docker, with no hosted API dependency, baked-in model images, live progress reporting, structured transcript output, and optional subtitle / transcript file generation.

# New Feature Highlights

I wanted to highlight a few key features that were just recently added:

## Max Jobs Per Server

You can now set a *per-server* maximum concurrent job limit.  So for example you can configure some of your underpowered servers to limit the number of jobs they can run concurrently.  This can be configured on the server details page by clicking the "Edit Server" button, or via the [update_server](https://docs.xyops.io/#Docs/api/update_server) API.  The default is no limit.

When a server is maxed out and a new job is starting, the way it works is that the server is "removed" from consideration when the server is being chosen from the event targets.  So any alternate servers with available "slots" will be chosen instead (assuming your event targets have multiple servers or groups).

If no servers are available due to max jobs, the behavior is the same as if the servers were unavailable for other reasons (i.e. servers offline, or blocked due to active alerts, etc.).  If the event has [queuing](https://docs.xyops.io/limits/max-queue-limit) enabled, then the job will automatically queue up until a server becomes available.

You can also set defaults for the max jobs per server at the group level, so you don't have to edit each of your servers individually.  To do this, simply edit the group, and you will see a new "Max Jobs Per Server" field.  If a server is a member of multiple groups, the group with the lowest max jobs per server prevails.

## Custom Job Weight

You can now set an optional "job weight" per event, which is used when selecting a server to run a job.  This is designed to compliment the max jobs per server feature (discussed above), and allows you to limit how many jobs can run on specific servers based on their weight.

As an example, say you have a job that is particularly "heavy", like a video conversion script using ffmpeg.  You can set the weight to something high like `8`, and this effectively treats the job as taking up 8 "slots" when calculating server availability and the "max jobs" server setting.

If a new job would exceed the "max jobs" setting on a server, that server is removed from consideration (just as if it was otherwise unavailable or offline).  If a server's "max jobs" is less than the job weight, it will never be chosen for that job.

By default all jobs have a weight of `1`.  To set a job weight higher, edit the [Max Concurrent Jobs](https://docs.xyops.io/limits/max-concurrent-jobs) limit on the event (or attach a [Limit Node](https://docs.xyops.io/workflows/limit-nodes) in your workflow), and in the dialog you will see a new "Server Job Weight" field.  Note that this weight is **only** used for server selection, and does not affect the job's own concurrent maximum setting.

## Group Priority Targeting

Two new [Event.algo](https://docs.xyops.io/data/event-algo)s (algorithms) were just added for selecting servers for jobs:

- `prefer_first_natural`: Prefer the first server when naturally sorted by the [Event.targets](https://docs.xyops.io/data/event-targets).
- `prefer_last_natural`: Prefer the last server when naturally sorted by the [Event.targets](https://docs.xyops.io/data/event-targets).

These can be used in conjunction with the new [Max Jobs Per Server](https://docs.xyops.io/servers/max-jobs-per-server) feature to effectively implement group priority targeting.  When an event is targeted at multiple groups, and `prefer_first_natural` is selected, the first group of servers will be preferred, *until* they cannot be (i.e. via max jobs per server limits), and only then will the second group will be considered.

Remember that the max jobs setting on servers will effectively "remove" the server from consideration when it is filled up (or cannot fit the new job based on its weight).  With `prefer_first_natural` this will pick the very next server in the group, until all those servers are maxed out (i.e. unavailable for further jobs), and only then will the additional groups be considered.

Note that you can control the order in which groups are sorted inside of the [Event.targets](https://docs.xyops.io/data/event-targets) list by going to the "Groups" page and dragging your groups around to resort them.  Groups higher on the list will appear first in the event target list.

## Priority Job Queuing

Jobs now have an optional [Job.priority](https://docs.xyops.io/data/job-priority) flag.  When this is set to `true` and [queuing](https://docs.xyops.io/limits/max-queue-limit) is enabled on the event, the priority job will effectively "hop" the queue, and be inserted right at the head, so it is processed before all other non-priority jobs.

The priority flag can be set when a job is started manually, or via API.  For manual runs in the UI, the job configuration dialog will show a new "High Priority" checkbox, if [queuing](https://docs.xyops.io/limits/max-queue-limit) is enabled on the event.  For API calls, when using the [run_event](https://docs.xyops.io/#Docs/api/run_event) API, simply include a `priority` property and set it to `true`.

When multiple priority jobs are queued for an event, the ones waiting the longest will be processed first (i.e. FIFO).  Once all the priority jobs are processed, the non-priority jobs are dequeued (also FIFO).

Note that you cannot set the priority flag via [Magic Link](https://docs.xyops.io/triggers/magic-link) triggers (by design).

## Custom QuickMon Plugins

You can now add your own custom QuickMon Plugins.  QuickMon (or "Quick Monitors") run every second and provide a view of the last minute on every server.  The way this works is by "enabling" QuickMon on your existing [Monitor Plugins](https://docs.xyops.io/plugins/monitor-plugins), so they are executed every second as part of the QuickMon fast metrics capture.  To do this, edit the plugin and check the "Include in Quick Monitors" checkbox.  But please read on, because here be footguns.

This means that your Monitor Plugins will be executed every second, potentially on every server (unless you limit where it runs by group selection).  So your Plugin needs to be fast -- like, *really* fast.  Ideally, your Plugin will just grab the contents of a file, rather than executing a complex heavy operation.  Note that QuickMon Plugins have a 1-second timeout, after which the process is killed.

Once your Monitor Plugin has been QuickMon-enabled, you can then add custom graphs to the QuickMon configuration JSON.  Example Monitor Plugin which grabs the CPU temperature from the kernel (available on most Linux flavors):

- **Executable**: `/bin/sh`
- **Script**: `cat /sys/class/thermal/thermal_zone0/temp`

And the JSON configuration for the quick monitor which pulls the value and divides it by 1,000 to get celsius:

```json
{ 
	"id": "_qm_cpu_temperature", 
	"title": "CPU Temperature", 
	"source": "integer(commands.MY_PLUGIN_ID) / 1000", 
	"type": "integer", 
	"suffix": "C"
}
```

To learn more, please read the newly updated [QuickMon](https://docs.xyops.io/monitors/quickmon) documentation.

# Looking Ahead

My focus for the rest of April is continued v1.1 work, more Marketplace growth, and more polish around upgrades, storage, monitoring, and operations.

The post-v1.0 phase has been a lot of fun so far.  The app feels more stable, more capable, and more practical for real-world production use with every release.

Thank you to everyone who reported issues and feature requests!

# Community

Please join the community and help improve xyOps:

- **GitHub Discussions:** [Discussion Board](https://github.com/pixlcore/xyops/discussions)
- **Reporting Bugs:** [GitHub Issues](https://github.com/pixlcore/xyops/issues)
- **Reddit:** [r/xyOps](https://reddit.com/r/xyOps/)
- **Discord:** [Join the Server](https://discord.gg/FTzqmbGbdd)
- **Social Media:** [Bluesky](https://pixlcore.bsky.social/), [Mastodon](https://mastodon.social/@pixlcore), [LinkedIn](https://linkedin.com/company/pixlcore)
- **RSS Feed:** [pixlcore.com/feed.rss](https://pixlcore.com/feed.rss)

Hope to see you there!  Come and say hi!
