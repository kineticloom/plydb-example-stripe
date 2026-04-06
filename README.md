# Stripe AI Analytics Starter — AI-Powered Subscription Intelligence

Accompanying [blog post](https://www.plydb.com/blog/plydb-example-stripe/).

--

Ask questions about your Stripe account in plain English — MRR, churn, at-risk
subscribers, expansion opportunities — answered instantly by an AI agent with
zero ETL, no SQL required.

> Which active subscribers have a failed payment or cancel_at_period_end set —
> and what's the total MRR at risk?

> Which customers on our entry-level plan have the longest tenure and best
> payment history — who should we pitch an upgrade to?

> How do invoice payment failure rates compare across subscription plans?

> What percentage of new subscribers cancel within their first billing cycle?

A starter project for SaaS founders, RevOps, and billing teams who want
AI-powered subscription analytics without standing up a data warehouse. Use it
as-is or adapt it as the Stripe layer in a broader analytics setup.

Under the hood: [dlt](https://dlthub.com/) loads your Stripe data into a local
DuckDB file, [PlyDB](https://www.plydb.com/) gives
[Claude Code](https://claude.ai/code) (or any AI agent) a SQL gateway into that
data, and a semantic overlay teaches the agent your data model so it asks better
questions and writes more accurate queries — your data stays local, zero ETL, no
cloud.

---

Demo with Claude

https://github.com/user-attachments/assets/af87f40a-83a8-4b96-96fb-a1b9d32729b3

---

## Workflow

1. [Install prerequisites](#step-1--install-prerequisites)
2. [Load Stripe data](#step-2--load-stripe-data)
3. [Configure PlyDB](#step-3--configure-plydb)
4. [Start analyzing](#step-4--start-analyzing)

---

## Step 1 — Install prerequisites

### PlyDB

PlyDB is the database gateway that gives your AI agent unified SQL access to
local data. Your agent translates your questions into SQL; PlyDB executes them.

**New to PlyDB?** The [PlyDB quickstart](https://www.plydb.com/docs/quickstart/)
walks through installation, config, and your first queries end-to-end.

### Python environment

The dlt data loading pipeline requires Python 3.8+. Create a virtual environment
and install dependencies:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements-lock.txt
```

> `requirements-lock.txt` pins every dependency to known-good versions. Use
> `requirements.txt` if you prefer to resolve the latest compatible versions
> yourself.

---

## Step 2 — Load Stripe data

### Get your Stripe API key

1. Log in to your [Stripe Dashboard](https://dashboard.stripe.com/)
2. Navigate to **Developers → API keys**
3. Copy your **Secret key** (starts with `sk_live_` for production or `sk_test_`
   for test mode)

### Add your API key

Create `.dlt/secrets.toml` and add the following, replacing the placeholder with
your key:

```toml
[sources.stripe_analytics]
stripe_secret_key = "sk_live_your_key_here"
```

> This file is excluded from version control. Never commit your secret key.

### Run the pipeline

```bash
python stripe_analytics_pipeline.py
```

This loads data from your Stripe account into `data/stripe_analytics.db` (a
local DuckDB file). The pipeline fetches the following endpoints:

| Endpoint             | Description                                                    |
| -------------------- | -------------------------------------------------------------- |
| `Subscription`       | Recurring subscription records with status and billing details |
| `Customer`           | Customer profiles including email and currency                 |
| `Product`            | Products and plans offered in your Stripe account              |
| `Price`              | Pricing configurations (recurring and one-time)                |
| `Invoice`            | Invoices with amounts, status, and billing reason              |
| `BalanceTransaction` | Full ledger of charges, payouts, refunds, and fees             |
| `Event`              | Audit log of all activity in your Stripe account               |

Re-running the pipeline refreshes all data.

---

## Step 3 — Configure PlyDB

`plydb-config.json` is pre-configured to point PlyDB at the loaded DuckDB file,
with `plydb-overlay.yaml` providing semantic context about the Stripe data
model. No changes needed — open your agent in this directory and it will pick up
the config automatically.

---

## Step 4 — Start analyzing

Open Claude Code (or any PlyDB-compatible agent) in this directory and start
asking questions. The agent will translate your questions into SQL, run them
against your local Stripe data via PlyDB, and return results.

### Sample prompts

**At-risk subscribers:** Which active subscribers are most likely to churn in
the next 30 days? Flag anyone with a recent failed payment attempt, a canceled
trial, or a `cancel_at_period_end` flag set — and list their email and plan so I
can prioritize outreach.

**Expansion revenue opportunity:** Which customers on the Starter plan have been
subscribers the longest and are paying on time every cycle? Rank them by tenure
— these are the best candidates for an upgrade offer to a higher tier.

**Pricing pressure signals:** Compare average subscriber tenure between plan
tiers. If lower-tier customers churn faster, that may indicate a pricing or
value-gap problem worth investigating before the next pricing review.

**Revenue at risk this month:** What is the total MRR from subscriptions whose
current period ends in the next 30 days where `cancel_at_period_end` is true?
This is the confirmed revenue walkoff unless those customers are saved.

**Churn analysis:** What is the overall churn rate, and does it vary by plan?
What is the most common cancellation reason? Are customers canceling early in
their lifecycle or after extended tenure?

**Growth cohorts:** Group new subscribers by the month they started. For each
cohort, how many are still active after 3, 6, and 12 months? Which months had
the best retention — and what changed around the months that didn't?

**Payout reconciliation:** What is the net amount paid out to the bank after
Stripe fees over the last three months? How do gross charges, refunds, and fees
break down month by month?

---

## Going further

This repo is intentionally minimal — a working starting point, not a production
system. Here are a few natural directions to take it:

**Cross-domain analysis:** PlyDB can query across multiple data sources in a
single SQL statement. Add your marketing data
([Google Analytics](https://dlthub.com/docs/dlt-ecosystem/verified-sources/google_analytics),
[Google Ads](https://dlthub.com/docs/dlt-ecosystem/verified-sources/google_ads),
[Facebook Ads](https://dlthub.com/docs/dlt-ecosystem/verified-sources/facebook_ads)),
sales data
([HubSpot](https://dlthub.com/docs/dlt-ecosystem/verified-sources/hubspot),
[Salesforce](https://dlthub.com/docs/dlt-ecosystem/verified-sources/salesforce)),
customer support
([Zendesk](https://dlthub.com/docs/dlt-ecosystem/verified-sources/zendesk),
[Freshdesk](https://dlthub.com/docs/dlt-ecosystem/verified-sources/freshdesk)),
or infrastructure costs alongside Stripe to answer questions that span the full
business — CAC vs LTV, pipeline-to-revenue conversion, or margin by customer
segment.

**Cloud storage:** The dlt pipeline can write to S3, BigQuery, Snowflake, or any
other [dlt destination](https://dlthub.com/docs/dlt-ecosystem/destinations/)
instead of a local DuckDB file. Swap the destination in
`stripe_analytics_pipeline.py` and update `plydb-config.json` to point at the
new source.

**Scheduled loads:** Run the pipeline on a schedule (cron, Airflow, GitHub
Actions) to keep the data fresh. PlyDB and your agent can then answer questions
against up-to-date data without any manual refresh.

**Proactive monitoring instead of static dashboards:** Most revenue dashboards
go stale the moment they're built — they answer the questions you thought to ask
when you built them, not the ones that matter today. Schedule your agent to run
nightly and proactively surface what changed: a churn spike, a cohort
underperforming its peers, a cluster of failed payments before they become
cancellations. Instead of checking a dashboard, you get a morning briefing with
the anomalies already flagged and a recommended action for each. Agents that
support scheduled runs (such as [Claude Code](https://claude.ai/code)) can
automate this end-to-end.

**From insight to action:** An AI agent with live data access doesn't just
answer questions — it can act on the answers. After identifying at-risk
subscribers, ask it to draft personalized outreach emails ranked by revenue
impact. After spotting an upgrade opportunity, ask it to prepare a one-pager for
the customer. The analysis and the response live in the same conversation,
collapsing the gap between insight and decision that kills most analytics
workflows.

**Richer semantic context:** The `plydb-overlay.yaml` file is a starting point.
After any analysis session, ask your agent to update it with what it learned —
business rules, enum meanings, metric definitions. These compound over time and
make future sessions more accurate.

---

## Data source

| Source                                    | Description                                                        |
| ----------------------------------------- | ------------------------------------------------------------------ |
| [Stripe API](https://stripe.com/docs/api) | Subscriptions, customers, invoices, balance transactions, and more |

---

## Reference

- [Stripe API reference](https://stripe.com/docs/api) — full reference for all
  Stripe API objects and fields
- [dlt Stripe verified source](https://dlthub.com/docs/dlt-ecosystem/verified-sources/stripe)
  — documentation for the dlt pipeline used to load the data
- [PlyDB documentation](https://www.plydb.com/docs/) — full PlyDB reference
