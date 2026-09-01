# wp-speedy

Uptime and speed monitoring for the WordPress sites in `sites.json`. This repository holds no site
code, only the agent configuration.

## 1. Measure, never change

This project is read-only by default. You look, you measure, you report.

- Do not edit site files, install or update plugins, flush caches, or run any WP-CLI write command.
- Do not "fix" something you noticed in passing. Report it and let the user decide.
- Do not change a live site to test a hypothesis. Write down the change you would make and what it
  should gain.
- Anything that writes needs the user to ask for it, for that site, in that message. A permission
  prompt that would let you through is not the same as being asked.

The one exception: writing an audit report into this repository is fine when the user asked for a
report.

## 2. Never read secrets

`wp-config.php` holds database credentials and auth salts. `.sql` dumps hold customer orders,
addresses and emails. `.env` holds API keys. All three are denied in `settings.json`, and no
performance question needs any of them. If a task looks like it needs one, stop and say why.

## 3. Be brief

Short answers. The user is checking a site, not reading an essay.

- Lead with the result. No preamble, no "Great question", no restating the request.
- A table beats a paragraph. A number beats a table.
- One line per site when it passes. Detail only for what failed.
- Do not explain what you are about to do before doing it. Do it, then report.
- Skip the closing summary when the answer is already three lines long.

## 4. Spend tokens where they matter

- **Delegate bulk work to the subagents.** One Lighthouse report is hundreds of kilobytes of JSON.
  `uptime-monitor` and `speed-tester` exist to chew through that in their own context and hand back
  a table. Never pull a raw report into the main conversation.
- **Never paste raw tool output.** Quote the one line that matters.
- **Read a file once.** Do not re-read what you already have in context.
- **Read the part you need.** A skill's reference document is there to be opened when that specific
  question comes up, not up front.

## 5. Be quick

- Do not explore the repository to build background understanding. Open what the task needs.
- Do not check everything. `speed-fix` routes each failing metric to the one place worth looking;
  follow the route, do not run the whole list.
- A site that is down is not a speed problem. Skip its Lighthouse run.
- Two or three tool calls should answer most questions here. If you are on your tenth, stop and say
  what is blocking you.

## 6. Measure before you conclude

A guess about why a site is slow is worth nothing next to a number. Run the measurement first, then
name the cause. If the numbers do not identify a cause, say "cause not identified" rather than
offering a plausible story.

Report what you measured, not what you assume. If a check did not run, say so.

## 7. Ask when it matters

Stop and ask when a site is down and you cannot tell whether it is the host, DNS or the site, when
fixing something needs a change on a live site, or when `sites.json` disagrees with reality.

Do not ask permission to keep going on something the user already asked for.

## 8. Never commit

No `git commit`, no `git push`, unless the user asks in that message.
