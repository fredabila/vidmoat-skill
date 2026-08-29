# Vidmoat SKILL.md

The editing craft manual we hand to AI agents that drive
[Vidmoat](https://www.vidmoat.com) over MCP.

**[Read SKILL.md](./SKILL.md)**

## What it is

A tool schema teaches an agent the *controls*. Nothing teaches it the *craft*,
so agents produce edits that are technically valid, correctly aligned, and
lifeless. This is the missing half: pacing targets by format, title dwell time
as a formula, audio levels, colour intent, and the specific mechanics that most
often cost a run.

Facts that were measured on our own system are labelled as such and kept
separate from convention, so you know which parts you can argue with. That
includes two audio controls that do not do what their names suggest.

## How agents get it

| surface | how |
|---|---|
| Any MCP client | call the `get_skill` tool |
| Anything that can fetch | `https://www.vidmoat.com/skill.md` |
| Humans, and PRs | this repo |

## This is a mirror

**The canonical copy is <https://www.vidmoat.com/skill.md>**, served from the
file in the Vidmoat repo. A workflow here re-fetches it daily and commits any
change, so this copy cannot quietly drift from what agents are actually reading.
Editing `SKILL.md` in this repo directly will be overwritten by the next sync.

To propose a change, open an issue or a PR here and we will land it upstream, or
send it through the `report_issue` tool from inside a session.

## Licence

CC BY 4.0. Use it, quote it, adapt it for your own tooling. Attribution to
Vidmoat appreciated but the video-editing advice belongs to everybody.
