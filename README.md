# Movie Suggester

Build a film recommender over 1455 films, with an AI coding agent as your pair.

## Setup

Clone the repo:

```sh
git clone https://github.com/Patryk-Kuchta/agentic-development-exercise.git
cd agentic-development-exercise
```

You need **Node 24** (pinned in `.nvmrc`). Any way of getting it works — nvm is just the
easiest:

- **Linux, macOS, WSL** — [nvm](https://github.com/nvm-sh/nvm#installing-and-updating), then
  `nvm install && nvm use`. It reads `.nvmrc`.
- **Native Windows** — nvm does not support it; use
  [nvm-windows](https://github.com/coreybutler/nvm-windows#installation--upgrades), which
  ignores `.nvmrc`: `nvm install 24 && nvm use 24`.

Then:

```sh
npm install
npm run dev
```

Open <http://localhost:5173>. The API downloads the dataset on first start, so the film count
fills in shortly after.

`stmt.setReturnArrays is not a function` means Node 23 or below, and nothing else.

## What you start with

- **1455 films** from
  [`MongoDB/embedded_movies`](https://huggingface.co/datasets/MongoDB/embedded_movies) — plots,
  cast, genres, ratings, posters, and a 1536-float `plot_embedding` for all but 27 of them.
- **Three workspaces**: `packages/contract` (the source of truth), `apps/api` (Express + oRPC +
  Drizzle over `node:sqlite`), `apps/web` (Vite + React 19 + Mantine + TanStack Query).
- **Somewhere to browse them** — a paginated list with search by title, filter by genre and sort
  by rating, all in the URL, and a page per film. SQLite → API → React, end to end.
- **Accounts** — sign-up, sign-in, sign-out and a header. Sessions live in the API process rather
  than the database, so **restarting the API signs everyone out**. That is deliberate: who is
  signed in is a fact about a running process, and a teaching app may lose it.
- **An empty `favourites` table**, migration committed. Nothing reads or writes it yet.
- **The gate.** `npm run check` — types, lint, format, tests, `db:check`, build — is the single
  definition of done. A pre-commit hook lints what you stage, and CI runs the gate on every push.
- **The house rules in writing** — `AGENTS.md` and the per-workspace `CLAUDE.md` files, for you
  and the agent both.
- **No infrastructure.** No Docker, no database server, no API keys.

## Your branch

Your trainer will tell you which branch to start on.

```sh
git checkout <branch>
```

| Branch                        | Start here for     | Adds                                                                 |
| ----------------------------- | ------------------ | -------------------------------------------------------------------- |
| `main`                        | Exercise 1         | The app, the gate and the written house rules                        |
| `exercise-1-solution`         | —                  | A worked answer to exercise 1                                        |
| `exercise-2-setup`            | Exercise 2         | The skills, subagents and hooks in `.claude/`, and two MCP servers   |
| `extension-exercise`          | Extension exercise | `exercise-2-setup` plus the example favourites — exercise 2's answer |
| `extension-exercise-solution` | —                  | A worked answer to the extension exercise                            |

Each branch is built on the one above it, except `exercise-1-solution`, which sits on `main`. A
fresh clone is already on `main`, so exercise 1 needs no checkout. Run `npm install` after every
checkout: `exercise-2-setup` adds packages `main` does not have.

Read a solution branch after you have built your own, not instead of it. To carry your own work
into the extension, see **Advanced** below.

<details>
<summary>Moving between branches</summary>

Work on your own branch, so a checkout never threatens your code:

```sh
git switch -c my-exercise-1     # before you start
```

Commit or stash before switching, or git refuses:

```sh
git stash                       # always works
git checkout exercise-2-setup
npm install                     # picks up the packages this branch adds
git switch -c my-exercise-2
```

Committing works too, but the pre-commit hook lints what you staged — so
`git commit -am "wip"` can be refused by the very errors you were parking.
`git stash` or `--no-verify` never is.

Back to the start, or to see what is on offer:

```sh
git checkout main
git branch -a
```

To push your own work, fork the repo on GitHub first and push to your fork.

</details>

## The exercises

Each one levels up **how** you use the agent, not just what you build — and that, not the app,
is the point. Exercises 1 and 2 build the same feature twice, so the only thing that changes
between them is the way of working; the extension is open-ended. Every exercise ends with the
way of working it is asking for.

### Exercise 1 — Favourites

Start on `main` · answer on `exercise-1-solution`

Accounts work and the `favourites` table exists; nothing connects them. Wiring it up is the
exercise — you should not need `npm run db:generate` at all. Start by reading
`apps/api/src/accounts.ts`. **A route is not done without a test that calls it through the
client.**

- **Favourite and unfavourite a film.** Favouriting one that does not exist is a 404, doing it
  signed out is a 401, and a double-clicked heart is not an error.
- **Every film says whether you have favourited it**, and a list can be narrowed to just those.
  Anonymous callers get an answer rather than an error — and a page of cards costs one query,
  not one per card.
- **Favouritedness is answered per request**, never stored on the film.
- **A heart wherever a film appears**, a favourites page, and a UI that still agrees with the
  server after every change.

**Done when** you can favourite three films, sign out, sign in as somebody else, see none of
theirs, sign back in and find all three — as long as you have not restarted the API.

<details>
<summary><b>Working with the agent</b> — plan, test, close the loop</summary>

The gate exists, so let the agent run it and review outcomes instead of characters.

- **Ask before you build.** "How does the contract turn into a React hook?" Use it to
  understand the codebase, not to skip it.
- **Plan first**, and push back on the plan. A bad plan is cheap to fix; bad code is not.
- **Ask for the test first**, then the code that passes it.
- **Let it run `npm run check`** and iterate until green without you relaying error messages.
- **Read the diff, not every line.** Look for what tests cannot catch: a duplicated type, a
  swallowed error, a hand-written `fetch`.
- **Ask why.** If it cannot justify a line, do not keep the line.

</details>

### Exercise 2 — Favourites, again

Start on `exercise-2-setup` · answer on `extension-exercise`

This branch adds `.claude/` — the skills, subagents and hooks listed in **The skills** at the
end — and two MCP servers. Use them; this is the exercise where you drive the agent rather than
type.

Build [exercise 1](#exercise-1--favourites) again: same brief, same **done when**. Start a fresh
branch off `exercise-2-setup` rather than carrying your exercise 1 across — the point is the
comparison.

**Done when** exercise 1's done-when holds, `npm run check` is green, and you can say what the
tooling changed: what you typed, what you caught yourself, what a reviewer or a hook caught for
you. Put the two side by side:

```sh
git diff main my-exercise-1 --stat
git diff exercise-2-setup my-exercise-2 --stat
```

<details>
<summary><b>Working with the agent</b> — drive it</summary>

You know the feature now, so the work is directing the agent, not writing the code.

- **Use the skills** — by name, or let the agent pick. `add-feature` is the procedure for this
  whole exercise.
- **`grill-me` on your plan** before anything is built.
- **Hand it whole slices** — "favouriting, contract to UI" — and judge the result against the
  brief.
- **Run `implementor-reviewer`**, and let the reviewers argue with the implementor instead of
  you.
- **Let it see what it built.** The `sqlite` MCP server reads the real `favourites` rows;
  `playwright` clicks the heart (it needs
  `npx @playwright/mcp install-browser chrome-for-testing` once).
- **Try the gate-on-stop hook.** Uncomment the `Stop` block in `.claude/settings.json` and the
  agent cannot end a turn on a red gate. It runs the whole gate every turn, so it is slow.
- **Write a prompt you would reuse.**

</details>

### Extension exercise — Suggestions

Start on `extension-exercise` · answer on `extension-exercise-solution`

This branch is `exercise-2-setup` plus the example favourites, so the tooling is all still
there. To build on your own favourites instead, see **Advanced** below.

Every film carries a `plot_embedding` — a vector standing in for what its plot is about, so
films with similar plots have similar vectors. What to do with that is the exercise.

- **"More like this"** on a film page, with the scores shown, so a suggestion is inspectable
  rather than magic.
- **Suggestions from your favourites** — films you have not already favourited, each one
  attributed to the favourite that earned it: "because you liked _Alien_". No favourites yet
  gets something better than an empty page.
- **Honest edges.** Not every film has an embedding, and no film is its own suggestion.
- **Fast, and server-side.** No vector database, no API key, no new service, and nothing that
  sends embeddings to the browser.

Decide with the agent how similarity should work, what makes a suggestion a good one, and what
tests would convince you the numbers are right rather than merely plausible.

**Done when** any film shows plausible neighbours, and three favourites produce a sensible
suggestion that names the favourite it came from.

<details>
<summary><b>Working with the agent</b> — design with it</summary>

The problem is open-ended, so the work is mostly deciding what to build.

- **Brainstorm the design.** What makes a _good_ suggestion? How do you know it worked?
- **Let it disagree with you**, and take the argument seriously.

</details>

<details>
<summary><b>Advanced</b> — carry your own favourites into the extension</summary>

`extension-exercise` contains the _example_ favourites, so checking it out replaces your work
with somebody else's. Your `my-exercise-2` is already `exercise-2-setup` plus your own
favourites — just keep going there:

```sh
git checkout my-exercise-2
```

To build on your exercise 1 instead, merge the tooling into it:

```sh
git checkout my-exercise-1
git merge exercise-2-setup
npm install
```

Hand the whole job to Claude — the merge, any conflicts and whatever the gate says afterwards.
It is exactly the kind of work it is good at.

</details>

## The stretch goals

Thirty-odd small extras. No answer branches; you are on your own, which is the point.

<details>
<summary>Pick one you would actually use</summary>

**Games**

- **Higher or lower** — two posters, guess which IMDb rates higher. The design problem is
  picking a fair pair.
- **Guess the plot** — a plot with the title and cast redacted. Redaction leaks the answer in
  more ways than you expect.
- **Odd one out** — three films close in embedding space, one from far away.
- **Six degrees** — connect two actors through shared casts, and show the chain.
- **Film of the day** — one film for everybody, seeded by the date, so a test can assert it.
- **Blind taste test** — ten poster pairs build a taste vector without an account.
- **Plot chain** — hop from _Alien_ to _Legally Blonde_ by nearest neighbour.

**Charts and leaderboards**

- **Genre co-occurrence** heatmap. Comedy-Romance is the boring answer.
- **Critics versus the crowd** — IMDb against Metacritic, and who disagrees most.
- **The ninety minute wall** — runtime histogram, and rating by runtime.
- **Leaderboards** — most prolific, most awarded, best rated. Pick a minimum film count and
  defend it.
- **Your taste, in numbers** — your favourites against the catalogue average.

**Embeddings, further**

- **Mood shelves** — k-means the vectors into a dozen clusters and name each one.
- **The map** — the whole catalogue projected to 2D, coloured by genre.
- **Misfiled films** — the film furthest from the centroid of its own genre.
- **Near-duplicate hunt** — remakes, sequels, and films with the same plot.
- **Anti-suggestions** — the film least like your taste.
- **Explain the score** — shared genres, cast and words beside the cosine number.

**Craft**

- **Command palette** (ctrl-K), **poster fallbacks** for the dead URLs, **dark mode**,
  **shareable top five** in the URL, **favourites export/import**, **published API docs**, **a
  terminal client** over the same typed client, **keyboard navigation**, **an accessibility
  pass**, **skeletons instead of spinners**.

**Under the hood**

- **Add a column the ingest throws away** — the whole schema-to-UI flow end to end.
- **Make ingest legible** — progress, a summary, and no redone work.
- **A fixture database** so tests can assert _which_ neighbour comes back.
- **Measure the scan** at 1455 vectors, then at 150,000.
- **Break rule 2, deliberately** — free-text search with a real embedding model, optional and
  honest about it.

</details>

## Commands

| Command             | Does                                                    |
| ------------------- | ------------------------------------------------------- |
| `npm run dev`       | API and web, both watching                              |
| `npm run typecheck` | `tsc --noEmit` across the workspaces                    |
| `npm run check`     | The gate: types, lint, format, tests, `db:check`, build |
| `npm run ingest`    | Re-download the dataset (`-- --limit=50` for a slice)   |

Every branch has the gate, along with `lint`, `test` and the pre-commit hook. `AGENTS.md` and
the per-workspace `CLAUDE.md` files carry the detail for agents. `exercise-2-setup` adds
`mcp:sqlite` and `mcp:playwright`.

## The skills

Exercise 2's and the extension's: `.claude/` arrives with `exercise-2-setup`. Invoke a skill by
name, or let the agent pick.

| Skill                  | For                                                             |
| ---------------------- | --------------------------------------------------------------- |
| `start-here`           | What the app is, how to run it, what the exercises ask          |
| `learn-this-stack`     | The repo explained in a language you already write              |
| `learn-the-frontend`   | React, Mantine and TanStack Query from whatever UI you know     |
| `code-style`           | The naming, layout and conventions of this repo                 |
| `add-feature`          | A new entity or route, contract → api → web                     |
| `change-schema`        | A schema edit and the migration that belongs with it            |
| `add-dependency`       | Whether an npm package is allowed here                          |
| `use-the-library`      | The library feature that already does the job                   |
| `ingest-data`          | Running, limiting and resetting the movie ingest                |
| `debug-this-stack`     | Failures by symptom, each with its fix                          |
| `verify`               | Run the gate and read what it says                              |
| `self-review`          | Review the diff — yours and the agent's — before claiming done  |
| `grill-me`             | Be interrogated on a plan until it holds                        |
| `implementor-reviewer` | Loop an implementor against reviewers until the review is clean |
| `git-commit-format`    | Commit messages in this repo's format                           |

Beside them, `.claude/agents/` holds an implementor and two reviewers, and `.claude/hooks/`
blocks infrastructure, logs permission prompts, and — if you opt in — refuses to end a turn
until the gate is green. `.mcp.json` adds two MCP servers: `sqlite`, read-only queries on the
real data, and `playwright`, which drives the running app. `tools/mcp/README.md` has the setup
for other editors.
