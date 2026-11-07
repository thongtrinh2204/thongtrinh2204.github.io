---
layout: post
title: "Omnara: A Self-Hostable, Postgres-Backed Alternative to Claude Managed Agents"
date: 2026-11-07 04:17:15 +0000
tags: [ai, agents, opensource, selfhosted, postgres, golang, devtools, llm]
description: "Omnara is an open-source (Apache 2.0) API for running durable agents. Each agent's state lives in Postgres, so agents survive restarts and use no compute while idle. It's a promising option if you wan"
---

TL;DR: Omnara is an open-source (Apache 2.0) API for running durable agents. Each agent's state lives in Postgres, so agents survive restarts and use no compute while idle. It's a promising option if you want hosted-agent features on your own server. Expect early-stage rough edges, and expect to run Postgres, an API server and workers yourself.

*This is a research-based review. I haven't installed or run Omnara. Everything below comes from the README, the docs, the Show HN post and community discussion.*

## What it is

Omnara, from Omnara AI, Inc. (founded by Kartik Sarangmath and Ishaan Sehgal, YC S25), is an open-source API for running durable agents. You create an agent with an API call. Omnara runs the agent loop, gives the agent sandboxes for running commands, and handles secrets, permissions and RBAC. You can message agents and stream their events over the API.

The team first built a remote-control phone app for Claude Code and Codex. According to the Show HN post, that design had limits: it needed a relay, it stopped when the laptop closed, and moving to a cloud sandbox meant syncing code and agent state between machines. So they changed the model. The agent runs in the cloud, any client talks to it through an API, and it can still use your laptop as one of its machines.

The pitch is an open-source counterpart to Anthropic's Claude Managed Agents and the OpenAI Agents API. You can run it on your own servers with your own models. The author describes it as a harness server like the opencode or codex servers, but durable and scalable.

Repo: [github.com/omnara-ai/omnara](https://github.com/omnara-ai/omnara). Homepage: [omnara.com](https://omnara.com). The project is written mainly in Go and has about 3,300 stars as of November 2026. The latest component release I found is `v0.2.11` (October 2026).

## Key features

**Append-only event log in Postgres.** Most harnesses, such as Claude Code, Codex, OpenCode and Goose, run the loop in one long-lived process and keep a JSONL or SQLite log. Omnara stores inputs, model outputs and tool calls as typed rows in their own tables.

**Stateless workers.** A worker runs one iteration of the loop, either a model call or the tool calls it produced. It then releases the agent and puts the next iteration on a Postgres queue. Any free worker can pick it up, so consecutive iterations can run on different machines.

**Atomic commits.** Per the Show HN post, each iteration's result commits in one transaction. Model output, tool calls, usage and the next wakeup are saved together or not at all. If a worker dies mid-iteration, nothing commits, the lease expires and another worker retries.

**Cheap idle agents.** Agents waiting on a user, a tool or a retry don't hold a worker. The author says one worker can multiplex thousands of agents.

**Outbound-only machines.** Any macOS or Linux machine can run a small daemon, `omnarad`, that connects out to Omnara, so no inbound ports are needed. An agent can have several machines connected. The author's example is a laptop plus a cloud sandbox on one agent. When the laptop goes offline, the agent pulls the latest changes (pushed to GitHub) into the sandbox and continues there.

**Harness features.** The docs list tool execution, sandboxes, MCP server support, subagents and context compaction. The Show HN post also mentions file and shell tools, permission prompts, skills and memory.

## Getting started

The quick start uses Docker Compose, as described in the repo:

```bash
git clone https://github.com/omnara-ai/omnara.git
cd omnara
docker compose -f compose.yaml --profile app up -d
```

The dashboard is at `http://localhost:8000`. The email verification link is printed in `docker compose logs api`.

Then connect the CLI and install the SDK:

```bash
npx omnara config --api-url http://localhost:8000/api/v1 --issuer-url http://localhost:8000
npx omnara login
npm install @omnara/sdk
```

The SDK is on npm, which suits TypeScript projects. Check the docs for the current API surface, because the project is moving fast.

## Pros and cons

**Pros**

- **Durability by design.** State sits in Postgres, and each iteration commits atomically, so a worker that dies mid-iteration commits nothing (per the Show HN post).
- **Efficient when idle.** Waiting agents don't tie up a worker, and the project says idle agents cost nothing.
- **Hybrid execution.** The outbound `omnarad` daemon avoids firewall and port-forwarding work, and lets an agent move between a laptop and a sandbox.
- **No lock-in.** Apache 2.0, self-hostable on a VPS or cloud instance, with your own model provider.
- **Simple pricing.** Self-hosting is free apart from your own compute and LLM keys. Omnara Cloud has no seat or platform fees and bills for usage: managed model tokens plus active sandbox runtime and memory hours. Bringing your own keys and machines is free. Enterprise pricing is custom.

**Cons**

- **Operational overhead.** You maintain Postgres, an API server and background workers. Hacker News commenters noted this is heavier than a single-binary local agent CLI.
- **Early-stage maturity.** GitHub issues point to frequent schema changes and some rough edges in CLI configuration and parsing. Read the release notes before upgrading.
- **Backend focus.** It's developer infrastructure, a harness API. It isn't a polished desktop assistant or end-user GUI.

My research turned up no red flags, such as licensing problems or abandoned maintenance.

## Who it's for

Omnara fits developers building products on top of agents who want durability without sending everything to a closed vendor. Examples are teams running long-lived coding or research agents, and people who want an agent that keeps working after a laptop closes.

I build bots and automation, and I run my own services on a VPS under systemd, so the design speaks to me. Long-running background workers fail in boring ways: restarts, half-finished jobs, workers that die at the wrong moment. A lease-and-retry model with transactional commits targets that kind of failure. I haven't tested whether Omnara delivers on it. If I try it, I'd test the worker-kill case first: stop a worker mid-iteration and check that another one picks the work up cleanly.

It's probably not for you if you only want a local coding agent in your terminal. A tool like OpenCode or Goose is simpler for that.

## Alternatives

- **Claude Managed Agents:** Anthropic's hosted, closed-source infrastructure for deploying autonomous Claude agents.
- **OpenAI Agents API:** OpenAI's managed platform for stateful agent workflows with hosted tools and sandboxes.
- **OpenCode / Goose:** open-source developer harnesses that run the loop inside a single local process. They're lighter to run.
- **E2B:** a dedicated cloud sandbox platform for running agent code in isolated environments. It covers the sandbox, not the durable loop.

## Verdict

Omnara's core idea is a durable agent loop built from a Postgres event log, a queue and stateless workers. It applies a familiar backend pattern to agents and makes self-hosting a first-class option. The outbound-only `omnarad` daemon is a practical touch for anyone who has fought NAT and firewalls.

The trade-offs are plain. You run more infrastructure than with a local CLI, and the project is young enough that schema and CLI changes are likely. For a side project or a pilot, I'd start with the Docker Compose setup on a small VPS and pin versions. For production, I'd wait for more releases and read the open issues first.

If you want managed-agent features without a closed platform, Omnara is worth a look.

Tags: #ai #agents #opensource #selfhosted #postgres #golang #devtools #llm
