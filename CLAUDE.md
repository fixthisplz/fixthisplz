# CLAUDE.md — FixThisPlz

This repository is developed with the help of an AI assistant (Claude Code). This file gives the assistant the project context and rules; it is public so anyone can see how the project is built.

## What this is

FixThisPlz lets people report what they see on a website and follows the fix back to them: **capture → triage → fix → tell the reporter**. A stack-agnostic widget, a self-hosted hub with a small built-in tracker, and adapters and integrations around them. See the [README](README.md) for the vision, architecture and roadmap.

## Rules

1. **Catalogue first.** Before writing any code, read [`CATALOGUE.md`](CATALOGUE.md) and the catalogue of the package you work in, then grep. Reuse what exists. A new reusable element gets its catalogue entry in the same commit.
2. **Simple code, no dead code.** Smallest solution that works; remove what becomes unused; no defensive code around deliberate user actions.
3. **Tests first**: failing test → code → passing test. Everything passes before a commit. Visible features are also checked in a real browser.
4. **Languages**: code, comments, commits and documentation in English.
5. **Commits only when the maintainer asks**, in Conventional Commits format.
6. **Scope**: stay within the non-goals of the README (no sprints, custom fields, SaaS or error monitoring).

## Local instructions

Machine-specific instructions (paths, test environments, project tracking) go in `CLAUDE.local.md` and in git-excluded files under `.claude/`; neither is versioned.
