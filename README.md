# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** Ahsan Gulzar
**Reference:** MATACS-TH-2026-02
**Time actually spent:** 3 hours

## Summary

This repository contains my design for an AI-assisted proposal triage tool. AI is used only to read proposals and extract facts with evidence quotes. Plain code handles mandatory pass/fail checks, scoring, ranking and logging. A human reviews, can override, and makes the final award decision.

## Files

- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [PROMPTS.md](PROMPTS.md): extraction prompt and reason-writing prompt
- [ranking.json](ranking.json): expected ranking of proposals A to E
- [diagram/flow.png](diagram/flow.png): process flow from upload to ranked output, showing AI steps, rule-based steps and human checkpoints

## Sample set result (short)

- A: ranked 1
- E: ranked 2
- B: ineligible (mandatory licence check fails)
- C: flagged (hidden instruction aimed at automated reviewers)
- D: flagged (no completion time, so timeline cannot be scored)

## How I used AI tools

I used Claude to discuss the design, draft the prompts and write-up structure, and check my reasoning. I wrote and reviewed the final files myself.

**Where I disagreed with or corrected the AI:**
Claude’s first ranking used the price of proposal D as the baseline for the price score. D is flagged and not ranked, so I changed the baseline to the cheapest ranked proposal (A). This changed the scores slightly (A 98.8, E 92.4) but not the order.

## Assumptions

- Proposals are short, so full text is sent to the model without a vector database.
- Price and timeline scores are relative to the other bids in the batch, so the batch is logged.
- No prototype is included. This is a design-only submission.