# ASU RC Data, Security, Credentials, and Process Guidance

Read this before inspecting a project that may hold sensitive data, choosing
where prompts and files are sent, handling credentials, exposing a network
service, managing processes, or touching scheduled tasks. The rules in
[`ASU_RC_AGENTS.md`](../ASU_RC_AGENTS.md) still apply.

## Data classification comes first

ASU requires users to know the classification of the data they give to AI
tools, and to provide only what the task needs. Output generated from
sensitive data inherits that data's handling requirements. Agents enforce this
by stopping, not by investigating.

- Sol, Phoenix, and Horizon (`/data`) are **not** for sensitive or regulated
  data. Regulated work (HIPAA, FERPA, CUI, export-controlled, and data under
  restrictive agreements) belongs in the
  [KE Secure Cloud](http://links.asu.edu/kesc), including the
  [Aloe supercomputer](http://links.asu.edu/aloedocs).
- If anything suggests restricted data, stop reading project content. Signs
  include agency sponsors, data use agreements, patient or student records,
  "controlled" or "export" markings, and identifiers. Ask the user for a
  specific confirmation, and point them to the ASU Data Classification Tool.
- Do not open the content to decide whether it is restricted. When in doubt,
  stop and ask.

If the user confirms the project contains restricted data:

- Stop AI-assisted work in that project. Do not read any more source, data,
  logs, configuration, or documentation.
- Create a conspicuous `AI_AGENTS_PROHIBITED.txt` at the project root saying
  that AI agents must not work there, and add a prominent warning to any
  existing `AGENTS.md` or `CLAUDE.md`.
- Direct the user to KE Secure Cloud support
  ([request](https://links.asu.edu/kesc-support)) and to their data steward
  or ASU Cybersecurity.

## Know where prompts go

Everything an agent reads can end up in a prompt.

- The **RC LLM gateway** (`https://openai.rc.asu.edu/v1`, keys from the
  [Voyager portal](https://voyager.rc.asu.edu)) runs models on RC hardware, so
  prompts are not passed to a commercial provider. That makes it the better
  default for research code and data on RC systems. It still does not
  authorize regulated data; the user's unit's data-handling rules still apply.
- Commercial agents (Claude, Codex, Copilot, Gemini, Cursor) send file
  contents to the vendor. Use them only under an ASU-approved agreement and
  only for data the user is cleared to share that way.
- Do not upload large logs, datasets, generated files, or other users' content
  into a prompt. Extract only the part you need.

## Credentials

- Treat API keys, Voyager tokens, GitHub tokens, and SSH keys as passwords.
  Never commit, print, or log them, and never paste them into prompts.
- On a shared login node, **command arguments are visible to other users**
  through `ps`. Pass secrets through environment variables, mode-600 files, or
  a tool's own login flow, never in argv (for example, not as
  `curl -H "Authorization: Bearer $KEY"`).
- Keep agent configuration that embeds a key (`opencode.json`, MCP configs)
  out of Git, or reference an environment variable instead. If a key leaks,
  regenerate it in Voyager right away.

## Access and security boundaries

The following are prohibited and may result in account suspension. A user's
request does not override this.

- Backdoors, hidden access, web shells, reverse shells, persistent tunnels,
  relays, alternate SSH daemons, remote desktop services, or background agents
  that accept external commands.
- Always-on agent gateways or bots on RC systems (OpenClaw is explicitly
  banned on Sol and Phoenix).
- Getting around Duo/MFA, ASURITE authentication, resource limits, scheduler
  policy, permissions, or software vulnerabilities. This includes stored
  second factors, unattended re-login, and auto-reconnecting mounts.
- Accessing another user's data, jobs, processes, credentials, or private
  directories; collecting credentials; or changing authentication
  configuration.
- Hiding processes or services.

## Services on shared nodes

`localhost` is **not** an authentication boundary on a multi-user node. Other
users on the same host can connect to a listening port. Any service an agent
starts must:

- run inside a Slurm job, never on a login node;
- bind to localhost or the job's node only;
- require application-level authentication when the software supports it (a
  token or password, as Jupyter and vLLM `--api-key` do);
- be reached from the workstation through an SSH port-forward that ends with
  the job.

Do not describe an unauthenticated service as secure.

## Processes

Inspect only processes the user owns (`ps -u $USER`). VS Code servers,
language servers, notebook kernels, file watchers, and agent processes often
outlive a session. Point them out and offer to end them. Use `scancel` for
jobs. A stuck VS Code tunnel is usually a stale lock file; see
[Troubleshooting VS Code](https://docs.rc.asu.edu/troubleshooting-vscode).

## Cron and scrontab

RC runs periodic user automation through **`scrontab`** (Slurm cron), not
login-node `crontab`.

- In every session where you use, change, or recommend periodic tasks, first
  list `scrontab -l` and `crontab -l`, and ask the user, one entry at a time,
  whether each entry is still needed.
- Do not change or remove an entry without approval.
- Periodic tasks must be lightweight. Keep them free of sustained compute,
  heavy I/O, frequent Slurm queries, and large recursive scans. Schedule them
  no more than every six hours, and normally daily.

## Reporting problems

For apparent system problems, policy questions, or restricted-data concerns:

- Stop retrying.
- Record only non-sensitive facts: hostname, time, job ID, path.
- Keep restricted content out of prompts, logs, and tickets.
- Send the user to RC support through a [Service Now
  request](https://rto.asu.edu/request-help/) or
  [#rc-support](http://links.asu.edu/rc-support).

Never attempt to repair shared infrastructure.
