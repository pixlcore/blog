<!-- Title: Coming Soon: xyOps™ -->
<!-- Summary: A next-generation workflow automation system, with job scheduling, monitoring, alerting, and ticketing all built right in. -->
<!-- Author: jhuckaby -->
<!-- Date: 2025/11/01 -->
<!-- Tags: Announcements -->

<p style="margin-bottom:30px;"><img src="/images/blog/xyops/workflow-edit.webp" class="img-shadow"></p>

xyOps™ is a next-generation system for job scheduling, workflow automation, server monitoring, alerting, and incident response -- all combined into a single, cohesive platform. It's built for developers and operations teams who want to control their automation stack without surrendering data, freedom, or visibility. xyOps doesn't hide features behind paywalls or push telemetry back to anyone. It's open, extensible, and designed to run anywhere.

## Beta Version Released!

Version 0.9 Beta is now available for testing:

https://github.com/pixlcore/xyops

Let me know what you think!

## The Idea Behind xyOps

Most automation platforms focus on workflow orchestration -- they run tasks, but they don't really help you see what's happening behind them. xyOps takes it further. It doesn't just schedule jobs; it connects them to real-time monitoring, alerts, server snapshots, and ticketing, creating a single, integrated feedback loop. When an alert fires, the email includes the running jobs on that server. One click opens a snapshot showing every process, CPU load, and network connection. If a job fails, xyOps can open a ticket with full context -- logs, history, and linked metrics. Everything in xyOps talks to everything else, so you can trace an issue from detection to resolution without ever leaving the system.

## Smarter Job Scheduling

<p><img src="/images/blog/xyops/active-jobs-many@2x.webp" class="img-shadow"></p>

At its core, xyOps features a next-generation scheduler that replaces cron with flexibility and safety. You can define multiple schedules for a single event, attach blackouts, and even extend scheduling behavior with plugins.  

Scheduling options include:

- Repeating jobs with time intervals or cron-style syntax.
- Single-shot jobs for one-time tasks.
- Import crontabs, seamlessly converted into xyOps events.
- Date ranges for start/end dates.
- Blackout ranges for scheduled maintenance or vacations.
- Catch-Up mode to guarantee all missed jobs run even after an outage.
- Precise mode for running jobs at exact seconds.

Also, each event can enforce **limiters** -- restricting CPU, memory, runtime, or log size -- ensuring that runaway jobs never consume a system.

## Compose Visual Workflows

<p><img src="/images/blog/xyops/workflow-edit-2@2x.webp" class="img-shadow"></p>

The visual workflow editor transforms complex, multi-step operations into clear, connected pipelines. Each node represents a job, trigger, condition, or action, and data can flow freely between them. You can run steps sequentially or in parallel, pass structured JSON or files, and attach safety controls such as retries, timeouts, or resource caps. The editor is designed to let you experiment and iterate quickly -- create, test, modify, and visualize your logic before running. When a workflow runs, you can watch every step unfold live with real-time logs and metrics.

A workflow can:

- Chain multiple jobs in sequence or run them in parallel.
- Pass JSON data or files between steps.
- Attach conditional logic, retries, and timeouts.
- Trigger external actions such as webhooks or notifications.
- Split data into separate serial or parallel jobs.
- Join data from multiple nodes back into one.
- Multiplex a job across a set of servers.
- Evaluate expressions to make flow decisions.
- Store and fetch permanent data for jobs using our storage bucket system.

While workflows are running, xyOps provides real-time visibility -- every job shows progress, logs, and output as it happens.

## Monitoring Everything That Matters

<p><img src="/images/blog/xyops/mem-cpu-details@2x.webp" class="img-shadow"></p>

xyOps also acts as a full-fledged monitoring system. It can track CPU, memory, disk usage, network throughput, and log growth, but you're not limited to system metrics -- any custom command or plugin can emit its own values. These metrics feed into dashboards that update in real time and automatically scale from hourly snapshots to yearly trends. Developers can group servers logically, compare performance across clusters, and export data for external reporting. All alerts are fully scriptable, so an unexpected spike in memory can trigger a webhook, send a message to Slack, or open a ticket for follow-up.

You can:

- Define custom monitors and alert expressions.
- Monitor and graph your own metrics from your own apps.
- Receive alerts via email, webhook, or any Plugin (Slack, Discord, etc.).
- Auto-create tickets or trigger recovery jobs on failure.
- Snooze alerts to prevent noise during known maintenance windows.

Every alert includes a "snapshot" of the system at the moment it fired, which contains every a full picture of the server, including all running processes, and every open network socket.  This is invaluable for troubleshooting an issue that occurred overnight, long after your servers recovered.

<p><img src="/images/blog/xyops/job-monitors@2x.webp" class="img-shadow"></p>

Monitor CPU, memory, disk and network, at the job level, at the server level, and at the group level.  View historical monitors from hourly, daily, monthly, to yearly.  And you can add your own custom monitors to track any metrics you want, as well as add custom alert triggers on them.

When something does go wrong, xyOps can take you from an alert to the root cause -- and even to an automated remediation job -- in seconds.

## Fleet Management and Redundancy

<img src="/images/blog/xyops/add-server-dialog@1x.webp" srcset="/images/blog/xyops/add-server-dialog@1x.webp 1x, /images/blog/xyops/add-server-dialog@2x.webp 2x" class="img-right img-shadow">

Adding worker servers to xyOps is as simple as running a one-line install command. Our agent software is available for macOS, Linux, Windows (even Raspberry Pi), and new servers automatically register themselves with the cluster. You can label servers, assign icons for easy recognition, and group them manually or by hostname pattern match.

