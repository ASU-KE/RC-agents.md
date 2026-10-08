# ASU RC Storage, I/O, and Transfer Guidance

Read this before choosing storage, creating many files, scanning a large path,
or moving substantial data. The rules in
[`ASU_RC_AGENTS.md`](../ASU_RC_AGENTS.md) still apply.

Current references: [File System Overview](https://docs.rc.asu.edu/file-system-overview),
[Scratch](https://docs.rc.asu.edu/scratch),
[Project storage](https://docs.rc.asu.edu/project-storage),
[/home cleanup](https://docs.rc.asu.edu/home-directory-cleanup-guide).

## Where things go

| Path | Limit | Backed up | Use for |
| --- | --- | --- | --- |
| `/home/$USER` | 100 GiB | no | small personal files, scripts, configs, small envs. Not for heavy I/O. When it fills, jobs and web-portal sessions fail. |
| `/scratch/$USER` | 100 TB | **no** | active job data, large intermediates, model/data caches. **Files not accessed for 90 days are deleted.** |
| `/data/<grp_…>`, `/data/<project>` | 100 GB free, then purchased | 14 days of nightly snapshots, on-site only | shared project data, kept long term. Same on Sol and Phoenix. **No sensitive data.** |
| `/tmp` | node-local, shared with other jobs | no | per-job temporary files; copy out anything you need before the job ends |
| Canyon (tape) | purchased | — | long-term, rarely accessed archives |

Anything that must outlive the scratch purge belongs in `/home`, `/data`, or
Canyon. Never write a workflow that depends on scratch being permanent.

Large caches belong on scratch, not in `/home`: Hugging Face (`HF_HOME`),
mamba package caches, model weights, and container images (`.sif`). Before
filling `/home`, run `mamba clean --all` and clear `~/.cache/pip`.

## `/data` is automounted

`ls /data` shows only directories that are currently mounted; there are
thousands of others. Always use the full path (`/data/grp_x/...`), which is
always reachable. Do not enumerate, `find`, or `du` across `/data` to discover
projects. That triggers mounts and metadata load, and it is not how you find
the user's paths. Ask the user, or use Voyager `get_my_account_profile`.

## Shared-filesystem hygiene

Shared filesystems suffer from:

- millions of small files, or thousands in one directory;
- repeated scans and frequent create/delete cycles;
- small reads and writes;
- many processes hitting one directory at once.

To reduce the load:

- Bundle small files into tar, HDF5, Zarr, Parquet, SQLite, or LMDB, and
  unpack them to `/tmp` or scratch inside the job.
- Cache metadata you would otherwise rescan.
- Use I/O blocks of at least 64 KiB, and several MB where practical.
- Estimate the scope of a recursive operation before running it, and narrow
  it when you can. For finding what fills `/home`, use `gdu ~` or
  `ncdu ~`.

## Transfers

- **Globus:** best for large data, many files, cross-institution transfers, and
  anything that needs to resume.
- **rsync between Sol and Phoenix:** run it on the data transfer node
  (`ssh soldtn`), **never on a login node**. `/data` is shared by both
  clusters and needs no copying.
- **Web portal** ([sol.asu.edu](https://sol.asu.edu), [phx.rc.asu.edu](https://phx.rc.asu.edu)):
  quick, occasional uploads and downloads.
- **scp, sftp, rsync, rclone:** scripted or repeatable transfers. Each new
  connection to a login node prompts for Duo, so do not script loops that
  open many connections.
- Do not set up SSHFS or other auto-reconnecting mounts. They cannot answer
  Duo, so they fail and retry repeatedly against the login nodes.

For large or long transfers, use checksums or verification and resume support.

## Sharing with other users

Share through group `/data` directories or ACLs as described in
[Sharing between users](https://docs.rc.asu.edu/sharing-files). Never
make `/home` or `/scratch` world-writable, and never `chmod -R 777`.
