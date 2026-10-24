---
paths:
  - "src/cactus/field.py"
  - "src/cactus/sky.py"
  - "src/cactus/garden.py"
  - "src/cactus/fieldproc.py"
  - "tests/test_field.py"
  - "tests/test_sky.py"
  - "tests/test_puffs.py"
  - "tests/test_garden.py"
---

# Field, sky and garden invariants

Moved out of CLAUDE.md so they load only when working on the field.

- Sky config save/recall slots (v6g): nine files beside `sky.toml`,
  `sky-slot-N.toml`, sparse-dumped the same way `sky.toml` itself is. The `T`
  overlay's bare digit `1`-`9` recalls a slot (applies live, rewrites
  `sky.toml`, flashes empty when unset); `S` arms, and the next digit opens
  a name prompt (v6h) rather than saving at once — `enter` saves with the
  typed name (or none), `escape` cancels. A slot's name is a plain
  top-level `name` key in its TOML file (`overlay()` ignores unknown
  top-level keys, so it never affects loading). The overlay's saved-skies
  grid sits at the very top, right under the "tuning" title, one `N name`
  cell per slot (`—` empty, `(unnamed)` filled but nameless) — the ~75 key
  rows below it push the grid off-screen otherwise.
- The garden (the field's landed cactus pile) is a shared, persistent file
  (`garden.py`), `garden.json` beside the database — every TUI on the same
  database reads and writes the same one, and it survives a restart.
  `World` only counts `landed_since_save`; it never touches the file
  itself. A landing saves on the next `_field_tick`; every `SKY_RELOAD_SECONDS`
  poll (the same clock the sky config reload uses) also stats the file and
  reloads it if another process's write is newer, flashing "garden updated".
  `~` (or settings `7`) hides/shows the field strip (`tui_settings["field"]`,
  persisted, default off);
  backtick drops a seed while shown, or while hidden flips `pile_only`
  (persisted) to show the pile alone without the sky/birds/seeds. A hidden
  garden is paused: nothing advances or polls, and a seed dropped meanwhile
  is owed (`field.owed`, columns) and falls on show, the field resuming
  where it stopped; `pile_only` is shown, so it keeps running. Quitting
  hidden queues the owed seeds in `garden-pending`. `cactus
  garden` prints the file's path, cell count, and drop count (or "empty");
  `--clear` removes it (`nothing to clear` if it was already gone).
- The field runs in one of two places (`fieldproc.py`), picked by
  `CactusApp(field_mode=...)`. `inline` (the default, so every `run_test`
  test, and `run_tui` under `CACTUS_FIELD=inline`): `InlineField` keeps the
  World in the TUI process (`app.world`), advanced by `_field_timer` at
  `1 / fps`, and the app polls sky.toml and garden.json itself
  (`SkyWatch`/`GardenSync`). `process` (`run_tui`'s default): `ProcessField`
  spawns a daemon child that owns the World, paces at `1 / fps` on real
  elapsed time, saves garden.json on landing, polls sky.toml and
  garden.json every `SKY_RELOAD_SECONDS`, and sends flashes and reloaded
  configs back. There `app.world` is `None`, no timer runs, and the T
  overlay reads and mutates the parent's own `SkyConfig` copy
  (`app.sky_config`), dumped first and then pushed with
  `apply_config(cfg, mtime=...)` so the child's poll skips that write.
  Frames are `("frame", seq, n_rows, pile_rows, {y: (plain, spans)})`,
  changed rows only (every row after a resize or a re-show). The child
  holds at most `CREDIT` (2) unacked frames and merges changed rows while
  it waits; the parent reads on `loop.add_reader`, drains everything
  ready, acks each frame, and paints only the merged newest state.
  Commands send only on change: drop, resize, pile_only, visible (hidden:
  the child saves any unsaved landing, then blocks on the pipe, advancing
  and polling nothing; shown, it resumes with no fast-forward, resends every
  row and polls at once; the parent holds drops made while hidden and sends
  them on show), fps, config, reseed, stop. `run_tui` calls
  `resource_tracker.ensure_running()` before `App.run()` — spawn after
  Textual swaps the std streams fails with `bad value(s) in fds_to_keep`;
  a spawn failure falls back to inline with a flash. `on_unmount` and
  `atexit` stop the child. `FieldView` is a Line-API widget: one cached
  `Strip` per row, `refresh(Region)` for changed rows only, cell-identical
  to the old `Static.update` path bar Textual's mouse-offset meta.
  `_rebuild_card` only re-fits the field (`_fit_field`); it never renders
  a frame.
- A falling clump has hidden `charge` (no UI text): +1 and `accrete_count` members on each
  clear-sky-to-cloud entry (`in_cloud`'s False->True edge, not per frame
  spent inside one), +1 per distinct bird it shares a terminal cell with.
  Landing bursts `charge` single-member clumps only if the clump touched at
  least one bird (`birds_hit`; cloud charge alone never bursts), spawned
  just above the pile top and sent sideways with a small downward `vy`
  (`EXPLODE_VY`), never up, so they skid off and land beside it; those are
  `bounty=False` so they can never collect and cascade. Every frame a clump
  is inside a cloud, each member calls `sky.scatter(px, py, ...)` once at
  its sky-pixel position: fluid pushes density outward (mass-conserving),
  puffs dent only the patch pixels the seed displaces, pushing that raw
  value outward into a rim (`_dent_patch`; the cloud itself never moves or
  ages, a `bands` lane dents the same way), texture is a no-op. Cloud
  presence is read off `World._sky_cells`, the glyph grid `render()` cached
  last frame — never a live query against the sky engine.
- `perspective` (v8) is a `SkyConfig` choices lever for the fluid engine
  only: `on` (default) is the v7 projection through `horizon`/`focal`/
  `z_far`; `off` is the v6 look, `Sky.render_cells` handing `downsample` the
  raw grids so each deck's bands draw flat across the sky at their own
  height. Texture and puffs never projected and ignore it.
- The `T` overlay shows only the keys the running engine reads
  (`sky.tuning_visible`/`tuning_fields_for`, v8): far/mid/near grids, shear
  and `perspective` under fluid, the projection levers only while
  `perspective == "on"`; `shear_base` alone under texture; the `cloud_*`
  levers under puffs; tone/haze/glyph thresholds, `fps`, `pile_style`,
  `stick_distance`, `seed_wind`, `sky_engine` always. `edge_*` and `flat_*`
  are never shown — the strokes they governed are gone: the sky draws
  braille only (a single dot on the fringe, the dither inside, the full
  cell at a core), the keys stay so an older sky.toml still loads. `_render_tuning` refilters on every
  redraw and keeps the cursor on the same key when it survives.
  The visible keys page as panels (`sky.TUNE_PANELS`, then `other` for any
  shared key listed nowhere, then far/mid/near), one on screen at a time
  under a strip naming every non-empty one: `tab`/`]` and `shift+tab`/`[`
  switch, and `j`/`k` walk across panel edges because `tuning_fields_for`
  returns rows panel-sorted. `#tuning-panel` sits in `#tuning-scroll`
  (`VerticalScroll`), scrolled after each redraw to keep the cursor in view.
- `cloud_style = "bands"` (v8, planetary layers) is the puffs engine's
  fourth style: `lanes * cloud_count` equal lanes down the whole sky, one
  full-width `_Puff` per lane (`lane=(index, y0, h)`), its noise lattice
  fitted to the width so it wraps with no seam, neighbouring lanes flowing
  opposite ways (odd east, even west). A dead band respawns in its own lane;
  a `cloud_count` (while `band_height` is 0), `band_gap`, or `band_flow`
  change re-cuts every lane.
  Lanes colour by thirds: top far, middle mid, bottom near. A lane never
  unfolds, recedes, or dies: its cutoff sits at rest (even lanes dense
  zones, odd lanes sparse belts) and only its age wraps.
  Its texture never repeats: it walks a chain of freshly baked patches a ->
  b -> c ... (`PuffSky._next_leg`), one cosine-eased leg per half `morph`
  scaled live by `band_evolve` (0 freezes, no re-bake), and every cloud
  draws at its fractional `x`, mixing neighbour columns so drift glides.
  `band_gap` is the empty share of each lane (0 = touching), `band_flow`
  deals directions (`alternate`, `same`, `random`). `band_height` > 0 fixes
  each lane at that many rows, laid top to bottom with leftover rows empty
  and `cloud_count` ignored (0 = count from `cloud_count`); `band_edge` is
  the share of a band's height that is a noisy fringe top and bottom, its
  depth wandering per column (`_lane_window`, periodic like the lattice).
  `band_belts` deals zones and belts: `alternate` (default), `dense` (all
  zones), `belts` (all belts); a belt also takes finer noise
  (`belt_scale_x`/`belt_octaves`), streaky between the smooth zones.
  All five show on the T page only under puffs/bands, and any change
  re-cuts every lane (`band_evolve` shows there too but re-cuts nothing).
  Off bands, `cloud_altitude` (0.5 even, lower toward the ground, higher
  toward the sky) draws each cloud's height as `u ** e`, `e = (1-b)/b`,
  inside its band and scales each band's count by its share of that
  distribution (`_altitude_weight`); 0.5 is today's placement exactly and
  any change re-bakes.
- Birds are levers on `SkyConfig` (v8), read off `World.sky.config` like
  `seed_wind`: `birds` picks the depth bands a flock may spawn in (`all`,
  `far`, `mid`, `near`, `far+mid`, `mid+near`, `none`; `World.bird_bands`),
  `bird_rate` the spawns per second, `bird_max` how many flocks fly at once
  (0 grounds them). `FLOCK_SPAWN_P`/`FLOCK_MAX_ALIVE` are only the defaults'
  documentation now. The glyph set per depth stays `field.DEPTH_GLYPHS`.
- Root toggles on `SkyConfig`: `show_sky`, `show_birds`, `show_cactus`
  (`on`/`off`), the `show` panel first on the T page. `show_sky` off skips
  `sky.advance` and draws no sky or ground lines (`World._sky_layer`;
  `_sky_cells` is `None`, so no seed reads a cloud); `show_birds` off clears
  every flock and spawns none; `show_cactus` off draws no falling seed and no
  pile on both render paths. `sky._tune_layer_off` hides an off layer's
  panels: sky hides engine/clouds/projection/wind/tone and the grids but
  keeps `fps` and `seed_wind`; birds hides `birds`; cactus hides seeds,
  pile and `seed_wind`.
- Pile age ramp on `SkyConfig`: `pile_shades` (3..64, default 32) and
  `pile_age_span` (10..1000 drops, default 100). `World._age_colour` blends
  `cactus_new` -> `cactus_mid` -> `cactus_old` piecewise-linearly over
  `[0, span]`, clamped past it, quantised to `shades` steps and cached per
  (anchors, shades, span, step) in `_age_tint`. The mid anchor is the ramp's
  halfway and sits at `CACTUS_NEW_MAX / CACTUS_MID_MAX` (34%) of the span, so
  `shades = 3` is exactly three flat colours. The static pile memo's key
  carries both levers, so a change redraws. Both always show on the T page
  (pile panel).
- Pile shape levers (v8) on `SkyConfig`, read off `World.sky.config` like
  `seed_wind`: `seed_mass` (`accrete`, a cloud pass adds a block via
  `_grow_clump`; `single`, the seed stays one block, charge still counts)
  and `pile_settle` (`drop`: `World._settle`, the arm adjustment — a landed
  shelf of two or more side-by-side blocks in the clump's lowest row that
  rests only on a diagonal drops exactly one row, and only when that row
  puts a shelf cell directly on the ground or a block and no target cell
  is taken — otherwise it lands as hit, so an overhang never slides down a
  pile's side; a lone block keeps its perch; `keep`: lands as it hit).
  `drop` also guarantees no floating piece: after every landing,
  `World._drop_floating` finds the landed cells' component (8-connected, x
  wrapped) and, when no cell of it is at `cy == 0`, drops it rigidly a row
  at a time until a cell reaches the ground or sits directly on another
  component, ages kept, to a fixed point. `World.drop_floaters()` sweeps the
  whole pile: `garden.load_into` calls it, and so does `apply_sky_config`
  on a switch to `drop`. `keep` never sweeps.
  Both always show on the T page. Under `pile_style = "dots"` a seed latches
  within `dot_latch` sub-cells (default 3.5, a dot's visible reach) of a
  pile cell at its height or below, then `World._snap` shifts the clump by
  the shortest integer vector (down first, then toward the pile) onto a
  `SUPPORT_OFFSETS` neighbour or the ground; blocks keep the 1-sub-cell rule.
- Seed wind: `SkyConfig.seed_wind` is in columns (0-30, default 3), the
  mean drift over a full fall at typical wind; gusts carry ~3x further.
  `World.seed_wind()` is world wind x `seed_wind / SEED_WIND_COLS_PER_UNIT`
  (0.55, measured over the 12 s fall at 120x30 with random gusts off:
  drift is linear in the multiplier), never a deck's
  `wind_scale` (puffs ignores wind, texture reads only `shear_base`, so
  `near.wind_scale` 0.1 used to still seeds with nothing on screen).
  `WIND_COUPLING` 0.08 against `SEED_DRAG_THETA`. A fall takes
  `LANDING_SECONDS` 12 s (`GRAVITY` 5.0, terminal velocity in ~1 s). On
  top of the wind each clump carries its own random gust
  (`World._advance_gust`): `gust_speed` (columns/s, 0 = off, default 6)
  and `gust_period` (mean seconds between gusts, default 1.0) — the target
  jumps to uniform(-1, 1) x `gust_speed` after an exponential wait
  (floor `GUST_MIN_LEG` 0.2 s), no forced sign flip, and the gust share
  of `vx` (`gust_vx`) eases toward it with `GUST_TAU` 0.15 s while wind
  and drag act only on the rest. A merge keeps the larger clump's gust
  state (non-gust `vx` mass-weighted); a burst seed starts its own.
  `_advance_seeds` sub-steps each frame so no clump moves more than
  `SEED_MAX_STEP` (1 sub-cell) per step — charge, merge and landing run
  every sub-step, the cloud scatter on the last only, and a landing's
  burst joins the world after the frame — or the 12 s fall tunnels into
  the pile at `advance(0.5)` or on a tall field.
- A multi-member clump turns as one rigid body: every place a member
  becomes a position (render splat, `_member_cell`, `_anchor`, `_touching`,
  `_merge`, `_explode`) goes through `World._member_offset`, the offset
  rotated by the clump's `angle`; a lone seed is never rotated. Landing
  freezes the turned cells, so a rod horizontal at touchdown lands as an
  arm. Each member's glyph tumble is unchanged. Under `seed_mass ==
  "accrete"`, a cloud entry grows `accrete_count` blocks shaped by
  `accrete_shape`: `rod`, a rod off the
  tip (the member farthest from centre, stepping along its dominant axis;
  a tip at centre picks a random horizontal side); `branch` (default), each
  block on a free side/diagonal neighbour of a member, weighted 1 + distance
  from centre, tip doubled (`_grow_branch`); and kicks `spin` by
  `accrete_spin` with random sign; charge stays +1. Multi-member spin damps
  by `ACCRETE_SPIN_DAMP` per second. All three levers always show on the T page.
- `cloud_fade` (v8, seconds, default 1.2; 0 = pop, the grid passes
  through) is `World._fade_sky`, run on the sky grid after `_sky_cells` is
  cached, so `cloud_at` still reads the engine's own cells. Each cell's
  presence climbs while lit and sinks once cleared; its colour blends from
  `Palette.fade_from` toward the tone, and a cleared cell holds its last
  glyph until presence hits 0. Blends quantise to 64 steps, cached per
  (`fade_from`, tone, step) in `_fade_tint`; the tone ramp itself is 64
  steps (`sky.TONE_STEPS`), so a cell's colour shifts frame by frame with
  its mean while its glyph holds. A row with no presence and an
  all-blank source passes through untouched. Time is `_frame_dt`:
  accumulated by every `advance`, zeroed by each `_fade_sky`, so the fade
  runs on wall time whatever `fps` is. Default `fps` is 8 — 10 broke the
  headless CPU test.
- `World.structure_version` (perf) counts every mutation of `structure`:
  `_anchor`, `_drop_component` (so `drop_floaters`/`_drop_floating`),
  `garden.load_into`, and `resize`/`reseed`. The memoised pile splat canvas
  (`_pile_splat_canvas`) and static pile/speckle layer (`_static_layer`)
  key on it plus `len(structure)` and the dict's identity, so new code that
  mutates `structure` must bump it. `sky.downsample` memoises
  `_cell_colour` on the quantised tone step (`_cell_colour_memo`), cleared
  when the palette or a tone/haze lever changes. Both are exact:
  `World._perf_cache = False` / `sky.COLOUR_MEMO = False` render the
  original path, and `tests/test_field.py` compares the two frame by frame.
- numpy (cactus's one heavy dependency) runs the sky's hot path:
  `PuffSky.composite` (`_composite_np`, patches/windows baked as float64
  arrays) and `downsample` over arrays (`_downsample_np`, owners as
  `_OWNER_NAMES` indices). float64 throughout, so every frame is
  cell-identical to the pure-Python reference behind `sky.NUMPY = False`;
  only tests flip it (`test_numpy_render_matches_the_pure_python_reference`,
  every engine, 40 frames, two sizes). Texture converts its list canvas
  once before `downsample`; fluid stays on lists, where converting measured
  slower. New sky render code keeps both paths and the exactness test.
