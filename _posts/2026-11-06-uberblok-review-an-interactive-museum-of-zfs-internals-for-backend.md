---
layout: post
title: "Uberblok Review: An Interactive Museum of ZFS Internals for Backend Engineers"
date: 2026-11-06 17:45:53 +0000
tags: [zfs, storage, copyonwrite, opensource, backend, devops, filesystems, astro]
description: "Uberblok is a free, open-source site that explains how OpenZFS works through animated diagrams and guided \"museum\" stops. According to the project, it covers copy-on-write, transaction groups, RAID-Z,"
---

**TL;DR:** Uberblok is a free, open-source site that explains how OpenZFS works through animated diagrams and guided "museum" stops. According to the project, it covers copy-on-write, transaction groups, RAID-Z, the ARC and more, and it links its explanations to real OpenZFS source. This review is based on the project's docs and public discussion, not hands-on use.

## What it is

Uberblok is an interactive virtual museum for the internals of the OpenZFS file system. The author says they have worked in storage for a long time and built it as a homage to ZFS and as a way to understand it better themselves. According to the Show HN post, it was built with their agent swarm organization, Algotitan.

The [live museum](https://uberblok.storagebits.workers.dev/) and the [GitHub repo](https://github.com/algotitanhq/uberblok) are both public. The author also invites corrections from people who know ZFS internals better, which is a good sign for an educational project. The [Show HN thread](https://news.ycombinator.com/item?id=50180257) had 126 points and 39 comments at the time of the listing.

## Key features

According to the documentation, the museum has 14 stops:

- **The Hall:** an interactive 2D and 3D view of the uberblock ring.
- **A History Wing:** ZFS from Sun Microsystems to OpenZFS.
- **11 technical rooms:** these include copy-on-write, txgs, vdevs, RAID-Z, snapshots, the ARC, metaslabs, ZIL/SLOG, send/receive, and compression, deduplication and encryption.
- **An Engine Room:** a colophon describing how the site is built.

Other points from the docs:

- **Code mapping:** explanations link to OpenZFS release `zfs-2.4.4` (commit `71a9f9578616a90c3c14bb59629fb4d31bfd68d1`), so you can compare a diagram with the actual C source.
- **Automated validation:** the project uses Playwright and Lighthouse to test interactive figures and performance.
- **Licensing:** MIT for the site code, CC BY 4.0 for texts and illustrations, and CDDL for the referenced OpenZFS snippets.

## Getting started

You can open the hosted site directly. To run it locally, you need Node.js 22 (pinned in `.nvmrc`) and Git:

```bash
nvm use
npm ci
npm run build
npm run preview
```

To run the repository's test suites:

```bash
npm test
```

According to GitHub, the stack is mostly Astro (49.2%), TypeScript (26.2%) and JavaScript (23.2%).

## Why backend engineers might care

Copy-on-write shows up well beyond file systems, for example in databases, immutable data structures and snapshotting schemes. ZFS is a well-known, production-grade example. The core idea is that a write never overwrites live data, and a small root record (the uberblock) is updated to commit a new consistent tree. That idea is worth understanding, and an animated format is a natural way to make it concrete. Hacker News commenters reportedly praised how the site turns abstract topics such as ARC ghost lists and metaslab space maps into intuitive visual models.

## Pros and cons

**Pros**

- Visual explanations of dense subsystems, praised by the HN community.
- Anchored to real OpenZFS C snippets rather than loose approximations, per the Engine Room documentation.
- Free and open source, with no paid tiers.
- Automated end-to-end tests for the interactive figures.
- Clear licensing for both code and content.

**Cons**

- Purely educational. The documentation describes no diagnostic, administrative or pool-tuning functionality.
- A very young project: about 26 stars as of late 2026, no formal releases or tags, and a contributor base centered on the author and their AI swarm.
- Pinned to OpenZFS 2.4.4, so it needs manual updates as upstream changes.
- The author openly asks for corrections, so cross-check important details against the OpenZFS source.

## Who it's for

- Backend and infrastructure engineers who use ZFS but have never looked inside it.
- Storage and DevOps people who want to reason about snapshots, send/receive or caching behavior.
- Students and curious developers who learn better from animation than from long prose.
- Anyone who wants a model of how a copy-on-write system stays consistent.

It is not for you if you want an operations guide or tuning advice for a live pool.

## Alternatives

- **OpenZFS Documentation:** the official technical manuals and architecture reference, including subsystem tunables.
- **Ars Technica ZFS 101/102 (Jim Salter):** classic long-form written explainers on copy-on-write, vdevs and RAID-Z.
- **FreeBSD Handbook, ZFS chapter:** a practical reference that pairs concepts with administration commands.
- **The Secret Lives of Data:** a comparable interactive step-by-step visualizer, but focused on Raft consensus rather than file systems.

## Verdict

ZFS internals are usually explained in dense text or source code, so an animated, code-linked tour is a welcome alternative. Its newness, small footprint and fixed upstream version are worth keeping in mind. I can't speak to how polished it feels in use, since this review is based on the docs and community feedback. It costs nothing to open, though, and if you want to understand copy-on-write and ZFS, it looks like a good place to start. Pair it with the official OpenZFS docs when you need to verify details.

Tags: #zfs #storage #copyonwrite #opensource #backend #devops #filesystems #astro

