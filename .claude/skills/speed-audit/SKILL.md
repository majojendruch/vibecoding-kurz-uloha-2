---
name: speed-audit
description: "Use when the user asks to audit, monitor or report on the WordPress sites in sites.json: are they up, are they fast enough, weekly speed report, or comparing a site against the performance budget."
---

# Speed audit

The routine for the sites in `sites.json`: is the site up, is it fast enough, and if not, why.

## Steps

1. **Read `sites.json`.** If it is missing, stop and ask. Never invent URLs.

2. **Availability first.** Delegate to the `uptime-monitor` subagent.

   A site that is down is not a speed problem. If a site comes back `critical`, skip its speed
   test: measuring the load time of an error page produces a meaningless number.

3. **Speed.** Delegate to the `speed-tester` subagent for the sites that passed step 2.

4. **Diagnose the failures.** For every site over budget, load the `speed-fix` skill and follow it.

5. **Report.** Write the result as a short markdown table: site, status, score, verdict, and for
   failing sites the one change with the biggest expected win. Write it for the person paying for
   the site, not for a developer.

## Why two subagents

Both run in their own context window. One Lighthouse report is hundreds of kilobytes of JSON; ten
sites would bury the conversation before the report gets written. The subagents chew through that
and hand back a table.

## Rules

- Never change a client site to test a hypothesis. Write down what you would change and why.
- Every site in `sites.json` appears exactly once in the report.
- Every failing site has a named cause, or an explicit "cause not identified".
