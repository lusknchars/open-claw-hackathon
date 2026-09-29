# Repro Relay for Plow

Run Repro Relay as an OpenClaw agent on your own Plow line. Paste meeting notes or
send a todo. Reach saves action items with source quotes, owners and deadlines,
answers daily-work questions, and records completion reports across restarts.

This is the chat agent edition. The macOS application remains available from
[the main README](../README.md). This image has its own task store; desktop
synchronization, live call transcription, automatic reminders to all teammates
and verified repository fixes are not included in this edition.

## Install

From a copy of this repository, in its top folder:

```sh
./relay agent
```

It checks Docker, signs you in to Plow, selects a free assistant line, starts this agent, waits until Plow reports it configured, prints a first reply, and shows the number to text. Repeating it continues an interrupted install. It never takes a line that already answers as an agent, and never overwrites an existing credential.

Needs Docker Desktop running, Python 3, and the phone that owns the Plow account. The pinned upstream image is Linux amd64; Docker Desktop emulates it on Apple Silicon, so the first start downloads several GB.

```sh
./relay agent --line 2            # use a free line by list position, number or uid, without being asked
./relay agent status              # agent, line, Plow setup and reported usage
./relay agent test "prompt"       # one prompt, printed with its token usage
./relay agent stop                # stop it, keeping memory and identity
```

The agent runs on the Plow OpenClaw base, which sets its model through Plow, so
`./relay agent model` and `./relay agent provider` are not available on it yet.

### The same steps by hand

The command wraps the official Plow client. To run the steps yourself, clone
[plow-agents](https://github.com/plow-pbc/plow-agents), then from `repro-relay/agent`:

```sh
python3 ../../plow-agents/bin/plow-agents login
python3 ../../plow-agents/bin/plow-agents lines
python3 ../../plow-agents/bin/plow-agents mint ln_xxx
docker compose up --build -d
docker compose logs -f agent
```

Select a **free** line from the list and replace `ln_xxx`. An occupied line already has an
agent: keep it running and use a different free line. With none, run `login --new-line`,
complete its phone confirmation, then run `lines` again.

Wait for `[plow] connected account=chat` in the logs. Text the selected line:

> Meeting notes: Ana will review the export bug by September 18, 2026. Bruno
> will document setup by September 19, 2026. Save these tasks, then show Ana's work.

Reach returns saved task IDs. Send `Mark <task-id> done` and ask for open work
again. Completion here is your report, not an independently verified code fix.

The base runtime supplies Plow model access. Any provider charges and Latch
permissions are separate from installation. Latch connects to the account's own
Mac when available. Text notes directly for the first task; calendar, Sentry,
Mem0 and a Relay application server are not prerequisites.

## Usage reporting

This image includes the official Agent Index reporter. Every five minutes it
publishes this installation's daily token counts by model to AI Worth Using.
It sends no prompts, task titles, source quotes, files, phone numbers or costs.
Its random installation ID and report key persist in the named home volume.
The reporter registers this installation under `repro-relay` using your own Plow
authorization. It does not make you the publisher of someone else's listing.

The client establishes a baseline and can recover same-day OpenClaw session usage
when available. An idle agent may show measurement pending until it does work.
Do not repeatedly recreate volumes to increase install counts.

Check the client without posting a report:

```sh
docker compose exec agent env -u HERMES_HOME HOME=/var/lib/plow \
  python3 /opt/plow/agent-index-client.py status
docker compose exec agent env -u HERMES_HOME HOME=/var/lib/plow \
  python3 /opt/plow/agent-index-client.py --agent repro-relay --dry-run
```

Reporting ships in the Plow base image, which runs it from its own boot, so it
is part of this image's behavior. The first pass after a fresh install reports
nothing until the agent has answered once.

## Staying connected

The OpenClaw base reconnects the Plow chat socket with backoff when it drops,
and restarts its bridge to the owner's Mac if that bridge crashes, so this
image carries no watchdog of its own.

## Restart and update

`docker compose down` stops the agent and preserves its memory and installation
identity. `docker compose up -d` starts it again. After pulling an update, use
`docker compose up --build -d`. Keep the same checkout directory or Compose
project name so the named volume remains the same.

`docker compose down -v` permanently deletes this install's tasks, conversation
history and reporting identity. Use it only when intentionally removing data.
Credentials live in `agent/plow-credentials`, outside Git and the build context.
The bind mount refuses a missing file instead of silently creating a directory.
Never share that file or bake it into an image.

## Publisher and verification

[Publishing instructions](../docs/AGENT-INDEX.md) explain registration,
usage checks and the organizer's verification step. A listing or successful
image build alone is not verification or approval for hosted deployment.

Repro Relay's code and agent content are MIT licensed. Upstream Plow components
retain Apache-2.0 and other dependency licenses, included in `vendor/` and in
the base image. See [NOTICE](NOTICE).
