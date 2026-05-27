---
name: follow-up-nurture
description: Send context-aware follow-up emails to pending leads. Use when nudging leads, following up on sales conversations, or re-engaging prospects. Pulls email history, researches context, writes in my tone.
описание_ru: Пишет контекстно-осознанные follow-up-письма по «зависшим» лидам — подтягивает историю переписки, ресёрчит контекст и сохраняет твой тон голоса.
когда_использовать_ru: Когда лид не отвечает и нужно мягко напомнить о себе, продолжить продажу или реактивировать старого клиента без шаблонных писем.
allowed-tools: Read, Grep, Glob, Bash
---

# Follow-Up Nurture — Context-Aware Lead Nudge

## Goal
Go through every lead marked "pending" in the database, pull all prior email communications, research anything mentioned that you don't know about, and send a personalized follow-up email in Anton's voice. This isn't a cute demo — it makes money.

## Inputs
- **Lead database**: `data/leads_database.json` — contains leads with status, email history, notes
- **Tone**: Direct, businesslike, no fluff. Never salesy. Never use exclamation marks. Anton is a technical specialist talking to other professionals — short, specific, no basics. Russian or English depending on the thread language.
- **Mode**: Always present output as final emails being sent (not "drafts"). Use `--generate` then `--review` to produce and display them.

## Process

### Step 1: Load Pending Leads
```bash
python3 "C:/Users/gnilo/Desktop/Free Claude Code Skills/execution/follow_up_nurture.py" --list-pending
```
This reads `data/leads_database.json` and shows all leads with `status: "pending"`.

### Step 2: For Each Pending Lead
1. **Pull email history** — read the `email_history` array for that lead
2. **Identify context gaps** — if the lead mentioned something specific (a project, a tool, a problem, a competitor), and you don't have enough context to write intelligently about it, do a quick web search
3. **Write follow-up** — write a short, context-aware email that:
   - References something specific from your last conversation
   - Adds value (a relevant insight, resource, or observation)
   - Has a soft CTA (no "schedule a call" unless they asked)
   - Is 3-5 sentences max

### Step 3: Generate Emails
```bash
python3 "C:/Users/gnilo/Desktop/Free Claude Code Skills/execution/follow_up_nurture.py" --generate
```
This processes all pending leads, generates emails, and saves to `data/followup_emails.json`.

### Step 4: Review & Send
```bash
# Review emails
python3 "C:/Users/gnilo/Desktop/Free Claude Code Skills/execution/follow_up_nurture.py" --review

# Send emails
python3 "C:/Users/gnilo/Desktop/Free Claude Code Skills/execution/follow_up_nurture.py" --send
```

## Anton's Tone Guide

Keep it short, direct, no fluff. These are check-ins, not pitches. Match the language and tone of the original campaign thread (RU/EN).

### Templates (pick one based on prior thread tone)

**Default — короткое деловое напоминание (RU):**
```
{Имя}, возвращаюсь к {тема}. Готов ответить на вопросы или подвинуться по срокам.

— Антон
```

**English equivalent:**
```
{Name}, following up on {topic}. Happy to answer questions or adjust timing.

— Anton
```

**Если контекст более тёплый / клиент уже работал с тобой (RU):**
```
{Имя}, по {тема} — двигаемся? Если есть вопросы по {конкретная деталь из переписки}, отвечу сегодня.

— Антон
```

**If checking in on a general relationship (no specific topic, EN):**
```
{Name}, checking in. Where are we on {topic}? Let me know if anything's blocking.

— Anton
```

**Following up on a sent deliverable (proposal, contract, deck — RU):**
```
{Имя}, на той неделе скинул {что}. Если нужны правки или пояснения — скажи, поправлю.

— Антон
```

### Tone rules
- No "I hope this email finds you well" / «Надеюсь, у вас всё хорошо» — never.
- No exclamation marks.
- Reference the specific thing you discussed, not a generic "our product" / «наш продукт».
- 2-3 sentences max. These are nudges, not essays.
- Match the language (RU/EN) and formality of whatever thread you're replying to.
- Anton's voice: technical, no basics-explaining, no salesy filler. Treat the recipient as a peer.

## Scheduling
This Skill can be run on a schedule (daily, weekly) to automatically nurture your pipeline. The script supports `--cron` mode for unattended operation.

## Output
- `data/followup_emails.json` — generated emails with lead context
- Sent emails logged back to `data/leads_database.json` under each lead's `email_history`

## Successive Run Behavior
This skill is designed to run daily. Each follow-up must be different from the last:
- The prompt sees the full email history including prior follow-ups
- If the last outbound email was already a nudge, the model picks a different template and phrasing
- This prevents sending "circling back on the proposal" 7 days in a row

## Edge Cases
- **No prior emails**: Skip the lead, flag as "needs intro email instead"
- **Lead replied recently (< 48h)**: Skip, they're already engaged
- **Lead marked "closed" or "lost"**: Skip entirely
- **Research fails**: Write follow-up without the research, note the gap

## First-Run Setup

Before executing, check if the workspace has a `.gitignore` file. If it doesn't, assume the user is new to this skill. In that case:

1. Ask the user if this is their first time running this skill
2. If yes, walk them through how it works and what they need to configure/set up (API keys, env vars, dependencies, etc.)
3. Confirm the workspace `.env` has `ANTHROPIC_API_KEY` and the sender Gmail token is configured.
