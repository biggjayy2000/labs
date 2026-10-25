# SQL filtering to investigate suspicious logins

**Narrowing authentication logs with `AND`, `OR`, `NOT` and a join against employee records**

| | |
|---|---|
| Type | Authentication log investigation |
| Data sets | `log_in_attempts` (timestamps, locations, outcome), `employees` (user, device, department) |
| Language | SQL |
| Status | Complete |

> Scenario-based: the data sets come from the Google Cybersecurity Professional Certificate case
> study. The queries and the reasoning are my own.

## Objective

Authentication tables are too large to read, and the interesting records are a small fraction of
them. The task was to use filtering to get from thousands of rows to the handful worth a human
looking at, then join employee records so a username becomes a person and a device.

## Query 1 — failed attempts on a specific day (`AND`)

```sql
SELECT *
FROM log_in_attempts
WHERE login_date = '2022-05-09'
  AND success = FALSE;
```

`AND` requires both conditions, so the result is only the failures on the day under investigation.
Filtering on date and outcome together removes the bulk of unrelated activity and leaves a set small
enough to analyse.

## Query 2 — activity from two countries (`OR`)

```sql
SELECT *
FROM log_in_attempts
WHERE country = 'CHN'
   OR country = 'RUS';
```

`OR` returns a row when either condition holds, which is how you express "in this set of locations".
Geographic filtering finds authentication activity outside an organisation's expected pattern. A
foreign login is not automatically malicious — it is an indicator that earns further investigation,
which is a different thing from a finding.

## Query 3 — everything outside the home country (`NOT`)

```sql
SELECT *
FROM log_in_attempts
WHERE NOT country = 'USA';
```

`NOT` reverses the condition. This is the query that builds a geographic baseline: anything that is
not the expected origin. Results need context to be judged — an employee travelling produces exactly
the same row as an attacker — so this is a starting set, not a verdict.

## Query 4 — joining logs to people (`JOIN`)

```sql
SELECT employees.*, log_in_attempts.*
FROM employees
JOIN log_in_attempts
  ON employees.username = log_in_attempts.username
WHERE log_in_attempts.success = FALSE
  AND log_in_attempts.login_date = '2022-05-09';
```

Login logs alone can only say "user `x` failed at 08:14". The join on `username` brings back the
employee record and the device they use, which is what turns a row into an action: review endpoint
activity for that device, contact the employee to confirm, or force a credential reset.

## What this demonstrates

- Filtering large authentication data down to a defined investigative set, and knowing which operator
  expresses which question.
- Correlating two data sets so the output answers "who and which machine", not just "what happened".
- The judgement that goes with it: an unusual country or a failed login is an indicator requiring
  context, and calling it a compromise without that context would be wrong.

## Limitations

The queries answer defined questions on a fixed data set — no attempt was made to tune thresholds,
score risk, or automate the triage. The join assumes `username` is a reliable key across both tables,
which in a real environment is worth verifying before trusting the output.

---

*Full document: [`SQL Filtering to Investigate Suspicious Login Activity`](https://docs.google.com/document/d/1J3o_rwLM5hVT2D9qI-BkJYnwnVdYOtf-qVdy3WWZ5zM/edit)
in the Drive archive.*
