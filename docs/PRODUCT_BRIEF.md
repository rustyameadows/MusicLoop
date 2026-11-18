# MusicLoop: Product Brief

**Status:** Initial concept / MVP definition  
**Platform:** iPhone (native iOS)  
**Working name:** MusicLoop  
**Updated:** November 18, 2026

## 1. Overview

MusicLoop is a focused Apple Music companion app for repeatedly listening to a precise segment of a song. Choose a track from Apple Music, mark an **A** start point and **B** end point, and loop that passage. Adjust the boundaries with fine-grained controls, see how many times the passage has repeated, and return to useful loops later.

Think of it as a tiny A/B loop player, not a full DJ application or general music player.

**Core promise:** Select a song, isolate a moment, and stay in that moment.

## 2. Problem

Apple Music can repeat an entire song but does not offer a dedicated, precise, easy-to-use passage-looping interface. Repeatedly seeking to a passage by hand is inconvenient, especially when studying a part, listening closely, rehearsing, or simply enjoying a favorite moment.

## 3. Primary experience

1. **Authorize Apple Music.** Request the system's MusicKit/media-library permission and check subscription/playback eligibility. Do not ask the user to type Apple credentials into the app.
2. **Select a song.** Browse/search Apple Music and, where permitted, the personal library. Use Apple's native Music Picker when available for the deployment target.
3. **Play and mark.** Play the track and set A and B at the current playhead, or drag each marker on a timeline.
4. **Fine-tune.** Nudge each marker forward/backward with fine time increments and inspect the displayed timestamps.
5. **Loop.** Repeat only the A–B passage until paused or disabled.
6. **Count.** Display completed repetitions, current passage duration, and cumulative loop listening time.
7. **Resume.** Optionally save the song and its loop boundaries as a named preset for later use.

## 4. Design principles

- **One purpose, one screen:** The selected track and loop controls dominate the interface.
- **Fast to a loop:** Minimal friction from launching the app to hearing a repeated passage.
- **Precise controls, realistic guarantees:** Offer fine adjustments, but do not imply sample-accurate or gapless streaming until verified.
- **Visible state:** A/B timestamps, current playhead, loop enabled/disabled, and count are always understandable.
- **Native behavior:** Respect audio routes, headphone controls, interruptions, lock-screen playback, and the Apple Music account already authorized on the device.
- **Private by default:** Local-only settings and counters in the initial version; no account, server, or analytics dependency.

## 5. MVP scope

### Must have

- Native iPhone interface built in SwiftUI.
- MusicKit authorization, subscription/eligibility check, and clear error/empty states.
- Track selection from available Apple Music content.
- Track artwork, title, artist, duration, and current position.
- Play/pause, manual seeking, and a passage timeline.
- Set A and B from the playhead; drag markers; fine nudge controls.
- Time labels with hundredths-of-a-second display as a UI precision target (not an audio accuracy guarantee).
- Enable/disable A–B repeat; enforce `0 <= A < B <= duration`.
- Completed-loop counter, elapsed time spent looping, and counter reset.
- Resume/interrupt/background-audio behavior tested on real hardware.
- Handle unavailable tracks, lost authorization, failed seeks, and subscription changes gracefully.

### Nice to have (after the playback spike)

- Saved named loop presets, persisted locally.
- Reopen the most recent song and loop.
- Configurable target number of loops with auto-stop.
- Shortcuts for shifting an entire loop window forward/backward.
- Support for several loops within one song.

### Explicitly out of scope

- Mixing multiple tracks, beat matching, pitch/tempo manipulation, effects, or DJ decks.
- Recording, exporting, downloading, or extracting Apple Music audio.
- Cloud accounts, sync, sharing, social features, or a backend.
- Audio waveform generation from protected Apple Music streams.
- Android, macOS, or iPad-specific interfaces for the first release.

## 6. Proposed main screen

- **Header:** Album artwork, song/artist, and an unobtrusive button to choose another track.
- **Timeline:** Full-track scrubbing bar with A/B markers and a highlighted active region. Initially use a stylized bar, **not** a real waveform.
- **Loop editor:** A and B timestamps with nudge controls (e.g., ±0.01 s / ±0.1 s / ±1 s) and “set from playhead” actions.
- **Transport:** Large play/pause and loop toggle; current playhead time.
- **Stats:** Completed loops, segment duration, and cumulative listening time.
- **Optional preset action:** Save/rename current loop.

Do not add a conventional library, playlists, or secondary navigation until the single-song flow works well.

## 7. Platform and architecture

