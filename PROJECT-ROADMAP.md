# Ship Something You Use

A build plan spanning full-stack/cloud, AI, and data engineering.
Budget: 5–10 hrs/week. First usable thing: end of week 1. Product line: week 8.

## Why one project, not five

The repos already show breadth — an applied-AI music recommender, forks of
vLLM and Search-R1, hackathon projects, a deployed Pages site. Breadth isn't
the gap. The gap is depth under load: a system kept running long enough that
it broke, cost money, got slow, and needed a second user.

Any application that ingests messy real-world data, reasons over it with a
model, and serves it to a human is simultaneously a data project, an AI
project, and a full-stack cloud project. Build one and you cover all three
tracks with one deploy pipeline to maintain — which matters a lot at 5–10
hours a week.

---

## Flagship: the application tracker that fills itself in

You apply to internships, scholarships, hackathons and programs. Today that
lives in a spreadsheet you update when you remember, a folder of tabs, and an
inbox where confirmations go to die. Deadlines get missed for filing reasons,
not merit reasons.

Paste a link → it extracts role, org, deadline and requirements into a row.
Connect Gmail → confirmations, rejections and interview invites move the row's
status without you touching it. Scrapers on the boards you care about start
finding things before you do. Sunday night it emails what's due and what's
gone quiet.

Stack: Python + FastAPI · Postgres · LLM structured extraction · Gmail API ·
scheduled workers · Fly.io or Render · GitHub Actions.

### Stage 0 — Week 1 — One script, one table  *(usable)*

A Python CLI and a SQLite file. `track add <url>` fetches the page, sends it
to a model with a strict JSON schema, writes one row: org, role, deadline,
link, requirements, status. `track due` prints the next 14 days. No web, no
framework, no deploy.

- **Teaches:** schema-constrained model output — the difference between
  parsing prose and getting back data you can trust. Your spreadsheet is dead
  by Friday, which is the point.
- **Hard thing:** valid, complete JSON on ten different sites, including the
  one behind a login wall and the one that's a PDF.

### Stage 1 — Weeks 2–3 — It exists on the internet  *(usable)*

Wrap the same functions in FastAPI, move SQLite to Postgres, add one
server-rendered page listing rows by deadline with a status dropdown. Deploy
it behind a real URL and open it on your phone.

- **Teaches:** migrations, secrets and environment config, container builds,
  connection pooling, reading logs you can't `print` your way through. Deploy
  on the *second day* of this stage, not the last — the deployment pain is the
  curriculum, not the obstacle.
- **Hard thing:** green local test to green production request with zero
  hardcoded values.

### Stage 2 — Weeks 4–5 — The inbox reads itself  *(usable)*

Gmail ingestion with incremental sync. Each message gets classified —
confirmation, rejection, interview invite, irrelevant — and advances the
matching row's status. Before writing the classifier, hand-label 60 of your
own real emails and freeze them as a test set.

- **Teaches:** OAuth done properly, sync by history cursor instead of
  re-reading the mailbox, and **evaluation**. Measure accuracy on those 60
  before and after every prompt change, and refuse to ship a change that drops
  it. This single habit separates "used an LLM API" from "can ship an LLM
  feature," and almost no student project has it.
- **Hard thing:** rejections that never say "unfortunately," and matching an
  email to the right row when the org name is spelled three ways.

### Stage 3 — Weeks 6–7 — It goes looking  *(usable)*

Scheduled scrapers for two or three boards you actually check. New postings
deduped against everything you have, then ranked against your profile. A
Sunday digest: due this week, gone quiet for 21 days, new and worth a look.