Our satellite agent, whom we lovingly call **xySat**, is a lightweight background daemon that ships as a pre-compiled binary with **zero dependencies**.  It allows to you run jobs on any of your servers, while providing active monitoring and alerting.  The agent also requires **no open network listeners or ports** -- it establishes a single outbound connection to the master node and that's it.

For large deployments, xyOps handles redundancy and failover automatically: when a master node goes offline, the system elects a new leader and continues running jobs without interruption. It scales cleanly to thousands of agents while maintaining sub-second coordination.

For ephemeral servers (autoscale, etc.) you can easily add xySat into your initial server bootstrap process, where new servers come up and automatically connect to the cluster, and authenticate themselves via API Key.

## Plugins and Extensibility

<img src="/images/blog/xyops/plugin-new-param-dialog@1x.webp" srcset="/images/blog/xyops/plugin-new-param-dialog@1x.webp 1x, /images/blog/xyops/plugin-new-param-dialog@2x.webp 2x" class="img-right img-shadow">

Everything in xyOps is built around plugins -- small, language-agnostic modules that communicate via JSON over STDIO. This makes it trivial to integrate any tool or system you already use.

Types of plugins include:

- **Event Plugins** -- run custom code or jobs.
- **Monitor Plugins** -- emit metrics to be graphed and alerted on.
- **Action Plugins** -- run custom code based on a job result.
- **Schedule Plugins** -- extend how and when jobs are scheduled.

You can also add custom parameters to Plugins, which can be text fields, text areas, drop-down menus, checkboxes, date selectors, and more.  Parameters can be marked as required, and even locked so only administrators can edit them.

A Plugin Marketplace will launch alongside xyOps, featuring integrations such as AWS S3, GitHub, Google Sheets, Twilio, Slack, Discord, Atlassian, Notion, Cloudinary, BlueSky, Mastodon, OpenAI, Replicate, Telegram, and Linear -- plus community-contributed extensions you can install or fork freely.

## Built-In Ticketing and Incident Response

<p><img src="/images/blog/xyops/ticket-attribs@2x.webp" class="img-shadow"></p>

xyOps includes a lightweight but capable ticketing system designed for engineers. When an alert fires or a job fails, a ticket can be created automatically, complete with logs, metrics, and relevant server details.

Tickets can also trigger new jobs -- perfect for automated recovery or CI/CD workflows. Because the system's API is fully open, external tools can create or resolve tickets programmatically, letting xyOps serve as a bridge between monitoring, automation, and incident management.

Compose your tickets in Markdown format for rich styling, and attach files as well.  Tickets can also have categories, tags and comments, for full team collaboration.

## Reports and Data Visualization

<p><img src="/images/blog/xyops/perf-metrics@2x.webp" class="img-shadow"></p>

In xyOps, jobs aren't black boxes -- they're living, streaming processes that can report their own progress, output, and metrics in real time. Every job detail page updates continuously while it runs, giving you instant visibility into what's happening, how far along it is, and what data it's producing. xyOps turns job output into a visual, interactive experience rather than a wall of text.

Key capabilities include:

- **Live progress reporting.** Jobs can emit a progress value (0-100) at any time, which instantly updates the UI's progress bar and estimated time remaining.
- **Rich, dynamic output.** Jobs can stream tables, Markdown, or full HTML as they run -- all rendered immediately in the browser, no refresh required.
- **Custom performance metrics.** Jobs can output JSON-formatted metrics that xyOps automatically visualizes as pie and bar charts, making it easy to surface custom KPIs.
- **Dynamic tagging.** Jobs can add tags mid-execution, allowing completion actions, alerts, or filters to trigger automatically when certain conditions are met.
- **Historical visibility.** When a job completes, its reports, metrics, and charts are archived and can be reviewed, shared, or exported at any time.

Together, these features make every job in xyOps a self-documenting process -- observable as it runs, reviewable after it finishes, and integrated into the larger operational picture.

## Security, Privacy, and Control

All of xyOps's features are available in self-hosted mode, and no telemetry or license verification ever leaves your network. Air-gapped installations are fully supported. Authentication is handled through API keys for programmatic access and optional SSO for enterprise deployments.

Data export and import are first-class operations -- you can take your configuration, metrics, and history with you at any time.

xyOps comes with a built-in secrets manager with strong encryption.  Use this to store sensitive information such as credentials, and then assign them to specific categories, workflows, and/or plugins.

## Pricing and Availability

xyOps will launch in early 2026 with three distribution options:

- The **Free Tier** offers the complete platform under the BSD 3-Clause license for anyone who prefers to self-host.
	- This is the full application with no strings attached, and **all** features available, forever.
- The **Professional Tier** provides managed cloud hosting with automatic updates, backups, and standard support for $100 per month or $1,000 per year. 
- The **Enterprise Tier** extends that with on-premise assistance, SSO setup, SLA (uptime guarantee), dedicated account management, and one-hour priority support for $1,000 per month or $10,000 per year.

No features will ever be removed or restricted from the open-source release -- the commercial tiers simply exist for professional support, and those who would rather not maintain infrastructure themselves.

## Launching Soon

Version 0.9 Beta is now available for testing:

https://github.com/pixlcore/xyops

Demo videos will follow soon, showing the visual workflow editor, job viewer, Plugin Marketplace, and live monitoring dashboards. Developers who want early access or to contribute plugins can follow the project on GitHub and join the discussion.

Stay tuned. The next generation of open automation is almost here!
