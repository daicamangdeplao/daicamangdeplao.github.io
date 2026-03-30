---
layout: post
title: "App Folder-Structure Layout"
date: 2026-03-16
categories: [backend_engineering]
---

# App Folder-Structure Layout

## Introduction

Every running application produces and consumes different kinds of data — configuration it reads, logs it writes, temporary files it generates. Without a deliberate layout strategy, these concerns get mixed together, making the system harder to scale, debug, and deploy.

A good folder-structure layout should answer three key questions:

* Where do logs go?
* Where does persistent data live?
* Where is the configuration read from?

## Concerns in Container-based Execution Environment

> Separate code from state depending on environment constraints

| Category | Example                 | Placement Strategy          | Implementation                  |
|----------|-------------------------|-----------------------------|---------------------------------|
| Code     | binaries, jars          | versioned, read-only        | ``/app``                        |
| Config   | `settings.yaml`, `.env` | externalized                | env vars or mounted ``/config`` |
| Logs     | runtime logs            | centralized or ``/var/log`` | stdout (collected by system)    |
| Data     | DB files, uploads       | Ephemeral and Persistent    | external DB + object storage    |
| Temp     | cache, scratch          | Ephemeral                   | ``/tmp``                        |

## Checklist for Genesis

* Where will it run?
* What must survive restarts (persistent vs ephemeral)?
* How will it be deployed? (CI\CD, rolling updates)
* What must be shared across instances?
* What needs isolation? (logs, config, secret)
* What must be backed up?
