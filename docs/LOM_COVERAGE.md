# Live Object Model coverage map

What the Ableton Live API (LOM) offers vs. what this bridge exposes, so feature
requests can be answered from a menu instead of re-derived one at a time.

Source: Live 11.0.0 introspected API (nsuspray/Live_API_Doc) cross-checked
against this repo's remote script + MCP server (30 tools as of 2026-07-02).
Live 12 is a near-superset; 12-specific notes are marked. The bridge targets
Live 12 (`create_audio_clip` already requires 12.0.5+).

Legend:
- ✅ wired — exposed by the bridge today
- 🟢 easy win — LOM supports it directly; wiring is mechanical
- 🟡 doable, needs design — LOM supports it but the tool shape isn't obvious
- ⚪ available, probably not worth a tool — niche; note it exists, skip until needed
- 🔴 wall — the LOM does not expose it; no remote-script code can add it

---

## Transport & Song

| Capability | Status | LOM surface |
|---|---|---|
| Play / stop / tempo / playhead position | ✅ | `start_playing`, `stop_playing`, `tempo`, `current_song_time` |
| Session/loop info in `get_session_info` | ✅ | `is_playing`, `song_length`, `loop`, `loop_start`, `loop_length` |
| Undo / redo | 🟢 | `song.undo()`, `redo()`, `can_undo/can_redo` |
| Undo step grouping (wrap a multi-command agent edit into ONE undo) | 🟢 | `begin_undo_step()` / `end_undo_step()` |
| Metronome, count-in | 🟢 | `metronome`, `count_in_duration` |
| Loop region set | 🟢 | `loop`, `loop_start`, `loop_length` (writable) |
| Time signature | 🟢 | `signature_numerator/denominator` (writable) |
| Record modes | 🟢 | `record_mode`, `session_record`, `trigger_session_record()`, `arrangement_overdub` |
| Capture MIDI | 🟢 | `can_capture_midi`, `capture_midi()` |
| Relative jumps / scrub | ⚪ | `jump_by()`, `scrub_by()`, `play_selection()` |
| Root note / scale | ⚪ | `root_note`, `scale_name`, `scale_intervals` (Live 12: full tuning systems) |
| Groove pool | ⚪ | `song.groove_pool.grooves`, `groove_amount`, `clip.groove` |
| Song-attached agent scratchpad (persists inside the .als) | ⚪ | `song.get_data(key, default)` / `set_data(key, value)` — also on Track/Clip |

## Scenes — entirely unwired

| Capability | Status | LOM surface |
|---|---|---|
| List / fire / stop scenes | 🟢 | `song.scenes`, `scene.fire()`, `stop_all_clips()` |
| Create / delete / duplicate scenes | 🟢 | `create_scene()`, `delete_scene()`, `duplicate_scene()` |
| Scene name / color / tempo | 🟢 | `scene.name`, `color`, `tempo` |
| Capture playing clips as scene | 🟢 | `capture_and_insert_scene()` |

## Arrangement markers (locators) — entirely unwired

| Capability | Status | LOM surface |
|---|---|---|
| List / create / delete / jump to locators | 🟢 | `song.cue_points`, `set_or_delete_cue()`, `cue_point.jump()`, `cue_point.name` (writable) |

Locators are the natural way for an agent to mark song sections
(intro/verse/chorus) in the arrangement.

## Tracks