- **UI:** SwiftUI.
- **Music account, catalog, and playback:** MusicKit. Prefer `ApplicationMusicPlayer` to keep the app's queue separate from the Music app; enable the background audio mode when appropriate.
- **Alternative playback path for evaluation:** MediaPlayer's `MPMusicPlayerPlayParametersQueueDescriptor`, which exposes per-item start/end times, together with repeat-one. Evaluate actual catalog support and behavior instead of assuming this produces a seamless passage loop.
- **Local persistence:** SwiftData or lightweight local storage for optional presets and recent session state.
- **No backend:** Not required for the MVP.

Apple documents that MusicKit's `MusicPlayer.playbackTime` is settable, moving the playhead to a requested position. A first implementation could observe time and seek back to A when B is reached. But polling/observer cadence, streaming latency, seeking accuracy, and background execution may introduce gaps, drift, or missed boundaries. Those are product-level constraints, not merely visual polish.

The legacy MediaPlayer queue descriptors also expose start/end times for a queued item. Test them with Apple Music catalog tracks and repeat-one before committing to a playback engine. Do not promise zero-gap playback or 10 ms accuracy based solely on the API surface.

## 8. Critical feasibility spike (build this first)

Implement a minimal on-device test before polishing UI:

1. Authorize Apple Music and pick one full-length catalog song.
2. Start playback at an arbitrary A point.
3. Loop a 5–15 second A–B region using MusicKit `playbackTime` seeking.
4. Independently test a MediaPlayer play-parameters queue descriptor with defined start/end times and repeat-one, if compatible with the chosen track.
5. Compare audible gaps, perceived precision, count reliability, background/locked-screen behavior, and Bluetooth/AirPlay behavior.
6. Test short passages (~1–2 seconds), longer passages (~30 seconds), repeated seeks, network interruptions, and headphone-control interruptions.
7. Document what works and choose the most stable supported path.

**Decision gate:** If reliable passage repetition works with minor but acceptable gaps, ship as an approximate A/B looper and describe its limits clearly. If the use case requires gapless, sample-accurate loops and Apple Music playback cannot provide them, revisit the product promise. Do not bypass Apple Music content protections.

## 9. MVP acceptance criteria

- A subscribed, authorized user can select and play a supported Apple Music track without entering Apple credentials into MusicLoop.
- A and B can be set by playhead and changed independently with both dragging and nudge actions.
- The displayed range is always valid; changing A/B while playing produces predictable behavior.
- With looping on, playback returns toward A each time it reaches B. The app counts **completed** passages, not seeks or play-button presses.
- Pausing does not falsely increment the count; resetting the count does not modify loop boundaries.
- The app handles lock/unlock, audio interruption, and app backgrounding without silently misreporting the number of completed loops.
- Any audio timing limitations observed in the prototype are reflected in the UI and release description.
- All core actions work without an app-specific account or server.

## 10. Suggested implementation sequence

1. **Playback feasibility prototype:** Test MusicKit vs. MediaPlayer approaches on a real iPhone and record findings.
2. **Single-song experience:** Authorization, picker, artwork, transport, A/B editing, continuous repetition.
3. **Reliability:** Background audio, interruptions, routing, counter correctness, edge cases.
4. **Polish:** Accessibility, tactile nudges, saved loops, launch/resume behavior.
5. **Release evaluation:** Apple Music licensing/review compliance, supported iOS version, device testing.

## 11. Open questions

- How much gap between repeats is acceptable for the intended experience?
- Does the chosen playback path support *both* full Apple Music catalog tracks and personal-library tracks reliably?
- What is the actual resolution of seeks and repeat boundaries on-device?
- Should loop counts reset per session, persist per preset, or both?
- Is saving multiple loops an MVP requirement or a follow-up?
- What minimum iOS version makes the latest Music Picker available, and what fallback is needed?

## 12. Official references

- [Apple: Integrate MusicKit into your app (WWDC26)](https://developer.apple.com/videos/play/wwdc2026/254/)
- [Apple: MusicKit](https://developer.apple.com/documentation/musickit)
- [Apple: MusicPlayer.playbackTime](https://developer.apple.com/documentation/musickit/musicplayer/playbacktime)
- [Apple: ApplicationMusicPlayer](https://developer.apple.com/documentation/musickit/applicationmusicplayer)
- [Apple: MPMusicPlayerPlayParametersQueueDescriptor](https://developer.apple.com/documentation/mediaplayer/mpmusicplayerplayparametersqueuedescriptor)

---

This brief defines a product direction, not a verified claim of seamless or millisecond-accurate Apple Music looping. The playback spike is the first development milestone.
