# CRUD state discipline for prototype features

Hard-won rules for building save/overwrite/select features (scenes, presets, favorites) whose
state must survive every gesture, refresh, and cross-entity effect. Extracted from Castle
Console's scene system, which took five user-reported bug rounds to nail down — each rule
below traces to a real shipped bug.

## The rules

1. **One serialization source for Create and Update.** If "create" builds its field set in the
   client and "overwrite" builds it on the server (or in a second switch statement anywhere),
   they WILL drift and saves become inconsistent. Put the entity→fields snapshot in ONE
   function (server-side) and have both paths call it. (Bug: resave preserved old patch keys,
   so an `on`-only scene never learned a volume adjustment — "settings don't stick".)

2. **Commit on `change`, never on `click`, for slider-like controls.** A touch DRAG fires no
   click; mouse tests pass while every phone adjustment is silently lost. Verify with real CDP
   touch drags — synthetic clicks lie.

3. **Hold gestures must be strict.** Primary pointer only; a second concurrent pointerdown
   CANCELS (never re-arm a shared timer — leaked timers fire phantom overwrites on rows the
   user never held); >8px movement cancels (scroll intent must never mutate); up/leave/cancel
   all clear; swallow the post-hold ghost click in a capture-phase handler so the hold doesn't
   also trigger the tap action.

4. **Selection is explicit, persisted, and invalidated deliberately.** Store "which entity is
   active" per scope (e.g. per user+room) and write down the full invalidation matrix:
   - what SETS it (run, create, overwrite),
   - what CLEARS it (any manual adjustment that makes it stale),
   - what happens on refresh (both selected and cleared states persist),
   - cross-scope effects (an action touching multiple scopes updates ALL of them — a
     cross-room scene run must mark/replace the selection in every room it touches).

5. **Complete the CRUD square before handoff.** If users can Create and Update, they will need
   Delete — and Delete must clean up every reference (selection keys, caches, UI) in the same
   motion. Guard Create against double-submit (disable the button while in flight).

## The test harness (all mutations, every time)

- **Isolation via full-table diff**: snapshot ALL rows before a mutation, assert exactly the
  intended row changed and every other row is byte-identical. Catches phantom writes that
  single-row assertions miss.
- **Real gestures**: CDP touch drags for sliders, touchStart+delay+touchEnd for holds,
  two-finger and drag-on-button cases asserting ZERO mutations.
- **Persistence cycle**: after each state transition (set, clear, overwrite, delete), reload
  and re-assert.
- **Cross-scope**: trigger the multi-scope action and assert every affected scope's state.
- **Restore**: capture entity + device states up front; restore them and delete test rows
  afterward — demo data must leave the run pristine.