| Capability | Status | LOM surface |
|---|---|---|
| Create MIDI/audio track, delete, rename | ✅ | |
| Mixer: volume/pan/mute/solo/arm (+ master/return via `track_type`) | ✅ | |
| Sends | ✅ | `mixer_device.sends` |
| **Real-time meters (incl. master)** | 🟢 | `output_meter_left/right/level`, `input_meter_*` — readable on every track and the master. Poll during playback → gain staging, clip detection, level balance. No plugin needed. |
| Duplicate track | 🟢 | `duplicate_track()` |
| Return track create/delete | 🟢 | `create_return_track()`, `delete_return_track()` |
| Track color | 🟢 | `color`, `color_index` |
| Delete / move device on a track | 🟢 | `track.delete_device()`, `song.move_device()` |
| Stop all clips on track | 🟢 | `track.stop_all_clips()` |
| Monitoring state | 🟢 | `current_monitoring_state` |
| I/O routing (incl. **Resampling** input) | 🟡 | `input_routing_type`, `available_input_routing_types`, `output_routing_*` — enables the resampling bounce (below) |
| Group tracks (read structure, fold) | ⚪ | `group_track`, `is_grouped`, `is_foldable`, `fold_state` |
| Crossfader / cue volume | ⚪ | `mixer_device.crossfade_assign`, `crossfader`, `cue_volume` |
| Track freeze / flatten | 🔴 | `is_frozen`/`can_be_frozen` are read-only; no freeze method exists |

## Clips (Session)

| Capability | Status | LOM surface |
|---|---|---|
| Create MIDI clip, import audio file, delete, rename, fire, stop | ✅ | |
| Read notes / add notes | ✅ | `get_notes_extended`, `add_new_notes` |
| Clip loop points & looping | 🟢 | `loop_start`, `loop_end`, `looping`, `position`, `start_marker`, `end_marker` |
| Quantize clip | 🟢 | `clip.quantize(grid, amount)`, `quantize_pitch()` |
| Crop / duplicate loop / duplicate region | 🟢 | `crop()`, `duplicate_loop()`, `duplicate_region()` |
| Remove notes in range | 🟢 | `remove_notes_extended(from_pitch, pitch_span, from_time, time_span)` |
| **Edit notes in place by id** | 🟢 | `get_notes_by_id()`, `apply_note_modifications()` — revise a melody without delete+re-add |
| Per-note probability / velocity deviation on write | 🟢 | `MidiNoteSpecification(probability=…, velocity_deviation=…)` — read side already wired |
| Clip mute / color / gain / transpose (audio) | 🟢 | `muted`, `color`, `gain`, `pitch_coarse`, `pitch_fine` |
| Warp control (audio) | ⚪ | `warping`, `warp_mode`, `warp_markers` |
| Recorded/audio clip file path | 🟢 | `clip.file_path` — key piece of the bounce workflow |
| Duplicate session clip slot | ⚪ | `clip_slot.duplicate_clip_to()` |

## Clips (Arrangement)

| Capability | Status | LOM surface |
|---|---|---|
| List arrangement clips per track | ✅ | `track.arrangement_clips` |
| Copy session clip → arrangement | ✅ | `duplicate_clip_to_arrangement()` |
| Edit/delete arrangement clips, arrangement automation | 🟡 | Arrangement clips are full `Clip` objects (notes, envelopes, `delete_clip` via track) — needs an addressing scheme (arrangement clips have no slot index; identify by start_time) |
| Create audio clip directly in arrangement from file | 🔴 | `create_audio_clip` exists only on `ClipSlot` (session); route: session import → duplicate to arrangement |
| Consolidate | 🔴 | not exposed |

## Automation

| Capability | Status | LOM surface |
|---|---|---|
| Write step automation into session clip envelopes | ✅ | `automation_envelope()` + `insert_step()` |
| Read envelope value at time | 🟢 | `envelope.value_at_time()` |
| Clear one/all envelopes | 🟢 | `clear_envelope(param)`, `clear_all_envelopes()` |
| Show envelope in UI (user visibility) | ⚪ | `clip.view.show_envelope()`, `select_envelope_parameter()` |
| Arrangement automation | 🟡 | same envelope API on arrangement clips — blocked on arrangement clip addressing (above) |
| Re-enable overridden automation | 🟢 | `song.re_enable_automation()`, `parameter.re_enable_automation()` |
| Smooth ramps (non-step) | 🔴 | only `insert_step`; approximate curves with dense steps |

## Devices & parameters

