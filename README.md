# agents

My setup for managing agent skills declaratively, plus a few small global
instructions every agent reads. It's mostly a personal system. I'm putting it
up in case the pattern is useful, not because it's a polished tool.

## The idea

I run a handful of coding agents (Claude Code, Codex, Cursor, Gemini, Copilot,
pi), and I want all of them pulling from one set of skills, without copying
files around or letting six directories quietly drift apart. So this works like
a `Brewfile`, but for skills: one manifest declares what's installed and where
each piece came from, and everything else gets rebuilt from that.

Four parts, that's the whole thing:

- **`.skill-lock.json`**: the manifest. Every skill, its source repo, and a
  pinned hash, the source of truth. This is `skills`' own global (`-g`) lockfile,
  which is why it is dotted and lives in `~/.agents`; the undotted
  `skills-lock.json` you see in projects is its per-project sibling.
- **`sync`**: a small bash script that points each agent's `skills/` directory
  at one shared store (`~/.agents/skills`) with a single symlink apiece. Add a
  skill once and every agent sees it, live.
- **[`chopz`](https://github.com/fielding/chopz)** + **`.skill-bundles.json`**: the
  CLI that installs and manages the skills, plus the bundle file it reads. A bundle is
  a hand-written allowlist of skills and their sources (e.g. `gate` and everything it
  composes), so `chopz install gate` pulls exactly that set and nothing else.
- **`AGENTS.md`**: a short set of global instructions all the agents read
  (Claude picks it up through a `CLAUDE.md` symlink).

`chopz` (`npx @fielding/chopz`) does more than bundles: same-repo dependency
resolution, a dev-link loop for the skills I'm writing (`chopz link <repo>` makes edits
live), content-hash pinning, and a heuristic scanner. Anything it doesn't define it
forwards to `skills`, so it's the only CLI I keep on my PATH.

The skill content itself isn't in this repo. `skills/` is gitignored, the same
way you don't commit `node_modules`. It's build output: `.skill-lock.json`
records every skill and where it came from, and `chopz restore` rebuilds the
whole store from it. (`skills` itself only restores a per-project lockfile, not
the global one, which is the gap `restore` fills.)

```sh
git clone git@github.com:fielding/agents.git ~/.agents
cd ~/.agents
./sync                       # bridge the one store out to every agent
npx @fielding/chopz restore  # reinstall every skill from .skill-lock.json, then pin each
```

## What's mine vs. borrowed

Most of the skills I lean on are other people's good work, installed through the
CLI and tracked in the manifest. Big shout-out in particular to
**[Dan Kubb (dkubb)](https://github.com/dkubb/skills)**. His `atomic-changes`,
`state-space-minimization`, and the rest are load-bearing in how I write and
review code. Others come from vercel-labs, openai, and the wider agent-skills
community.

My own skills live in a separate, private repo and won't install for you. Half
of them are work-specific anyway. What's here is the plumbing and the manifest,
not the secret sauce.

## Honest caveats

This is a personal, opinionated setup. `sync` hardcodes my agent list and my
paths, and the manifest points at a private repo you can't pull, so it won't run
turnkey for anyone but me. Read it as a worked example of declarative skill
management rather than something to install. If the part you came for is the
shape of it (`manifest + sync + rebuild`), that idea travels just fine.
