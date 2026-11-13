# TALON team: who does what

Last updated 2026-11-11 by Wali. Edit this file when anything in it stops
being true. The board's workers read `CLAUDE.md`, which carries a short copy
of the ownership table, so change both together.

TALON is an autonomous power-line inspection system: a drone, a dock with a
motorised gate, a radio tag on the drone, a laptop server, and a defect
model. `docs/PROJECT_BRIEF.md` explains it without jargon.

## People

| Person | GitHub | Owns | Working on right now |
|---|---|---|---|
| Austin | `Ausnguy` | Gate motor + its ESP32, defect model training, 3D-printed parts. Maintainer of this repo and the board | Wiring the gate motor to an ESP32. Training the model. Printing parts |
| Wali | `YatoVoid` | The ESP32 + relay that rides on the drone. Laptop server, upload, protocol, end-to-end tests | Hooking up the relay and ESP32 for the drone |
| Sakina | not added yet | Not decided | Still choosing a part. See "Open work" below |
| Aayan | `Sp00keee` | The operator website and the pitch: presentation, script, demo narration | Starting on the website (`frontend/`) and the pitch (`pitch/`). Start with `docs/PROJECT_BRIEF.md` |

## Roles on the board

A role is one folder, one branch (`agent/<role>`), and one listener. A task
addressed to a role is picked up by that role's listener on its owner's
machine. No listener running means the task waits.

| Role | Folder | Owner | What belongs there |
|---|---|---|---|
| `tag` | `tag/` | Wali | Firmware for the ESP32 on the drone, relay control, tag-side radio |
| `gate` | `gate/` | Austin | Firmware for the gate ESP32: motor, limit switches, dock sensor, safety stops |
| `ai` | `ai/` | Austin | Training code, model export, the `analyze(video)` function the server calls |
| `backend` | `backend/` | Wali | Laptop server: control API, event stream, video upload with SHA-256 check |
| `integration` | `integration/` | Wali | Shared contracts (radio frames, API), end-to-end and fault tests |
| `frontend` | `frontend/` | Aayan | Operator website: Start button, live state, review of findings |
| `pitch` | `pitch/` | Aayan | Script, slide outline, claims list, demo run order. Text only |
| `human` | none | every person | Anything a person has to do or decide. Never runs a worker |

3D printing has no role. It is physical work; ask Austin with a `human` request.

### Adding Sakina

Aayan (`Sp00keee`) is already in `.agentboard.json` with `human,frontend,pitch`.
Austin runs this once he has Sakina's GitHub login:

```powershell
python agentboard.py team add <sakina-login> --roles human
git add .agentboard.json; git commit -m "add teammates"; git push
```

Give her whichever role she picks. `frontend` and `pitch` are Aayan's for now.

## Where the pieces meet

Each row is an agreement between two owners. Post the agreed version on the
board before either side builds against it, then keep the file in
`integration/` current.

| Between | What has to be agreed | State |
|---|---|---|
| `tag` and `gate` | Which radio (ESP-NOW between the two ESP32s, or BLE), message bytes, who measures signal strength | **Open.** Wali has a frame format and a BLE draft from an earlier plan that assumed an Arduino UNO Q as the base. With two ESP32s, ESP-NOW is possible again. Wali and Austin decide |
| `gate` and `backend` | How the laptop tells the gate to open and close, and how the gate reports limit switches, dock sensor and faults | **Open.** Depends on whether the gate ESP32 talks to the laptop over USB serial or Wi-Fi |
| `backend` and `frontend` | HTTP API and event stream | Drafted by Wali: start, return, emergency stop, reset, state, live events |
| `backend` and `ai` | `analyze(video_path)` returns a status and a list of findings | Drafted by Wali. Until the model exists the server reports "model not ready" and zero findings |
| everyone and `pitch` | What may be claimed on stage | See "What is real" in `docs/PROJECT_BRIEF.md`. Aayan asks the owner before adding a claim |

## Current state, honestly

As of 2026-11-11:

- Wali has the laptop side working with simulated hardware (state machine,
  radio frame code, API, upload with resume and hash check, a control panel,
  92 passing tests). That code is on Wali's laptop, not in this repo yet.
- Nothing has been tested on real hardware by Wali: no board flashed, no
  radio link measured.
- Austin's motor, model and printing status: in progress. Austin, please
  replace this line with specifics.
- The defect model does not exist yet.
- The drone flight is not automated by any of this.

## How to ask someone for something

From a terminal in this repo:

```powershell
# Ask a role's worker to do a bounded coding task in its folder
python agentboard.py post --from human --to gate --title "Limit switch debounce" --message "Debounce both limit switches at 20 ms in gate/" --accept "switch bounce shorter than 20 ms does not stop the motor"

# Ask a person to do or decide something. No worker runs.
python agentboard.py post --from human --to human --kind request --title "For Austin: gate ESP32 link" --message "USB serial or Wi-Fi between the gate ESP32 and the laptop? Wali needs this to write the driver."
```

Rules that keep the board readable:

1. Start the title of a `human` request with the person's name: "For Austin: ...".
2. One task, one folder. If a job touches two roles, post two tasks and link
   them with `--depends`.
3. Say how you will know it is done (`--accept`).
4. Physical work is always a `human` request: wiring, flashing a board,
   running a motor, printing, measuring. A worker cannot do it and must not
   pretend to.
5. A change to an agreement in "Where the pieces meet" goes to both owners.
6. Reports say what was tested and how. "Works" without a test output is not
   a report.

From a phone, comment on Issue #1:

```text
AGENT_TO: gate
AGENT_FROM: human
KIND: task

Debounce both limit switches at 20 ms in gate/.
```

## Starting your listener

```powershell
git clone https://github.com/Ausnguy/AgentBoard.git ; cd AgentBoard
python agentboard.py setup --as <your role> --start
```

One listener per role. If you own two roles, run two terminals.
`python agentboard.py status --as <role>` shows what is queued or blocked.

## Open work

Good places for Sakina to start, in rough order of how much they unblock. Aayan owns 1 and shares 3 and 4:

1. Operator website (`frontend`, Aayan): the Start button, the live mission state,
   and the screen where a person accepts or rejects each finding.
2. Review and ticket flow: what a repair ticket contains, what happens to a
   rejected finding.
3. Demo footage: a short patrol video for the upload and analysis steps.
4. Test plan for the demo day: who brings what, power-on order, fallbacks.

## Open questions

| Question | Who answers |
|---|---|
| What does the relay on the drone switch, and what powers the tag ESP32 in flight? | Wali |
| Tag to gate radio: ESP-NOW or BLE? | Wali and Austin |
| Gate ESP32 to laptop: USB serial or Wi-Fi? | Austin |
| Which motor, driver, limit switches and dock sensor? What supply voltage? | Austin |
| Is there a physical emergency stop that cuts motor power by itself? | Austin |
| Where does the model run, and how long does one video take? | Austin |
| Which part does Sakina take? | Sakina |
| GitHub login for Sakina | Sakina |
| Where does Wali's existing code go: this repo's `backend/` and `integration/`, or its own repo? | Wali and Austin |