- **Teaches:** idempotent jobs, retries with backoff, a worker process
  separate from the web process, entity resolution (deciding "Mastercard
  Foundation" and "The MC Foundation" are one org), and being a polite
  scraper — robots.txt, rate limits, caching so you don't refetch what hasn't
  changed.
- **Hard thing:** a job that can run twice and leave the database identical.

### Stage 4 — Weeks 8–10 — Someone else can use it  *(product)*

Accounts with strict per-user data isolation, rate limits, structured logging,
an error tracker, and a dashboard showing tokens and dollars spent per user
per day. Hand it to five classmates and watch what they do that you never
would.

- **Teaches:** multi-tenancy and the security posture that comes with holding
  other people's mail-derived data. Unit economics — what one user costs per
  month, and whether that number survives a hundred of them.
- **Hard thing:** the query that can't accidentally return another user's
  rows, and the test that proves it.

---

## Two alternates with the same shape

Both span all three tracks and stay useful half-finished. Pick on which daily
annoyance is loudest.

**Ask my own material** *(retrieval · heaviest on AI)* — everything you've
read for a course behind one search box that answers in sentences and cites
the page. The twist that makes it teach: build the question set *first*. Pull
40 real questions from past problem sets and exams, mark which document
answers each, and measure retrieval recall before writing a single clever
chunking rule. Then earn every improvement against that number. Most RAG
projects are a demo; this one is an experiment.

**A warehouse of one** *(pipelines · heaviest on data)* — GitHub activity,
calendar, bank statement exports and course deadlines, landed raw in object
storage, modeled into clean tables, served as one dashboard showing where your
weeks actually go. DuckDB and Parquet on cheap storage, modeled with dbt,
orchestrated by a scheduler: the entire modern data stack for near-zero cost,
on data you care enough about to notice when it's wrong — the only way anyone
learns data quality. Lower product ceiling, highest density of employable
skills per hour.

---

## Warm-ups (a weekend each)

- **Give this site a pipeline.** `my-first-cloud-site` is two files and a
  deploy. Add GitHub Actions: build, link-check, a Lighthouse budget that
  fails the run, preview deploys on PRs, publish on merge. More real CI/CD in
  two evenings than any tutorial, and every later project reuses the workflow.
- **Standup generator.** A CLI that reads the week's commits across your repos
  and writes the summary you'd otherwise type. Forces you to handle an API's
  pagination and rate limits.
- **Grow the birthday wisher up.** Turn `birthday-wisher` into a general
  scheduled reminder service with retries, a dead-letter list, and a record of
  what it already sent — Stage 3's job infrastructure at a tenth the scope.
- **One real upstream fix.** You already forked vLLM. Land one merged PR —
  docs gap, flaky test, good-first-issue. Reading a large codebase you didn't
  write, and defending a change to strangers, is a skill no solo project
  teaches.

---

## House rules

- **Be your own first user.** If you wouldn't open it on an ordinary Tuesday,
  it's a demo, and demos don't generate the problems worth solving.
- **Deploy in week one.** Secrets, builds, logs, DNS and cold starts only show
  up in production.
- **One hard thing per stage.** Everything else in that stage should be boring
  on purpose.
- **Write the check before the clever part.** The eval set, the test, the
  assertion — otherwise you can't tell improvement from noise, and with models
  you definitely can't.
- **Keep a decisions log.** Three lines per choice in `DECISIONS.md`: what you
  picked, what you rejected, why. It's what you'll be asked about in
  interviews.
- **Watch the meter.** Log tokens, latency and dollars from the first model
  call. Every AI feature has a bill.
- **Stop at good enough** on anything that isn't this stage's hard thing.

## Not worth the hours

- Another to-do app, weather app, or clone of something you'd never open twice
- Tutorial chains — following along produces a copy of someone else's understanding
- Any project with no user, including you
- "I'll add tests later" — later is when you can no longer tell whether it still works
- Swapping frameworks mid-project because the new one is nicer

---

At 5–10 hours a week, Stage 0 lands in about seven days and the product line
at Stage 4 is roughly ten weeks out. That pace is fine. What decides whether
this works isn't the schedule — it's whether you're still opening the tool in
month three.
