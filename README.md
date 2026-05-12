# agent-blueprint

A framework-agnostic checklist for building reliable agentic systems.

## What it is

`AGENT-BLUEPRINT.md` is a structured reference for evaluating and designing agentic AI systems. It organizes the core problems into four areas — **Control**, **Scale**, **Trust**, and **Context** — each with principles, checklists, and notes blocks for per-project decisions.

Derived from the Claude Certified Architect – Foundations curriculum. Abstracted to apply across LangChain, AutoGen, CrewAI, DSPy, OpenAI Assistants, and raw API implementations.

## How to use it

Copy the **Project Checklist** block at the bottom of `AGENT-BLUEPRINT.md` into your project's documentation. Work through the 11 items, filling in the Notes blocks with your architectural decisions.

## The Four Problems

| # | Problem | Question |
|---|---------|----------|
| 1 | Control | Does the agent do the right thing? |
| 2 | Scale | Do multiple agents work reliably? |
| 3 | Trust | Can you verify the output? |
| 4 | Context | Are the other three constrained by context window limits? |
