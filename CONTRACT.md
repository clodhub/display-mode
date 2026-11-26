# Display mode

## Purpose and facilities
Some applications run on a shared screen as well as on each person's device: a television showing the table to the room, a tablet propped up as a scoreboard. A shared screen is seen by everyone, so it must never show one person's private information, never act on anyone's behalf and never turn on a camera or microphone. `display-mode` gives the application one switch for that. A session is either in `player` mode (a person's own device) or `board` mode (a shared screen). The application asks the component before it acts, captures media or shows private data, and the answer is no on a board.

On the web, `createMode(initial, eventName)` holds the current mode; publishing a new mode records it on the page (`data-mode` on the document element, so styles can hide private parts) and then sends a browser event that other components listen for, so video stops the moment a screen becomes a board. `displayCapabilities(board)` answers `canAct`, `canCapture` and `canSeePrivate`, all false on a board. Apple (Swift) and Android (Kotlin) have the same three answers as `DisplayPolicy`.

This component decides what the device itself does. It does not sign anyone in and does not hide data on the server: the server must still send a board only public information.

Facilities: `display-policy` (Keeps a shared screen, such as a television showing the table, from acting, using the camera or microphone, or showing private information).

## User inputs
There are no installer settings. The application gives two values in code when it creates the mode (or through the optional `display-mode-configuration` port):

- `initial`: the mode a session starts in, `player` or `board`. Most applications start in `player` and switch to `board` when the person chooses "use this screen as the table".
- `eventName`: the name of the browser event sent on each change, any non-empty text such as `app:mode`. Every component that reacts to the mode (video-calls, windows-desktop) must be given the same name.

Remembering the chosen mode between visits is up to the application.

## Artefacts and dependencies
Every implementation file maps to `components/display-mode/` with its `implementation/` prefix removed: `web/mode.ts` and `web/index.ts`, `apple/Sources/DisplayPolicy.swift` and `android/src/main/java/com/clodhub/display/DisplayPolicy.kt`. SOURCE.md records where each file came from and SOURCE.json pins the source hashes. No runtime dependencies. `tools.lock.json` pins the check tools (Node 26.5, Python 3.12, Swift 6.4, Kotlin 2.2, Java 26.0).

## Provided interfaces
`interfaces/display-policy.json` is an OpenRPC 1.3.2 document for `com.clodhub/display-policy` version 1.0.0: `createMode`, the returned handle's `ModeHandle.current` and `ModeHandle.publish`, and `displayCapabilities` (web), and `DisplayPolicy.canAct`, `canCapture` and `canSeePrivate` (Swift and Kotlin), with types as JSON Schema 2020-12.

## Required interfaces
`display-mode-configuration` (`com.clodhub/display-mode-configuration` version 1.0.0) is optional: the two values can be passed in code, so a design may leave it unconnected. `interfaces/required-display-mode-configuration.json` is the JSON Schema 2020-12 document of its fields, `initial` and `eventName`.

## Integration
Create the mode once when the application starts and publish the starting mode before anything can capture media:

```ts
import { createMode, displayCapabilities } from "./components/display-mode/web/index";
const mode = createMode("player", "app:mode");
mode.publish(mode.current());
const board = () => mode.current() === "board";
if (displayCapabilities(board()).canAct) sendMove(move);
```

Call `displayCapabilities` (or `DisplayPolicy` on Apple and Android) at every point where the device would act, open the camera or microphone, or show private data. Give video-calls and windows-desktop the same event name so they react to a switch. The server must still decide for itself what a board may see. `examples/consumer.ts` is a complete small example.

## Visual definition and connected examples
`visual/definition.json` is the component block. `visual/examples/connected.json` wires an illustrative configuration provider and consumer. `visual/output/connected.svg` and `connected.png` are the rendered diagram and `visual/output/description.md` its text equivalent.

## Infrastructure
None. It runs inside the application on each device; no accounts, services or costs.

## Verification
`node --test tests/mode.test.mjs` (required) checks, against `tests/fixtures/modes.json`, that a board may not act, capture or see private data and a player may; and that publishing records the mode on the page and in `current()` before the event is sent, that the event carries the new mode, and that two modes created side by side do not affect each other. `python3 scripts/check-native.py apple` and `python3 scripts/check-native.py android` compile the Swift and Kotlin `DisplayPolicy` and check the same three answers.

## Scenarios and presentation
Four friends play cards on their phones and put the table on the living-room television. The television's session switches to `board`: it shows the shared cards and the pot, but no one's own cards, its buttons do nothing, and if video was on it stops at once. A phone stays in `player` mode: its owner sees their cards and can bet. A person who turns their phone into a board for a moment loses those rights until they switch back.

## Updates and removal
Updates replace the component's files; it stores nothing. The mode names `player` and `board`, the page marker and the event's `{mode}` detail are part of the contract, so changing any of them is a new major version. Removing the component leaves any mode the application remembered in its own storage.
