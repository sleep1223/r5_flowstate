# Repository Guidelines

## Project Structure

This repository contains the deployable R5/Flowstate runtime content. Game scripts live under `vscripts/`; playlist configuration lives in `defaults/platform/`; Flowstate mod resources and RPaks live in `defaults/mods/Flowstate/`. Treat archives such as `sleep1223_r5_flowstate.zip` as generated local artifacts, not source.

## R5 VScript Rules

- Do not execute functions at file top level. The script compiler rejects global statements such as `SomeInit()` with `Global variable definition is followed by "("`. Declare/init functions in the script file, load them before callers in `scripts.rson`, and invoke them from an existing initialization entry point such as a `*_LevelInit()` or other established `*_Init()` function.
- Do not add `global function` declarations for functions implemented in another file. The compiler requires a global function declaration to be implemented in the same file.
- Respect `scripts.rson` load order. Do not directly reference a later-loaded global from an earlier-loaded file; either call it from an established later init point or use a guarded `getroottable()` lookup so load order does not create compile-time undefined symbol errors.
- Do not `expect` or cast to complex types such as structs, static arrays, or `functionref`. For guarded `getroottable()` dynamic function calls, keep the table value as `var`, call it only after confirming the key exists, and cast only the returned simple value when a typed wrapper requires it, for example `return expect bool( dynamicFunc( arg ) )`.
- Make new `*_Init()` functions idempotent with a file-level bool guard before registering callbacks or starting threads.
- Give every long-lived thread a single owner, appropriate `EndSignal`/cleanup behavior, and validity checks inside `OnThreadEnd`; never dereference a disconnected or destroyed entity just for debug logging.
- Prefer standalone compatibility layers for cross-version fixes. Do not directly modify existing functions or behavior that may be called by other scripts unless the user explicitly asks for that change.
- Keep shared structs backward compatible across Flowstate/script versions. Do not remove public fields used by older scripts; when replacing an `entity` reference with a handle, preserve the old field or provide a compatibility sync path.
- Keep duplicated SDK and Flowstate script contracts synchronized. Before changing a shared struct, global function signature, enum, or tracker field, search both this repository and `D:\Project\r5\sleep1223-r5sdk`, then migrate both deployable copies together.
- Treat shared enum values as an ABI: append new values instead of inserting before existing values, and update every server, client, UI, switch, comparison, and networked-state consumer.
- Prefer UID or `GetEncodedEHandle()` for state that may outlive the current frame, player connection, or entity validity. Validate `entity` values with `IsValid()` and `IsPlayer()` before use.
- Validate indexes before reading or writing arrays such as `file.PlayerMetricsArray`; use helpers such as `IsValidPlayerMetricsIndex()` instead of indexing directly from `GetPlayerMetricsIndex*()` results.
- Guard runtime inputs before recording stats or match reports: reject invalid players, empty UIDs, null `damageInfo`, empty weapon names, and non-positive damage where applicable.
- Avoid nested mutable data in hot damage-event structs when a scalar can represent the same state. Store/update damage events through a helper such as `Tracker_StoreDamageEvent()` so array history and lookup maps stay in sync.
- `RandomIntRange( min, max )` uses an exclusive upper bound. Use `array.len()` for array indexes and `count + 1` when selecting an inclusive one-based identifier.
- Every playlist/config variable must have a live code consumer, and its value must be used instead of a hardcoded equivalent. Validate contradictory combinations such as disabling every allowed input method; keep server-specific features default-off unless the repository is explicitly dedicated to that deployment.
- The engine loads the canonical `playlists_r5_patch.txt`; an alternate patch filename has no effect unless a documented build/deploy step explicitly selects or renames it.
- When manually building serialized strings or JSON, skip invalid/empty entries and use an explicit `firstItem`/`firstPlayer` flag for comma placement instead of relying on array indexes.
- Keep `#if SERVER`, `#if TRACKER`, and `#if STUB` contracts aligned. Shared structs and public function surfaces must compile under every relevant preprocessor branch.

## Validation

There is no project-owned automated test harness. For script changes, run `git diff --check`, load the touched scripts in the appropriate server/client context, verify `scripts.rson` ordering, and exercise the affected playlist manually. Check both canonical playlist configuration and all localized UTF-16 resource files when adding user-visible tokens. Binary RPaks cannot be meaningfully code-reviewed from a Git diff; record their source/build process and verify that paired client/server outputs have the expected hashes.

## Commit Hygiene

Keep generated archives, local game files, logs, credentials, and private endpoints out of Git. Prefer focused conventional commits and describe manual runtime verification for script, playlist, localization, or RPak changes.
