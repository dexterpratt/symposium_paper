# launchd plists — record of deployed scheduling

This directory holds the macOS launchd `.plist` files that scheduled the
agent fleet for the Symposium paper's data collection period (approximately
2026-04 through 2026-05).

These files are **a record of the deployment**, not a portable template.
They carry hardcoded absolute paths in `<string>` values
(`/Users/dexterpratt/.local/bin/...`, `EnvironmentVariables` `HOME`, etc.)
because launchd does **not** expand `~`, `$HOME`, or any other variable
syntax inside plist string values — they must be literal absolute paths
matching the user the daemon runs as.

The fleet was suspended on 2026-05-17 (in
`~/Library/LaunchAgents/_suspended_2026-05-17/`) ahead of the repo
split. The plists are preserved here for paper-reproducibility and as
the source artifact if a future deployment wants to re-establish the
schedule on a different machine.

## Re-deploying on a different machine

If you intend to actually load these onto a new machine's launchd:

1. Substitute every `/Users/dexterpratt/` with `/Users/<your-username>/`
   (or whatever your `$HOME` resolves to). Plists do not interpolate.
2. Confirm `/Users/<your-username>/.local/bin/run-scheduled-agent.sh`
   exists, executable, and references the new Memento checkout path.
3. Confirm `EnvironmentVariables.PATH` includes whatever binaries the
   agent runs need (`python3`, `git`, etc.).
4. Copy the (substituted) plist into `~/Library/LaunchAgents/` and
   `launchctl load` it.

## Why we kept the literals

For the paper repo, the plists are evidence-of-deployment. Rewriting
them with placeholders would obscure what was actually running. They
are the historical record, not a template artifact.

The companion `scheduled-tasks/*/SKILL.md` files (the prompts launchd
fed to Claude Code) **have** been rewritten to use `~/` notation, because
Claude Code's tools (`Read`, `Bash`) do expand `~` at fire time. Those
files are usable across machines without substitution; these plists are
not.

## File inventory

11 plists, one per scheduled agent slot:

- `com.dexterpratt.agent.rboreal-{morning,afternoon}.plist`
- `com.dexterpratt.agent.rcorona-{morning,afternoon}.plist`
- `com.dexterpratt.agent.rdaneel-quals-prep.plist`
- `com.dexterpratt.agent.rgiskard-{morning,afternoon}.plist`
- `com.dexterpratt.agent.rsolstice-{morning,afternoon}.plist`
- `com.dexterpratt.agent.rzenith-{morning,afternoon}.plist`
