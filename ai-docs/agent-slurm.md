# ASU RC Slurm, Partition, and Node Guidance

Read this before choosing nodes, submitting jobs, starting interactive
sessions, inspecting scheduler state, or troubleshooting Slurm. The rules in
[`ASU_RC_AGENTS.md`](../ASU_RC_AGENTS.md) still apply.

Current references: [Partitions and QoS](https://docs.rc.asu.edu/partitions-and-qos),
[Requesting Resources](https://docs.rc.asu.edu/requesting-resources),
[Resource Limits](https://docs.rc.asu.edu/resource-limits),
[Interactive Sessions](https://docs.rc.asu.edu/interactive-sessions).
Limits change, so prefer the live pages (or Voyager `search_docs`) over the
numbers copied here.

## Node roles

| Node | Use it for | Not for |
| --- | --- | --- |
| Login (`sol-login*`, Phoenix login) | editing, reading source, writing job scripts, `sbatch`/`salloc`, light monitoring | computation, package installs, environment builds, model downloads, rsync, servers, VS Code Remote-SSH |
| Data transfer (`soldtn`) | `rsync` and bulk copies between systems | computation |
| Compute (via Slurm) | everything substantial | — |

Login nodes require Duo on each new SSH connection. Do not try to get around
this with stored second factors, unattended re-login, or persistent tunnels. For
IDE work, use a VS Code tunnel running in a `lightwork` job (see
[VS Code](https://docs.rc.asu.edu/vscode)), not Remote-SSH to a login node.
Remote-SSH spawns server processes on the login node that commonly trip
usage limits.

## Choosing a partition and QOS

ASU job scripts name both a partition (`-p`) and a QOS (`-q`), and the choice
matters. It decides which hardware the job can reach, whether the job can be
preempted, and what it costs in fair-share.

| Need | Use |
| --- | --- |
| Most CPU/GPU work, up to 7 days | `-p public -q public` |
| Fits in 4 hours (faster start; includes owner nodes, no preemption) | `-p htc -q public -t 0-4` |
| Light interactive work: mamba envs, compiles, VS Code tunnel, bulk file ops (≤8 cores, ≤24 h) | `-p lightwork -q public` |
| Checking a script for syntax/path/module errors (≤15 min, max 2 queued) | `-q debug -t 15` with `public`, `htc`, `general`, or `lightwork` |
| More than 512 GB memory (≤48 h) | `-p highmem -q public` |
| More than 7 days on RC hardware (batch only, ≤14 days) | `-p public -q long` |
| The user's lab owns hardware | `-p general -q grp_<labname>` (no fair-share cost) |
| Borrowing idle owner hardware, **user accepts preemption** | `-p general -q private` |
| Class accounts | `-A class_<…>`; tighter limits apply (2 running, 24 h, 4 GPUs) |

Rules:

- **`lightwork` is not a compute partition.** Jobs that hold cores above 99%
  for extended periods, or that request excessive resources there, can be
  cancelled. Move real compute to `htc`, `public`, or `general`.
- Use `private` only with the user's explicit agreement to preemption, and
  preferably with checkpointing. It is cancelled when the owning group needs
  the node.
- Users with both class and research accounts must pass `-A` explicitly.
- `arm` (Grace Hopper, aarch64) and `fpga` are special-purpose. Confirm the
  software supports that architecture or device before you request them.
- Request node features or constraints only when they are real requirements.
  Each one shrinks the set of eligible nodes. Node shapes and GRES (including
  MIG slices) can be checked with Voyager `get_cluster_nodes`; that tool's
  `features` field is not reliable.

## Resource requests

Every job specifies cores, nodes, memory, and wall time. Always request memory,
even small amounts. Do not request:

- all of a node's memory by default;
- GPUs for code that does not use them;
- many cores for single-threaded code;
- wall time far beyond the measured need.

While you are still testing, it is fine to pad wall time a little so a job
isn't killed just before it finishes. Tighten it once you have measurements.
Use `seff <jobid>` after a run to compare what was requested with what was
used. `seff` does not report GPU use; check GPU utilization separately
(`nvidia-smi` inside the job, or Voyager `get_my_gpu_efficiency`). Do not hold
GPUs that sit idle.

The `interactive` command is a shortcut for
`salloc -c 1 -p htc -q public -t 0-4`. Add `-p lightwork` for a near-instant
core for light tasks.

## Sizing, waiting, and short jobs

- Run a small representative test, measure, adjust, then scale up.
- Do not submit duplicates because a job is pending, and do not resubmit a
  failed job without finding the cause. Pending reasons are in
  `squeue -u $USER` and `scontrol show job <id>`.
- Wait at least 60 seconds between status queries, back off for long or
  pending jobs, and run at most one monitoring loop. Prefer `sbatch --wait`,
  `--dependency`, `scontrol wait_job`, or reading the job's output files.
- Aggregate short tasks to about 10–30 minutes of useful work per job. Use
  arrays and throttle them (`--array=1-1000%50`). Never pad with `sleep`.

## If Slurm appears down

Do not report an outage because of a timeout or because a client command
failed inside an agent sandbox. Work through these checks first:

- **Login-shell delays.** Agent tools often start login shells, and Lmod
  initialization can take several seconds. A short outer timeout can expire
  before the Slurm command even runs. Allow at least 15 seconds, or use a
  non-login shell.
- **Sandboxing.** The sandbox may block the client from reaching the
  controller. If the command works only outside the sandbox, the problem is
  the execution context, not Slurm.
- **Wrong host.** Confirm `type squeue sbatch` and that you are on a Sol or
  Phoenix login node. Slurm commands are not available on a workstation.
- **Rapid retries.** Do not run many probes in parallel or retry quickly;
  each retry adds load on the controller.

If the problem persists, give the user the hostname, time, and exact error to
pass to RC support.

## Long jobs and checkpointing

For jobs that run a day or longer, or that use `private`, recommend
application-level checkpoint/restart. That protects against wall-time limits,
failures, maintenance, and preemption. If a first draft doesn't need it yet,
leave a short `TODO` near the main loop.
