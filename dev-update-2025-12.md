<!-- Title: xyOps Dev Update: December 2025 -->
<!-- Summary: A quick development update on xyOps, including what was completed in November, and what is left before release. -->
<!-- Author: jhuckaby -->
<!-- Date: 2025/12/01 -->
<!-- Tags: Announcements, Updates -->

# Overview

Wow, November was quite a month!  I feel like I spent the entire time writing docs, but a few other big things got completed as well.  See below for details.

Okay, so despite my best efforts (16 hour days, 7 days per week, non-stop since April 1st), I don't think I can release the beta in December.  I'm *so close*, but the documentation took a lot longer than I anticipated, and I have a few more key features on the "MUST HAVE" list (see below).  So, here is the new tentative release schedule:

- **January 1, 2026**: Beta Release.
- **February 1, 2026**: Full v1 Release.

I apologize to those expecting a beta sooner, but the app is just not quite there yet.  Also, the holidays will eat up much of my time towards the end of the month, so I really only have 2 weeks left in the year.  But please be assured, I am absolutely firing on all cylinders here.  I am working on xyOps from the moment I wake up to the moment I fall asleep, every single day.  It has consumed my entire life (in the best possible way).  I am **extremely** excited about the app, and I hope you all love it as much as I do.  I hope you will stick with me as I push it over the finish line.

Oh, and if you haven't yet done so, please [sign up to be notified at the bottom of this page](https://xyops.io/), so I can email you when the app releases!

# Completed

The following items were completed in November:

## Documentation

I *finally* finished writing all the xyOps documentation!  I feel like I've been working on this for a year.  I am so happy it is done.  Here is the final tally:

- 39 Chapters
- 16,687 Lines
- 95,032 Words
- 760KB Size

A few documentation items worth noting:

- The **ENTIRE** API is documented.  All 170 endpoints!  Not a single call was left behind.
- All internal data structures are documented, including all properties inside them.
- All protocols and file formats are documented.
- SSO setup is fully documented.
- Multi-master including Nginx load balancing and TLS is fully documented.
- Plugin development and Plugin Marketplace submission are fully documented.

## Remote Plugin Support

Support was added to have certain Plugins run "remotely" (in Docker containers or over SSH, for example).  The idea here is to allow xyOps to still monitor resources for remote processes, and manage input and output files.  To facilitate this, a new **xyOps Remote Job Runner** was developed (nicknamed "xyRun").  Here is the repo:

https://github.com/pixlcore/xyrun

The idea is that when a job is running "remotely" (i.e. not a direct child process of [xySat](https://github.com/pixlcore/xysat)) then we cannot monitor system resources for the job.  Also, input and output files simply do not work in these cases (because xySat expects them to be on the local filesystem where it is running).  xyRun handles all these complexities for you by sitting "in between" your job and xySat.  xyRun should run *inside* the container or on the far end of the SSH connection, where your job process is running.  See [xyRun](https://github.com/pixlcore/xyrun) for more details.

## Stagehand Marketplace Plugin

The first xyOps Marketplace Plugin is now complete!  Here is the repo:

https://github.com/pixlcore/xyplug-stagehand

This Plugin provides an AI-powered browser automation framework for xyOps.  Using it you can drive a headless browser with simple English instructions, take actions, extract data, capture network requests, and even record a video of the whole session.

A headless Chromium is launched and automated locally in a Docker container.  The Plugin does not use any "cloud" browser environments.  Note that if you use any of the AI features, then you need to be careful with sensitive information, as they may be sent to the AI provider.  See the link above for discussion and mitigation techniques.

This Plugin ships as a pre-built Docker container, and uses [xyRun](https://github.com/pixlcore/xyrun) for monitoring resources *inside* the container, as well as file upload/download.

## Secret Key Rotation

Administrators can now safely and easily rotate the xyOps secret key right from the UI.  This generates a new cryptographically secure key, and automatically re-encrypts all secrets, re-authenticates all servers, and updates the key securely across all master peers (for multi-master setups).  This can all be done *while* xyOps is running and requires no restarts whatsoever.

# In Progress

The following features are still in progress:

- **Plugin Marketplace**
	- My goal is to get a basic version of this working for the beta release.  Submission will be manual, but at least users will be able to see a list of available Plugins and install them.
- **Accessibility Pass**
	- Make sure the app works well with screen-readers.  The UI uses custom buttons and other controls, so I need to make sure I have all the correct ARIA tags everywhere.
- **Documentation Viewer**
	- All the documentation needs to be viewable from inside the app (for offline / air-gapped installs, etc.).
- **Quality of Life Features**
	- There are a bunch of smaller "quality-of-life" features I want to implement before release.
- **Unit Tests**
	- In progress!  About 50% complete.

## Nice to Have

The following features are on the "nice-to-have" list, and aren't blockers for release, but are on the top of the post-release TODO list:

- **Configuration Editor**
	- It would be really nice to be able to edit the configuration from inside the UI.
- **Log Viewer**
	- It would be extremely helpful to be able to view and search logs from inside the UI.
	- For example Docker deployments, where the logs aren't immediately accessible.
- **More Marketplace Plugins**
	- I would love to build out the marketplace catalog.

## Future Roadmap

Some fun things coming up in 2026 right after the v1 release:

- Web-based Terminal to "SSH" into any server
- API Tool (sort of like Postman) built into the UI 
- Web Push Notifications (like, real ones)
- API Key rate limiting
- Compliance (SOC2 / HIPAA)
- Calendar view of upcoming events
- Timeline view of workflow jobs
- Full remote CLI
- Lots more!
