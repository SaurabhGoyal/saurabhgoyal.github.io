---
layout: post
title:  "[Dump] May Archive"
subtitle: Archival of things learned in May, 2020.
date:   2020-05-25 21:26:07 +0530
categories: info, design
tags: info, design
---

- **Controllership** - A way to ensure correction of processes. Two ways-
  - Detective (Cure) - Finding incorrect/unexpected state in the system after they have occcuured, This includes process around how to handle such cases and what is their resolution. Ex.- Situation - A customer billed incorrectly for 100$, Resolution - Issue a 100$ credit-note to close this.
  - Preventive - Stopping incorrect/unexpected states in the system even before they can happen. Ex.- No unauthorized customer can place an order at all.


- **Two-Phased Commit (2PC)** in distributed systems - 
  - Context -
    - Fault-Tolerance is making sure a system is not faulty even if the componets in it are faulty.
    - There can be multiple ways - Ignore, fail-fast (report to controller and end), fail-safe (fallback to acceptable option), mask (override error and proceed).
    - Two areas for fault-tolerance - Safety (Bad things shouldn't happen, ex.- banking) and Liveness (Good things must happen eventually, ex.- social-networking). Different systems target a different balance of these two.
  - Problem - General purpose, distributed agreement on some action, with failures. (Ex.- tranfer of x from A to B)
  - Solution - A correct atomic protocol with **write-ahead non-volatile logging**.
    - Any server (A, B or transaction-cordinator) can go down at any point of time.
    - Final commit happens only after everyone has agreed for commit. Agreements are stored in disks before communicating to others.
    - Timeouts are the way to ensure something not to commit in case of failures.
    - **Blocking happens during timeout.**

- **Multi-tenancy (Platform) tenets** -

References
---
- Two-Phase Commit - cs.princeton.edu/courses/archive/fall16/cos418/docs/L6-2pc.pdf
- 