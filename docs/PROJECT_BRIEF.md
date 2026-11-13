# TALON in plain words

For anyone who joins late, and for whoever presents. Read this first, then
`docs/TEAM.md` for who owns what.

## The problem

Power lines are inspected by people driving or climbing along them. It is
slow, and faults get found late.

## What TALON does

A drone lives in a small dock next to the line. Nobody has to be there.

1. A manager clicks **Start autonomous inspection** on a website.
2. The dock opens its gate. The drone takes off.
3. A sensor in the dock sees the drone is gone, and the gate closes.
4. The drone flies along the line and records video.
5. On the way back, a small radio tag on the drone is heard by the dock. When
   the signal is strong enough, the dock knows the drone is close and opens
   the gate.
6. The sensor sees the drone is fully inside, and the gate closes.
7. The video is sent to a server. The server checks a fingerprint of the file
   (SHA-256) so a damaged or incomplete video is never accepted.
8. A defect model looks through the video and marks possible faults.
9. A person reviews each one. Accepted findings become repair tickets.
   Rejected ones are kept as corrections to train the model.

The long-term goal is inspections on a schedule with nobody on site.

## The parts and their owners

| Part | What it is | Owner |
|---|---|---|
| Drone tag | A small ESP32 board with a relay, attached to the drone | Wali |
| Dock gate | A motor, driven by another ESP32, that opens and closes the dock. 3D-printed parts | Austin |
| Server | A program on a laptop that takes the Start command, tracks the mission, and receives the video | Wali |
| Defect model | The AI that finds faults in the video | Austin |
| Website | What the manager and the reviewer see | Aayan |
| Pitch | The presentation and demo narration | Aayan |

## Words you will hear

| Word | Meaning |
|---|---|
| ESP32 | A cheap microcontroller board with Wi-Fi and Bluetooth built in |
| Tag | The ESP32 on the drone. It sends a radio signal. It does not fly the drone |
| Relay | An electrically controlled switch |
| Dock, base | The box the drone lives in |
| Gate | The dock's door |
| RSSI | Radio signal strength. Stronger means closer. It is a rough "near or far", not a distance in metres |
| ESP-NOW, BLE | Two short-range radio methods the ESP32 can use. The team has not picked one yet |
| Limit switch | A switch the gate presses when it is fully open or fully closed |
| Dock sensor | Tells whether the drone is inside the dock |
| SHA-256 | A fingerprint of a file. Same file, same fingerprint |
| Finding | Something the model thinks is a fault |
| Ticket | A repair job created from an accepted finding |
| AgentBoard | This repo's tool: a GitHub Issue where teammates and their coding assistants hand each other tasks |

## What is real, as of 2026-11-11

Ask the owner before saying anything stronger than this table.

| Thing | State |
|---|---|
| Server, mission logic, video upload with fingerprint check | Built and tested on a laptop with simulated hardware |
| A control panel with a Start button | Built, simple, runs on the laptop |
| Radio between tag and dock | Not tested on real boards. Method not chosen |
| "Near or far" from signal strength | Works on simulated signal. Not measured in a real room |
| Gate motor | Being wired by Austin |
| Dock sensor, limit switches, emergency stop | Not confirmed |
| Defect model | In training. No results yet |
| Review and repair tickets | Storage exists on the server. No screen for it yet |
| Drone flying itself | Not part of what has been built. The demo video may be recorded in advance |
| Scheduled, unattended inspections | The goal, not built |

## Things not to say on stage

- That the tag measures distance. It estimates near or far from signal strength.
- That the model detects faults, until Austin has numbers.
- That sample findings are real. If the demo shows sample data, it is labelled
  as sample data on screen; say so.
- That the drone flies the route on its own, unless that has been shown.
- That anything works on hardware if it has only been simulated.

## Questions for the presenter to get answered

1. What exactly will be shown live, and what is a recording?
2. Who presses what, in which order, and what is the fallback if a part fails?
3. Which one number or picture proves the idea? (For example: time from Start
   to verified video, or one real finding once the model has one.)
4. Who is the customer, and what does an inspection cost them today?

Post these on the board as `human` requests so the answers are written down.
`docs/TEAM.md` shows how.
