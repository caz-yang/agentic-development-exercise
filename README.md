# Movie Suggester

Add favourites, then suggestions, to an app over 1455 films, with an AI coding agent as your pair.

## Setup

Clone the repo:

```sh
git clone https://github.com/caz-yang/agentic-development-exercise.git
cd agentic-development-exercise
```

You need **Node 24** (pinned in `.nvmrc`). Any way of getting it works — nvm is just the
easiest:

- **Linux, macOS, WSL** — [nvm](https://github.com/nvm-sh/nvm#installing-and-updating), then
  `nvm install && nvm use && nvm alias default 24`. It reads `.nvmrc`; the alias stops every
  new terminal going back to your old Node.
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

In a second terminal, run the gate. It should be green before you change anything:

```sh
npm run check
```

`No such built-in module: node:sqlite` or `stmt.setReturnArrays is not a function` means the
wrong Node, and nothing else: run `nvm use 24` in that terminal.

## What you start with

A working app over **1455 films** from
[`MongoDB/embedded_movies`](https://huggingface.co/datasets/MongoDB/embedded_movies) — plots,
cast, genres, ratings, posters — split into three workspaces: folders that are each their own
npm package, all installed by `npm install` at the root.

1. **`packages/contract` — the source of truth.** It declares the database tables (`schema.ts`), the validation derived from them (`zod.ts`), and every API route with what it takes and returns (`contract.ts`). The API and the web app both import it, so a change here shows up as a type error everywhere that has to follow.
2. **`apps/api` — the server.** It implements each route the contract declares.
   - **Accounts** live in `src/accounts.ts`: sign-up, sign-in, sign-out. Sessions are held in memory, not the database, so **restarting the API signs everyone out** — and `npm run dev` restarts it every time an API file is saved.
3. **`apps/web` — the browser app.** It calls the API only through hooks generated from the contract, never a hand-written `fetch`. Existing features:
   - a paginated movie list
     - with search by title, filter by genre and sort by rating
     - all kept in the URL, so a reload, the back button or a shared link lands on the same results
   - a page per film
   - sign-up and sign-in pages
   - a header showing who is signed in

**Also:**

- **The gate.** `npm run check` — types, lint, format, tests, `db:check`, build — is the single
  definition of done. A pre-commit hook lints what you stage — only that, so a green commit is
  not a green gate — and CI runs the gate on every push.
- **The house rules in writing** — `AGENTS.md` and the per-workspace `CLAUDE.md` files, for you
  and the agent both.
- **No infrastructure.** No Docker, no database server, no API keys.

## Your branch

Your trainer will tell you which branch to start on.

```sh
git checkout <branch>
```

| Branch                        | Start here for     | Adds                                                               |
| ----------------------------- | ------------------ | ------------------------------------------------------------------ |
| `main`                        | Exercise 1         | The app, the gate and the written house rules                      |
| `exercise-1-solution`         | —                  | A worked answer to exercise 1                                      |
| `exercise-2-setup`            | Exercise 2         | The skills, subagents and hooks in `.claude/`, and two MCP servers |
| `extension-exercise`          | Extension exercise | `exercise-2-setup` plus the solution to favourites                 |
| `extension-exercise-solution` | —                  | A worked answer to the extension exercise                          |

Each branch is built on the one above it, except `exercise-1-solution` and `exercise-2-setup`, which both sit on `main`. Run `npm install` after every checkout: `exercise-2-setup` adds packages `main` does not have.

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

## Commands

| Command             | Does                                                    |
| ------------------- | ------------------------------------------------------- |
| `npm run dev`       | API and web, both watching                              |
| `npm run typecheck` | `tsc --noEmit` across the workspaces                    |
| `npm run check`     | The gate: types, lint, format, tests, `db:check`, build |
| `npm run ingest`    | Re-download the dataset (`-- --limit=50` for a slice)   |

`exercise-2-setup` adds `mcp:sqlite` and `mcp:playwright`.

## The exercises

Exercises 1 and 2 build the same feature twice — the only difference is extra tooling. The focus is on **how** you work with the agent.

### Exercise 1 — Favourites

Start on `main` · answer on `exercise-1-solution`

> [!IMPORTANT]
> **Your Task: Implement a new feature that lets signed-in users favourite films.**

**Starting Point:** Accounts work, and the `favourites` table is declared in
`packages/contract/src/schema.ts` with its migration committed. Nothing connects them yet: that is
the exercise. The table is already right, so you should not need a new migration.

**<ins>Acceptance criteria</ins>**

- **Favourite and unfavourite a film.** An unknown film is a 404, signed out is a 401, and a
  double-clicked heart is not an error.
- **Every film says whether you have favourited it**, and a list can be narrowed to just those.
  Anonymous callers get an answer, not an error. A page of cards costs one query, not one per card.
- **Favourited or not is worked out per request**, never stored on the film.
- **A heart wherever a film appears**, a favourites page, and a UI that still agrees with the
  server after every change.
- **Every new API route has a test.** A route is an endpoint declared in `contract.ts`; its test
  boots the real API and calls it with the typed client, like those in `apps/api/test/`.

**Done when:** you can favourite three films, sign out, sign in as somebody else, see none of
theirs, sign back in and find all three — as long as you have not restarted the API — and
`npm run check` is green.

<details>
<summary><b>Working with the agent</b> — plan, build in pieces, review</summary>

You own the plan and the spec, the agent writes the code, and the gate referees. This is a
rough shape, not a script: change it as you find what works.

1. **Get it running and look around.** Sign up, browse, open a film. Then ask the agent for the
   architecture, a diagram of it, and where each part lives. Ask it anything you are unsure of.
2. **Understand the part you will change** before you/the agent change it (e.g. how does the API know who is signed in?)
3. **Plan with it, and hunt edge cases together** until you agree on what is being built. A good
   plan follows the house order: contract, then API with tests, then web. Be wary of one that
   starts from the UI.
4. **Write the spec yourself, in a file.** For each route or screen: what it does, what it
   returns, how it fails, and the test that proves it.
5. **Build in pieces you can review**, one route at a time with `npm run check` green
   before the next. Notice how it fixes red and call it out if it silences a check instead of fixing the cause (e.g. using `as`, `!` or `eslint-disable`). If it goes in circles, stop it and tighten the spec.
6. **Try it in the running app** against **done when** above.
7. **Review the diff** as if a colleague wrote it. The gate has checked the mechanics; look for
   what it cannot see:
   - a type or Zod schema that duplicates `schema.ts` or the contract;
   - an error swallowed, or a default guessed, to keep a path alive;
   - a test that passes because it asserts too little.

   **Ask why** about anything you do not follow. If it cannot justify a line, cut it.

8. **Iterate until you would merge it.** If time runs out first, note what you would still ask
   to change, and bring that to the retro.

</details>

### Exercise 2 — Favourites, again

Start on `exercise-2-setup` · answer on `extension-exercise`

> [!IMPORTANT]
> **Your Task: Build favourites again, this time driving the agent with the tooling in `.claude/`.**

**Starting Point:** `main` plus `.claude/`, which holds the skills, subagents and hooks listed in
**The skills** at the end, and two MCP servers. Start a fresh branch off `exercise-2-setup`
rather than carrying your exercise 1 across: the point is the comparison.

**<ins>Acceptance criteria</ins>:** [exercise 1's](#exercise-1--favourites), unchanged.

**Done when:** exercise 1's done-when holds, and you can say:

- what the tooling did that you would not have thought to ask for;
- what you caught that the reviewers did not;
- where you wanted the keyboard back.

Put your exercise 1 and 2 branches side by side:

```sh
git diff main my-exercise-1 --stat
git diff exercise-2-setup my-exercise-2 --stat
```

<details>
<summary><b>Working with the agent</b> — drive it</summary>

You know the feature now, so the work is directing the agent, not writing the code. The steps
follow exercise 1's, with the tooling taking a bigger share of each. Again, a rough shape.

1. **Get oriented with the skills.** Try `start-here`, and `learn-this-stack` if the stack is new
   to you.
2. **Plan with `grill-me`**, and let it interrogate you until the design holds. How does that
   compare with writing the spec alone?
3. **Hand it bigger slices than in exercise 1**, e.g. "favouriting, contract to UI", and judge
   the result against the acceptance criteria. `add-feature` is the procedure it should follow.
4. **Try `implementor-reviewer`.** It loops an implementor subagent against two reviewers, one
   hunting regressions and one hunting needless code, until the review is clean and the gate is green.
5. **Watch the permission prompts.** `.claude/settings.json` pre-approves the gate and the tests.
   What are you still asked for?
6. **Let it see what it built.** The `sqlite` MCP server reads the real `favourites` rows;
   `playwright` clicks the heart, once you have run `npx @playwright/mcp install-browser chrome-for-testing`.
7. **Run `self-review` before you read the diff yourself.** Then read it. Did it find what you
   found?
8. **Try the gate-on-stop hook.** Move the `Stop` block from `_disabledHooks` into `hooks` in
   `.claude/settings.json`, and the agent cannot end a turn on a red gate. It is slow.
9. **Keep a prompt you would reuse**, and bring it to the retro with your answers to **done
   when**.

</details>

### Extension exercise — Suggestions

Start on `extension-exercise` · answer on `extension-exercise-solution`

> [!IMPORTANT]
> **Your Task: Suggest films people will like, from what their plots are about.**

**Starting Point:** This branch is `exercise-2-setup` plus the solution to favourites, so the tooling is all still
there. To build on your own favourites instead, see **Advanced** below. Most films carry a
`plot_embedding`: a vector standing in for what the plot is about, so films with similar
plots have similar vectors.

**<ins>Acceptance criteria</ins>**

- **"More like this"** on a film page, with the scores shown, so a suggestion is inspectable
  rather than magic.
- **Suggestions from your favourites**: films you have not already favourited, each attributed to
  the favourite that earned it, as in "because you liked _Alien_". No favourites yet gets
  something better than an empty page.
- **Honest edges.** Not every film has an embedding, and no film is its own suggestion.
- **Fast, and server-side.** No vector database, no API key, no new service, and nothing that
  sends embeddings to the browser.

**Done when:** any film shows plausible neighbours, and three favourites produce a sensible
suggestion that names the favourite it came from.

<details>
<summary><b>Working with the agent</b> — design with it</summary>

The problem is open-ended, so most of the work is deciding what to build. Build and review as
in exercises 1 and 2; these steps are about the design.

1. **Understand the data first.** Ask the agent what an embedding is, and how you would tell
   whether two films are similar.
2. **Brainstorm the design.** What makes a _good_ suggestion, and how should similarity be
   scored?
3. **Let it disagree with you**, and take the argument seriously.
4. **Decide what would prove it works.** Which tests would convince you the numbers are right,
   not merely plausible?

</details>

<details>
<summary><b>Advanced</b> — carry your own favourites into the extension</summary>

`extension-exercise` contains the solution to favourites, so checking it out replaces your work
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

## Stretch goals

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
- **Break rule 2 in AGENTS.md, deliberately** — free-text search with a real embedding model, optional and
  honest about it.

</details>

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

All of it is plain text, and worth reading once you have seen it work:

- **A skill** is a folder under `.claude/skills/` holding one `SKILL.md`: a name, a one-line
  description that tells the agent when to use it, and the instructions. `grill-me` is the
  shortest; then read `self-review` and `add-feature`.
- **A subagent** is a markdown file under `.claude/agents/` in the same shape, run in its own
  context so its working stays out of yours.
- **A hook** is a shell script under `.claude/hooks/`, wired to an event in
  `.claude/settings.json`. `.claude/hooks/README.md` explains each.
- **`.claude/settings.json`** is also the permission policy: what runs silently, what asks, and
  what is refused.
