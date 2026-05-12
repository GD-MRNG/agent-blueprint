# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A documentation-only repository. There are no build steps, test suites, or code to run. The primary artifact is `AGENTIC_FRAMEWORK.md` — a framework-agnostic checklist for building reliable agentic systems.

## Document Structure

`AGENTIC_FRAMEWORK.md` is organized around **four problems** that every agentic system must solve:

| # | Problem | Core Question |
|---|---------|---------------|
| 1 | **Control** | Does the agent do the right thing? |
| 2 | **Scale** | Do multiple agents work reliably? |
| 3 | **Trust** | Can you verify the output? |
| 4 | **Context** | Are the other three constrained by context window limits? |

Each section follows the same structure: a **Why it matters** principle, a **checklist** of items (written as `- [ ]` tasks), and a **Notes** block for project-specific decisions.

The document ends with a **Project Checklist** (copy-per-project template) that distills all 11 checklist items into a single block for use in actual projects.

## Working With This Document

The intended workflow:
1. Copy the Project Checklist block at the bottom of `AGENTIC_FRAMEWORK.md` into a new project's documentation.
2. Work through each of the 11 items, filling in the Notes blocks with project-specific architectural decisions.
3. Use the Skill Connection Map (bottom of the framework) to understand how the four problem areas depend on each other.

When editing the framework itself, preserve the `- [ ]` checkbox format — these are meant to be copied and used as live checklists in other projects. Notes blocks use HTML comment syntax (`<!-- -->`) intentionally so they render as invisible placeholders when copied into markdown.
