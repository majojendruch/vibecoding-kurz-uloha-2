# wp-speedy

A Claude Code agent that watches WordPress sites: are they up, and are they fast enough.

Homework 2 for the VIP coding course: **configure a coding agent using MCP servers, skills and
subagents.** No plugins, no marketplace.

## Structure

```
wp-speedy/
├── .mcp.json                 # 2 MCP servers
├── sites.json                # the sites being watched + performance budget
└── .claude/
    ├── CLAUDE.md             # project rules
    ├── settings.json         # permissions: allow / ask / deny
    ├── agents/
    │   ├── uptime-monitor.md # is the site up?
    │   └── speed-tester.md   # how fast is it?
    └── skills/
        ├── speed-audit/      # the routine that runs both subagents
        ├── speed-fix/        # what to check when a site is slow
        └── wp-*/             # four official WordPress skills, unmodified
```

## MCP servers

Both run through `npx` and neither needs an API key. Both were tested by connecting to them over
stdio, listing their tools and calling one:

| Server | Package | Verified |
|---|---|---|
| `lighthouse` | `lighthouse-mcp` | `run_audit`, `get_performance_score`; returned a real score for a test page |
| `playwright` | `@playwright/mcp` | `browser_navigate`, `browser_snapshot`, `browser_console_messages`, `browser_close`; navigated, read the page and caught a console error |

Lighthouse measures. Playwright renders the page for real, which is how the agent catches a white
screen and JavaScript errors that a status code hides.

## Skills

Two written for this project:

| Skill | What it is |
|---|---|
| `speed-audit` | the routine: check availability, then speed, then diagnose, then report |
| `speed-fix` | routes each failing Lighthouse metric to the WordPress thing that usually causes it |

Four from [WordPress/agent-skills](https://github.com/WordPress/agent-skills), the official
WordPress project, installed with the repository's own installer and byte-identical to upstream:

| Skill | What it teaches |
|---|---|
| `wp-performance` | profiling, autoloaded options, object cache, cron, slow queries |
| `wp-project-triage` | detects what a WordPress repo actually is, with a script |
| `wp-wpcli-and-ops` | safe `search-replace`, db export/import, multisite |
| `wp-plugin-directory-guidelines` | GPL compliance and plugin review |

The upstream repository ships 18. The other 14 are about building Gutenberg blocks, block themes
and the REST API: useful for developing WordPress, not for monitoring it.

**One quirk.** The official skills refer to their helper scripts as `skills/<name>/scripts/…`,
which is where they sit upstream. Installed into a project they live under `.claude/skills/`, so
the command needs the prefix:

```bash
node .claude/skills/wp-project-triage/scripts/detect_wp_project.mjs
```

The skill files were left untouched so they stay identical to upstream and can be re-installed on
update.

## Subagents

| Subagent | Job | Tools |
|---|---|---|
| `uptime-monitor` | HTTP status, response time, whether the page really rendered, console errors | `Read`, `Bash`, four playwright tools |
| `speed-tester` | Lighthouse score against the budget | `Read`, two lighthouse tools |

**Why subagents and not two more skills.** A skill is instructions loaded into the main agent's
context. A subagent is a separate agent with its own context window and its own tool list.

*Context:* one Lighthouse run produces hundreds of kilobytes of JSON. In the main context window
that would crowd out the conversation before the report gets written.

*Tools:* `speed-tester` has no browser, `uptime-monitor` has no Lighthouse, and neither can write
files. Each can do its job and nothing else.

## Permissions

`.claude/settings.json`, on one rule: **look at anything, ask before changing anything.**

- **allow** - reading, searching, curl, the read-only WP-CLI commands, the two MCP servers
- **ask** - `Write`, `Edit`, `git commit`, `git push`
- **deny** - `rm -rf`, and reading `wp-config.php`, `.env` and `.sql` files

Those last three are the WordPress-specific ones: `wp-config.php` holds the database password,
`.sql` dumps hold customer orders. A speed agent has no business in either.

## Running it

```bash
git clone <this repo>
cd wp-speedy
npx playwright install chromium   # once: Playwright does not fetch its browser by itself
# edit sites.json: replace the examples with your own sites
claude
```

Then:

```
> run the speed audit
> are the sites up?
> how fast is the shop on mobile?
```

Requirements: Node 18+ and Chrome (Lighthouse needs a browser it can drive).

## Credits

The four `wp-*` skills are copyright the WordPress contributors, licensed GPL-2.0-or-later and
included unmodified. See `LICENSE-wordpress-skills`. Everything else is my own work for the course.
