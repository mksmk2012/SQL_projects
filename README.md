# Email & Account Analytics (BigQuery SQL)

A single BigQuery query that combines **account** and **email campaign** metrics into one dataset, adds per-country totals and rankings, and keeps only the top-10 countries.

## Goal

Answer questions such as:

- How many accounts are created per day, country and account type?
- How many emails are sent, opened and clicked through to a visit?
- Which countries lead by number of accounts and by number of sent emails?
- How do metrics differ by `send_interval`, verification status and unsubscribe status?

## Data source

BigQuery dataset `DA` (project `data-analytics-mate`).

| Table | Used for |
|---|---|
| `DA.account` | Account attributes: `id`, `send_interval`, `is_verified`, `is_unsubscribed` |
| `DA.account_session` | Link between accounts and sessions |
| `DA.session` | Session date |
| `DA.session_params` | Session country |
| `DA.email_sent` | Sent emails (`id_message`, `id_account`, `sent_date`) |
| `DA.email_open` | Opened emails (`id_message`) |
| `DA.email_visit` | Emails that led to a site visit (`id_message`) |

## Metrics

| Column | Description |
|---|---|
| `account_cnt` | Number of unique accounts |
| `sent_msg` | Number of unique sent emails |
| `open_msg` | Number of unique opened emails |
| `visit_msg` | Number of unique emails that led to a visit |
| `total_country_account_cnt` | Total accounts per country (window function) |
| `total_country_sent_cnt` | Total sent emails per country (window function) |
| `rank_total_country_account_cnt` | Country rank by total accounts |
| `rank_total_country_sent_cnt` | Country rank by total sent emails |

## Dimensions

`date`, `country`, `send_interval`, `is_verified`, `is_unsubscribed`

## Query structure

| Step (CTE) | What it does |
|---|---|
| `base` | Joins accounts with sessions to get the account date and country |
| `account_met` | Counts unique accounts by date, country and account attributes |
| `email_met` | Counts sent, opened and visited emails; the email date is the session date plus `sent_date` days |
| `union1` | Combines both metric sets with `UNION ALL`, filling the missing metrics with zeros |
| `final_mtrx` | Sums the combined data so each dimension combination has a single row |
| `final0` | Adds country totals with window functions |
| `final` | Ranks countries with `DENSE_RANK()` |
| Final `SELECT` | Keeps only countries in the top 10 by accounts **or** by sent emails |

## How to run

1. Open the BigQuery console and select a project with access to `data-analytics-mate.DA`.
2. Paste the contents of [`main.sql`](main.sql) into the editor.
3. Run the query.
4. Export the result or connect the query as a data source in your BI tool.

## Notes

- Metrics use `COUNT(DISTINCT ...)`, so repeated rows of the same account or email are not double-counted within one group.
- The query assumes that an account is linked to a single session in `DA.account_session`. If an account can have several sessions, the same emails may appear in several date/country groups and be counted more than once in the sums. This can be checked with:

  ```sql
  SELECT account_id, COUNT(*) AS sessions
  FROM `DA.account_session`
  GROUP BY 1
  HAVING COUNT(*) > 1;
  ```

- Window totals are calculated over the whole dataset, not per date.

## Tech stack

Google BigQuery (Standard SQL): CTEs, joins, `UNION ALL`, window functions (`SUM() OVER`, `DENSE_RANK()`).

## Repository contents

| File | Description |
|---|---|
| `main.sql` | The analytical query |
| `README.md` | Project description |
