# Shotgun — an AI assistant that explains itself, built for used-car auction buyers

*I call shotgun!*

## The context

Manheim runs live online car auctions called "simulcast" — dozens of used cars roll across a video feed one after another, and buyers bid on them in real time, often watching several of these feeds ("lanes") at once. It's fast, it's high-stakes, and it doesn't pause for anyone.

This project came out of an internal Cox Automotive "hackathon" — a short, intense two-day build sprint (Aug 3–4, 2026) that ends in a live demo. Our team, Pod 11, went in wanting to help buyers with a very specific, very human problem: a buyer watching several live auction lanes at once gets roughly **30 seconds** to decide whether to spend $10,000–$30,000 on a car. Worse, the three screens they need for that decision — their "workbook" (a personal shortlist of the cars they're actually considering, pulled out of the hundreds up for auction that day), the page where they'd enter a bid, and the live auction feed itself — don't share any information. Every single car means manually cross-checking three different places against a countdown clock.

Before writing a line of code, we pulled together five existing buyer interviews and complaints that were already on record — real feedback that had been gathered previously, not something we went out and collected ourselves. The obvious fix — just have the computer place bids automatically — is the one that feedback pointed away from almost unanimously. Most of it described automated bidding as "lazy" and said it took the judgment out of a decision that's still partly a gut call. So this tool deliberately **never places a bid.** Instead, it reads every car in a buyer's workbook and rates how strongly they should consider bidding on it — High, Medium, or Low confidence — with a specific, checkable reason behind every rating. And critically, the buyer can question that reasoning at any time, the same way they'd ask a colleague "wait, why do you think that?"

The idea underneath all of it: people trust and act on a recommendation they can question and verify. They don't trust a black-box score that just tells them what to do.

## Watch it in action

*End-to-end walkthrough — no auction background required to follow along.*

[▶ Watch the demo](https://youtu.be/kraUHJAEAB0)

## A look at the tool

### The workbook

<img width="2523" height="1267" alt="shotgun_workbook" src="https://github.com/user-attachments/assets/5e8d2235-5855-4c7d-8fe9-8425a2c9904a" />

The workbook is the buyer's personal shortlist — pulled out of a shared pool of roughly 600 cars up for auction that day. Rather than trying to analyze all 600 cars (most of which a given buyer would never bid on anyway), the tool only rates the ones the buyer has actually added to their workbook. That keeps the analysis focused on what matters to that specific buyer, and fast enough to be useful in the moment.

### Loading screen

<img width="2521" height="1262" alt="shotgun_loading" src="https://github.com/user-attachments/assets/b9d27dd6-e6e0-43a7-b922-31e5e06d9f32" />

While the tool works through a buyer's workbook, it shows a small animated indicator — a car icon moving along a road — that fills in as more cars finish being analyzed. Instead of a blank screen while you wait, you can see exactly how far through the list the analysis has gotten.

### The cart

<img width="2518" height="1270" alt="shotgun_cart" src="https://github.com/user-attachments/assets/a38a07ff-562a-49e2-b2c3-538aadc24dac" />

The cart is a personal planning space: for each car a buyer is interested in, they can set the price they'd actually be willing to pay, with a suggested starting price offered as a reference point. This is worth calling out explicitly — nothing staged in the cart is ever sent to the real auction. It's a private plan the buyer builds for themselves; a human still has to place every real bid by hand.

### Preferences

<img width="2521" height="1267" alt="shotgun_preferences" src="https://github.com/user-attachments/assets/390ef576-57c0-49a4-90af-dbda945c5331" />

A simple text box where a buyer describes, in their own words, what they're actually looking for — something like "cheap trucks under $14,000" or "I run a body shop, so I can take cars with heavier damage." The moment a buyer saves their preferences, every car in their workbook gets re-evaluated against what they specifically said they wanted, instead of some generic one-size-fits-all standard.

### Notes and "ask the agent"

<img width="2516" height="1257" alt="shotgun_note_ask" src="https://github.com/user-attachments/assets/22663a71-2dcc-4b56-9376-d9853e16b4b8" />

This screen shows two connected features. First, a buyer can leave a free-text note on any individual car — for example, "my body shop fixes this for $100" — and the tool factors that note directly into its rating of that car. Second, there's a chat-style box where the buyer can ask follow-up questions, like "why is this rated Low?" and get an answer built only from facts actually visible on that car's page — nothing invented, nothing pulled from outside what's shown. This is the heart of the whole project: a rating you're allowed to interrogate, not one you're expected to take on faith.

## Why this approach

The bet this project makes is simple: a tool that shows its reasoning — and lets you push back on it — earns trust that a tool which just hands you a number never will. Everything above was built around making that reasoning checkable against what's actually on screen.

---

## Technical details

This project was built on a company account during the hackathon, so the actual source code isn't something I have access to or can share here — this repo is a showcase, not the codebase. What follows is a description of how the project works and what it's built from, not setup or run instructions.

### How it's built

The project splits into four pieces, each with a single, deliberate job:

- **A rating engine** that takes one car's data and produces a confidence rating plus a reason. It's pure logic with no network calls of its own, which is what makes its output easy to test and reason about in isolation.
- **The web board itself** — a plain HTML/CSS/JavaScript frontend backed by a lightweight Python server, with no frontend framework and no build step.
- **A persistence layer** that stores which cars are in which dealer's workbook, their notes, and their cached ratings in a small database (AWS DynamoDB), so nothing is lost between sessions.
- **A model client** that calls Claude, Anthropic's AI model, through AWS Bedrock to generate the actual ratings and reasoning.

Every outside dependency — the AI model, the database, the live auction data feed — is isolated behind exactly one narrow connection point in the code. That separation is also what let the whole rating pipeline be fully tested (hundreds of automated tests) without needing any real AWS credentials or network access — the outside world is faked at those same narrow seams.

One test file is worth calling out on its own: a dedicated suite of tests exists purely to prove that a dealer's free-text note can't be used to hijack the AI model's instructions (a "prompt injection" attack) — the note is sanitized and fenced off before it ever reaches the model.
