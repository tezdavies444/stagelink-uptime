# stagelink-uptime

An uptime probe for **stagelink365.com**, and nothing else.

It calls the site's public health endpoint every few hours and fails loudly if
three consecutive attempts don't come back `200`. GitHub emails the failure.

## Why this is a separate, public repository

The probe used to live in the application's own repository, which is private.
GitHub bills Actions minutes on private repositories and **rounds every job up
to a full minute**, so a short probe run frequently costs the same as a real
build. It grew into the single largest consumer of the monthly allowance and
starved CI of it.

Standard runners are **free and unlimited on public repositories**, and this
probe has nothing private to protect: it reads one public, deliberately
unauthenticated URL and looks at the status code. No application source, no
checkout of anything private, no credentials, no secrets. Moving it here made
it free and gave the whole CI allowance back.

## What's here

| Workflow | Does |
| --- | --- |
| `uptime.yml` | Probes the health endpoint 3× per run; red only if all three fail |
| `keepalive.yml` | One commit a month, so GitHub doesn't auto-disable the probe |

`keepalive.yml` is not busywork: GitHub disables scheduled workflows on public
repositories after 60 days of repository inactivity, and a monitoring repo is
never otherwise committed to — so without it the probe quietly switches itself
off after two months.

## Configuring

The target defaults to `https://stagelink365.com/api/health`. To point it
elsewhere, set a repository variable `HEALTH_URL` — no code change needed.

The endpoint answers `200 {"status":"ok"}` when healthy and `503
{"status":"degraded"}` when it isn't, so a plain status-code check is enough.

## Resolution

Currently every 6 hours, which suits a site that is pre-launch. **At launch,
change the cron in `uptime.yml` to `*/5 * * * *`** — five-minute checks are what
this was designed for and they cost nothing here.

Note that GitHub's scheduler is late under load and drops runs, so treat the
interval as a floor rather than a guarantee. If you later need real paging
(SMS, Slack) or a public status page, a dedicated monitoring service is the
right tool; this is a free, zero-dependency baseline.
