# ASU RC Development Guidance

Read this before writing or modifying code, scripts, or workflows. The rules
in [`ASU_RC_AGENTS.md`](../ASU_RC_AGENTS.md) still apply.

Login nodes are for editing, inspecting source, and modest development. Test
suites, benchmarks, builds, and anything sustained go through Slurm
([agent-slurm.md](agent-slurm.md)).

## Context and token efficiency

These are soft recommendations. They save context and, on the RC gateway,
shared capacity.

- Read a known path directly. Do not probe for it first with `ls` or `find`;
  only search if the direct read fails.
- Prefer targeted searches over repository-wide ones, and do not re-inspect
  structure you already know.
- Keep command output to what the next decision needs: `git status --short`,
  targeted `git diff`, filtered test output.
- Read only the relevant ranges of large files.
- Do not feed logs, generated files, lockfiles, build artifacts, session
  histories, or unrelated content to the model.
- Convert documents to Markdown locally (for example with
  [MarkItDown](https://github.com/microsoft/markitdown)) and share only the
  part you need. For sensitive input, run the conversion under
  `unshare -Unr` so it cannot reach the network.
- Mind the data classification of anything you send
  ([agent-security.md](agent-security.md)).

## Testing and validation

- Validate in proportion to the change: unit and regression tests for
  reusable code, plus linting, type checks, smoke tests, or example runs as
  appropriate.
- For computational workflows, first run a small representative case in a
  `debug` or `htc` job. Measure runtime and memory, adjust the request, then
  scale up.
- Do not repeat expensive runs without using the results to diagnose a
  failure or improve the request.

## Version control and project notes

- Use Git where practical. Keep commits focused, and never commit credentials,
  API keys, `opencode.json` with an inline key, generated data, or
  machine-specific paths.
- Keep a top-level `AGENTS.md` with the project's facts, commands, and
  conventions, and have it reference `ASU_RC_AGENTS.md`. The
  [RC Getting Started with AI guide](https://docs.rc.asu.edu/ai/getting-started)
  shows the shape.
- Keep a short lessons-learned file. Each entry should be standalone,
  actionable, and specific enough to be useful on its own.
- RC publishes HPC skills (Slurm scripts, stuck-job diagnosis, GPU sizing, data
  layout, Python environments, and more) under Voyager → **LLM Access** →
  **Skills**. Prefer them over guessing site details such as partitions,
  modules, or `#SBATCH` flags.
