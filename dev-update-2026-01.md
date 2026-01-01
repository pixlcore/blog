<!-- Title: xyOps Dev Update: January 2026 -->
<!-- Summary: A quick development update on xyOps, including what was completed in December, and the long-awaited beta release! -->
<!-- Author: jhuckaby -->
<!-- Date: 2026/01/01 -->
<!-- Tags: Announcements, Updates -->

# Overview

It's here!  **xyOps Beta v0.9 is live**, and ready for you to take it for a test drive:

https://github.com/pixlcore/xyops#readme

Let me know what you think!

# Community

Please join the xyOps community and help improve xyOps:

## GitHub Discussions

Have a question or suggestion?  Post a topic on our GitHub Discussion Board:

https://github.com/pixlcore/xyops/discussions

## Reporting Bugs

To report a bug with xyOps, or request a new feature, please use GitHub Issues:

https://github.com/pixlcore/xyops/issues

Please search the existing issues first, to make sure your issue isn't already being worked on.

## Reddit

Join our subreddit to get help from the xyOps community:

https://reddit.com/r/xyOps/

## Discord

Join our Discord server to chat with community members in real-time:

https://discord.gg/FTzqmbGbdd

## Social Media

Follow us on social media:

- [Bluesky](https://pixlcore.bsky.social/)
- [Mastodon](https://mastodon.social/@pixlcore)
- [LinkedIn](https://linkedin.com/company/pixlcore)

## RSS feed

Keep up with xyOps news via your favorite RSS reader:

https://pixlcore.com/feed.rss

# December Dev Log

A lot was completed in December!  Here are the major items:

- Added Magic Link triggers, with custom landing pages!
- Added Docker Plugin!
- Added inline documentation viewer!
- Added page descriptions.
- Accessibility pass complete.
- Unit tests complete.

And the rest:

- xyOps can now launch an instance of xySat (our remote satellite agent) inside the main Docker container.
	- This is great for people getting up and running quickly, but not recommended for production.
- Plugin Parameters, Event Fields and Toolsets now support a JSON field (for complex objects).
- The JSON editor now validates and reports errors in the browser.
- Added configurable Hot Keys (Keyboard Shortcuts).
- Added "Clone" feature on several key pages.
- Added "Apply Tags" Job Action.
- Added "Daily Max Limit" system for events.
- Convert trigger plugins into "modifiers" (must be accompanied by some kind of scheduler trigger)
	- This makes them much more flexible.
- Now minifying all client-side production dist code.
- Support for automated docker container upgrades.
- Allow exporting events and workflows with dependencies in tow (multi-item XYPDF file)
- Support importing files with multiple items in them.
- Added "stream_job" API, which can stream live job updates using Server-sent events.
- Workflows: All sub-jobs now have access to job.workflow.params (user fields supplied at workflow launch)
- Workflows: Added "Note" node (custom markdown annotations for your flows).
- Workflows: New "Tool" system, with a move tool and a select tool.
- Workflows: During runs, you can now select nodes to highlight all that node's jobs.
- Added proper HTTP response codes for all APIs.
- Added sort by job elapsed time.
- Allow custom email markdown content in actions.
- Wrote more documentation (support guide, welcome guide, more format specs).
- Added a bunch of quality-of-life improvements.
- Fixed a ton of small bugs.
- Added a CHANGELOG.

# January 2026

My big TODO items for January are:

- Fix all major issues that come up during the beta test.
- Finish the Plugin Marketplace.
- Add a Configuration Editor right in the UI.
- Add a Log Viewer in the UI.
- Record a bunch of YouTube videos.