| Capability | Status | LOM surface |
|---|---|---|
| List devices / parameters, set parameter (incl. master/return) | ✅ | |
| Load instrument/effect from browser (incl. master/return) | ✅ | `browser.load_item()` |
| Device on/off | 🟢 | parameter "Device On" (`parameters[0]`) — settable today via `set_device_parameter`; worth documenting or a convenience tool |
| Delete device | 🟢 | `track.delete_device(index)` |
| **VST/AU preset switching** | 🟢 | `PluginDevice.presets`, `selected_preset_index` (many VSTs expose an empty list — degrade gracefully) |
| Rack macros | 🟢 | `RackDevice`: `macros_mapped`, `add_macro()`, `randomize_macros()`, variations (`store_variation()`, `recall_selected_variation()`) |
| Rack/drum-rack chains | 🟡 | `chains`, `drum_pads`, per-chain mixer — useful for drum kit editing; addressing needs design |
| Specialized device classes | ⚪ | `SimplerDevice` (sample, slicing, warp), `Eq8Device`, `WavetableDevice` (mod matrix), `CompressorDevice` (sidechain routing), `HybridReverbDevice`, `MaxDevice` |
| Loading .fxp/.adv preset files onto an existing device | 🔴 | not exposed; only browser `load_item` (replaces device) or `presets` index |
| Reading a meter plugin's *measurements* (e.g. Youlean LUFS) | 🔴 | plugin params expose settings, not readings — use track meters or bounce+offline analysis |

## Browser

| Capability | Status | LOM surface |
|---|---|---|
| Browse tree / items at path / load by URI | ✅ | includes plugins, user library, packs, samples roots |
| **Audition samples without loading** | 🟢 | `browser.preview_item()`, `stop_preview()` |
| Hotswap | ⚪ | `hotswap_target`, `relation_to_hotswap_target()` |

## Selection & UI (agent↔user shared attention)

| Capability | Status | LOM surface |
|---|---|---|
| Switch Session/Arranger view | ✅ | `application.view.show_view` |
| Select track / scene / clip for the user to look at | 🟢 | `song.view.selected_track`, `selected_scene`, `highlighted_clip_slot`, `detail_clip` |
| Select a device (show its panel) | 🟢 | `song.view.select_device()` |
| Zoom/scroll views | ⚪ | `application.view.zoom_view()`, `scroll_view()` |

## Rendering & audio output — the walls

| Capability | Status | Notes |
|---|---|---|
| Export/render audio (File → Export) | 🔴 | No method anywhere in the LOM. Options: user bounces manually, agent drives the export dialog via UI automation, or the resampling bounce below. |
| **Resampling bounce (in-API render substitute)** | 🟡 | All pieces exist: `create_audio_track()` → set `input_routing_type` to Resampling → `arm` → `trigger_session_record()`/`record_mode` while playing → stop → `clip.file_path` → analyze offline (e.g. `ffmpeg ebur128` for LUFS). Real-time (a 3-min song takes 3 min). Needs design: timed stop, record length, cleanup. |
| MIDI CC / pitch-bend / per-note expression in clips | 🔴 | `MidiNote` carries pitch/time/duration/velocity/probability only; MIDI CC clip envelopes are not addressable. Routes: M4L device, or virtual MIDI port + live recording. |
| Import .mid files | 🔴 | No API. Parse the .mid externally and feed `add_notes_to_clip` (loses CC — see above). |
| Waveform/audio analysis of clips | 🔴 | No sample-data access (`Sample` exposes markers/slices, not PCM). Analyze the file at `clip.file_path` externally. |

## Suggested next waves (opinionated)

1. **`get_track_meters`** — one read-only tool returning output meters for all
   tracks + master. Cheapest possible feedback loop; partially obsoletes
   loading a metering plugin at all.
2. **Scene + locator tools** — section-level workflow (fire scene, mark chorus).
3. **Clip editing pack** — loop points, quantize, remove-notes-range,
   apply_note_modifications, clip gain/transpose.
4. **Undo tools** — `undo`/`redo` + wrap multi-step agent edits in one undo step.
5. **Resampling bounce** — closes the render loop without UI automation.
6. **Browser preview** — audition-before-load for sample picking.

Anything marked ⚪ we skip until a real workflow demands it — every tool costs
agent context.
