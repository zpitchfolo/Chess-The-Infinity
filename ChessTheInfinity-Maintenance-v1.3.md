# Chess the Infinity
## Maintenance Document v1.3

- **Document version:** 1.3.0
- **Current application version:** 1.32.0
- **Current implementation filename:** `ChessTheInfinity-Code-v1.32.0.html`
- **Current source:** [ChessTheInfinity-Code-v1.32.0.html](https://github.com/zpitchfolo/DVAexe/blob/main/Projects/ChessTheInfinity/ChessTheInfinity-Code-v1.32.0.html)
- **Previous source preserved:** `ChessTheInfinity-Code-v1.31.11.html`
- **Previous recipe:** [chessrecipe(v1.11).md](https://github.com/zpitchfolo/DVAexe/blob/main/Projects/ChessTheInfinity/chessrecipe%28v1.11%29.md)
- **Purpose:** Replace the old recipe as the current operational guide. This document records the inspected implementation, confirmed behavior, intended future features, acceptance tests, and the development state machine. Historical recipe files remain unchanged.

> Version mapping: the historical filename `chess8v30.html` is retained. For the new SemVer convention, it is treated as the baseline for application version `1.30.0`. This is a new version label, not a claim that earlier development followed SemVer.

## 1. Product Goal

Chess the Infinity is a browser-based chess application that treats each position after a move as a node in a persistent, branching game tree. Users can play standard chess, navigate history, branch from earlier positions without overwriting other lines, and inspect alternate continuations.

The defining feature is the freedom to explore alternate move histories while preserving the original line.

## 2. Working Baseline and Implementation Facts

### 2.1 Current code

- Single-file browser application: `Projects/ChessTheInfinity/ChessTheInfinity-Code-v1.31.12.html`.
- Card View is implemented as an alternate visualization of the same node tree, with branch rows and horizontal card carousels.
- Node flagging is implemented: each node can be flagged/unflagged, with tree-order flag navigation and a direct-jump flag list.
- Uses chess.js 0.10.3 from external CDN scripts for chess rules and position validation.
- Tree, board, dialogs, appearance controls, and persistence logic live in the HTML file.
- Game state is autosaved in browser `localStorage`; appearance settings use a separate local-storage key.
- JSON export downloads a complete Chess the Infinity tree. JSON import validates the tree and asks before replacing the current game.
- Published through GitHub Pages. GitHub Pages serves the static frontend; it does not provide multiplayer server state by itself.

### 2.2 Actual node model

The inspected `makeNode()` function creates nodes with this shape (optional `flagged` state was added in v1.32.0):

```js
{
  id,
  parentId,
  move: move || null,
  fen,
  ply,
  children: [],
  createdAt: Date.now(),
  result: null,
  flagged: false // v1.32.0; optional when reading older saves
}
```

- `id`: unique string ID.
- `parentId`: parent node ID, or `null` for the root.
- `move`: move metadata, or `null` for the starting position.
- `fen`: chess position stored as Forsyth–Edwards Notation.
- `ply`: half-move depth.
- `children`: IDs of direct continuation nodes.
- `createdAt`: creation timestamp.
- `result`: normalized end/result metadata, where applicable.
- `flagged`: optional boolean bookmark state added in v1.32.0; older saves without it are interpreted as unflagged.

The tree is stored as a node lookup object. The parent and children references represent the same relationships in both directions. New features must keep those relationships consistent.

### 2.3 Actual game-save envelope

The inspected `exportData()` function exports:

```js
{
  format: "ChessTheInfinity",
  version: 1,
  exportedAt,
  rootId,
  currentId,
  nextId,
  flipped,
  autoFlip,
  treeDirection,
  nodes: Object.values(nodes)
}
```

The validator checks the format identifier, node IDs and FENs, root and current positions, parent/child consistency, cycles, and reachability from the root. Import currently replaces the active game tree after explicit confirmation.

**Compatibility rule:** add optional fields with safe defaults where possible. Do not silently change the meaning of existing fields or break old JSON files. If the save schema must change incompatibly, introduce an explicit migration/version strategy and test older exports.

### 2.4 Current tree and navigation behavior found in source

- A tree panel renders nodes and connecting lines.
- Node styling distinguishes the side to move; selected nodes receive strong visual emphasis.
- Selecting a node navigates to that stored position.
- Tree panning, mouse-wheel zoom, Fit Tree, Current Position, and multiple tree directions exist.
- Back moves to the parent. Forward chooses a child using the deepest-continuation behavior. Longest Path is a separate root-to-leaf navigation action.
- Tree-side Back and Forward buttons are present.
- Existing controls include New Game, Delete Future, JSON Import/Export, board flip and Auto Flip, move history, and appearance settings.
- The board uses chess.js to validate and apply legal moves.
- The current import accepts **Chess the Infinity JSON**, not PGN. PGN import is not implemented in v30.
- The code includes shared themed dialogs for alerts, confirmations, and text prompts, plus a custom four-piece promotion chooser.
- No browser-native `alert()`, `confirm()`, or `prompt()` calls remain in the inspected source.
- Appearance settings are separate from the game-tree save. Color presets must not be reintroduced into the user-facing appearance workflow.

## 3. Verified Status and Confidence

The user explicitly tested the v30 build in the browser and reported that it works. The GitHub Pages deployment also became available after deployment delay, and the user confirmed it opened successfully.

User-confirmed behavior from the project's history includes:
- Tree-side Back and Forward navigation works.
- Draw can create a separate branch and the user can return to an earlier position to explore another line.
- Game reload/save, JSON import, and JSON export work.
- The orientation banner updates when flipping the board.
- The v30 dialog and promotion changes were browser-tested by the user and reported working.
- The user tested node flagging in v1.32.0 and reported that it “works pretty well.” This confirms initial functional acceptance, but not exhaustive testing of persistence, wraparound, branch deletion, or responsive-layout edge cases.

This is user-reported verification, not an automated exhaustive test suite. Do not claim every historical feature or edge case has been retested in the latest build. Keep regression testing for existing behavior.

## 4. Development Rules

1. Preserve the last verified implementation. Never overwrite the only known-working copy during a risky change.
2. Inspect the current code and this maintenance document before designing a feature.
3. Clarify ambiguous behavior before implementation; do not silently decide product behavior for the user.
4. Refine the design until it is precise enough to implement, including edge cases, state changes, persistence, and acceptance tests.
5. Make targeted changes. Avoid unrelated redesigns or broad rewrites.
6. Use SemVer for application releases: MAJOR.MINOR.PATCH.
7. Keep maintenance-document versioning independent from application code versioning.
8. Keep master design document milestones independent from both. A new master generation is created only when the user decides the app is feature-complete to the desired scope.
9. Preserve older versioned files and Git history. Never label source-only checks as browser testing.
10. After implementation, inspect the new source, run feasible static checks, then test the actual deployed/local app when possible. Ask the user to verify browser-only behavior when the assistant cannot directly test it.
11. If a regression occurs, use the last working code plus the current maintenance document and the failed version to diagnose and correct the implementation. Retest until it passes.
12. Update this document only after the new implementation is checked; record what is implemented, what is actually tested, and what remains unresolved.
13. Keep the **Current Status** section at the very bottom of this document in future revisions.

## 5. Development State Machine

This application uses the shared workflow defined in [Development State Machine Workflow v1.0](../../DevelopmentWorkflow/Development-State-Machine-Workflow-v1.0.md).

### States

- **S0: Idea / request** - record the goal and ask necessary clarifying questions.
- **S1: Initial design** - create a feature design describing behavior and constraints.
- **S2: Design refinement** - resolve ambiguities, dependencies, edge cases, risks, and acceptance tests.
- **S3: Ready to implement** - user intent is clear and the design is specific enough to build.
- **S4: Implementation** - create a new candidate code version from the maintenance document and working baseline.
- **S5: Source inspection** - inspect changes and run static checks.
- **S6: Runtime/browser testing** - exercise the acceptance tests and regression checks.
- **S7: Decision gate** - if any required test fails, transition to S8; if tests pass and behavior is accepted, transition to S9.
- **S8: Repair** - combine the current maintenance document, last known-working code, and failed candidate to correct the candidate. Return to S5 and retest.
- **S9: Maintenance update** - record actual implementation and verification status; version the maintenance document independently.
- **S10: Feature accepted / next request** - repeat from S0/S1 for the next requested feature.
- **S11: Master milestone** - when the user says the app is complete to the desired scope, produce a master document from the latest maintenance document and verified code. Test whether a fresh LLM session can reproduce a sufficiently similar app from the master document. The result informs the next iteration; it is not a requirement for byte-identical output.

### State transition rule

`request → design → refine → ready → implement → inspect → test → (fail: repair → inspect → test) or (pass: update maintenance) → next feature`

At every repair loop, keep the last known-working code recoverable. A candidate does not become the working baseline merely because it was generated or committed.

## 6. Requested Future Features and Difficulty

Difficulty is a planning estimate relative to the current v1.31.10 single-file app, not a promise. Scale: 1 easy, 2 manageable, 3 moderate, 4 hard, 5 very hard.

| Feature | Estimated difficulty | Main work and risks |
|---|---:|---|
| Goal view: branch starts and flagged nodes | 2–3/5 | Add a focused view/filter that shows only branch-start nodes and flagged nodes, to help the user review important alternatives for a goal. A node that is both a branch start and flagged should not appear as a confusing duplicate. Selecting a result must select the actual node and update the playable board. Define how it handles the root/current node. |
| Goal view: branch starts and flagged nodes | 2–3/5 | Add a focused view/filter that shows only branch-start nodes and flagged nodes, to help the user review important alternatives for a goal. A node that is both a branch start and flagged should not appear as a confusing duplicate. Selecting a result must select the actual node and update the playable board. Define whether the view combines both categories by default and how it handles the root/current node. |
| Existing White/Black node-tree filtering idea | 2/5 | Retain as a possible future view/filter, but clarify whether White/Black means the side that moved, the side to move, or node color before implementation. Keep the active node easy to locate and do not mutate the tree. |
| In-app PGN-to-JSON converter/importer | 3/5 | Parse PGN, replay legal moves with chess.js, and create valid node records using the existing tree schema. Handle headers, comments, result markers, invalid moves, and PGN variations deliberately. Do not confuse PGN with the existing JSON format. |
| Last-known target position and progression toward it | 3–4/5 | Define what “camera” means in the UI, persist the target if desired, and provide forward/back progression along its path when the active node belongs to that tree. Specify behavior when the user jumps to a different branch or the target is deleted. |
| Local computer opponent | 3–4/5 | Integrate an existing chess engine such as Stockfish rather than building an engine from scratch. Test worker/CDN loading, move legality, user-selected side, engine strength, responsiveness, and interaction with branching. Browser hosting and engine asset loading must be planned. |
| Online multiplayer | 5/5 | Requires a real-time communication and shared-state service, identity/room strategy, authoritative move validation, reconnect behavior, conflict handling, and security. GitHub Pages alone cannot host the server component. Not impossible, but a major architecture expansion. |

### Relative comparison with the other app concepts

- **Chess the Infinity:** moderate overall complexity. Standard chess rules are delegated to chess.js; the custom difficulty lies in branching history, navigation, and persistence.
- **DVA writing workspace:** high complexity. Many interacting workspace behaviors, panel layout, persistent writing state, file workflows, and future AI-provider integrations create cross-cutting dependencies.
- **Soundthing DAW:** very high complexity. Real-time audio, timing, looping, slicing, and effects must work together with low latency, alongside a flexible editing canvas.

These are qualitative comparisons based on the described designs, not a full source audit of DVA or Soundthing.

## 7. Proposed Work Order

Do not implement this list without a design pass for the selected feature. Recommended order:

1. **Goal view** showing only branch starts and flagged nodes.
2. **PGN import/conversion** into the existing node tree.
3. **Target position and progression navigation** after the interaction model is specified.
4. **Local computer opponent** using an existing engine.
5. **Multiplayer** only after deciding on a backend and the desired online experience.

The user may reorder, defer, or cancel any feature. “Finished” means finished to the scope the user chooses, not that every possible feature must be built.

## 8. Acceptance Tests for Future Changes

For every feature:
- Confirm the requested behavior with the user before coding if any key behavior is ambiguous.
- Check that the existing game initializes and legal chess moves still work.
- Check branch creation and navigation do not overwrite sibling branches.
- Check save/reload and JSON export/import preserve all relevant new metadata.
- Check the current node, root, parent, children, and FEN remain consistent.
- Test error, cancellation, and invalid-input paths.
- Test responsive UI if the feature adds controls or changes layout.
- Record tests as **source checked**, **runtime tested**, or **user-confirmed**. These are different confidence levels.

Additional feature-specific acceptance tests:

### Card View and Card-to-Tree Focus
- Card View displays the same nodes and branch relationships as the existing tree; it does not create duplicate nodes or alter chess logic.
- Each card renders the actual position and clearly shows whose turn it is.
- Horizontal carousels use left/right arrows; alternate branches are represented in separate rows.
- The view uses whole-view vertical scrolling and remains uncluttered when several branches exist.
- Selecting a card selects that node in the existing tree model and updates the main playable board.
- Switching from Card View to Tree View resets native viewport scroll offsets, restores the transformed tree canvas, and centers the selected node after the viewport has updated.
- Switching views does not change the selected chess position, and returning to Card View retains existing card-centering behavior.
- The user confirmed that the selected-node centering behavior works in v1.31.12.

### Node flags (implemented in v1.32.0)
- The user can flag and unflag the currently selected node.
- Flagged nodes show a flag icon and distinct highlight in both Tree View and Card View.
- The flag list includes flagged nodes across all branches in tree traversal order and allows direct navigation.
- Previous Flag and Next Flag follow the same tree order and wrap at either end. If the current node is not flagged, Next starts at the first flag and Previous starts at the last flag.
- Flag state is optional node metadata, does not change chess position or branch relationships, and defaults to unflagged for older JSON saves.
- Flags are carried through save/reload and JSON export/import. The user reports v1.32.0 works pretty well; exhaustive edge-case testing remains outstanding.

### Goal view: branch starts and flagged nodes
- The view includes branch-start nodes and flagged nodes according to the finalized filter behavior.
- A node matching both criteria appears only once, with both properties still discernible.
- Selecting a result selects the actual node and updates the playable board.
- The view does not modify or prune the underlying tree.
- Empty results, root behavior, and the active node are handled clearly.

### White/Black node-tree filtering
- White/Black controls select the intended sets of nodes according to the finalized definition.
- The active node can still be located and selected.
- Filtering does not mutate the underlying tree or change the active chess position.
- Empty-filter and root-position behavior are clear.

### PGN import
- A basic legal game imports to a valid chain of nodes.
- Headers and game result are preserved when supported by the design.
- Variations are either represented as branches or explicitly reported as unsupported; never silently misrepresent them.
- Invalid PGN fails with a useful message and does not destroy the current game.
- Import confirmation and cancellation preserve the current tree when canceled.
- Exporting the resulting JSON and reimporting it preserves the generated tree.

### Target position / progression
- The target node is clearly visible or identifiable.
- Back/forward progression follows the target's ancestor path, with defined behavior for branches and jumps.
- Deleted or unreachable targets are handled safely.
- Existing navigation controls retain their documented behavior.

### Computer opponent
- Engine loading failures are reported without breaking manual play.
- The engine makes legal moves only and uses the intended side.
- Users can still explore branches and manually alter the computer's prior choice as supported by the existing app design.
- UI remains responsive while the engine thinks.

### Multiplayer
- Two clients remain synchronized on the authoritative position.
- Illegal or duplicate moves are rejected.
- Disconnect/reconnect and stale-state cases are handled.
- One player's branch exploration cannot silently corrupt the shared game.
- No credentials or secret keys are embedded in the static frontend.

## 9. Master Document Milestones

When the user declares the app complete to the desired scope:
1. Freeze the latest accepted code version.
2. Update the maintenance document with accurate final behavior and test status.
3. Create a master document from the verified code and latest maintenance document.
4. Give the master document its own major milestone version (for example, Master 1.0), independent of code SemVer and maintenance-document version.
5. Test the master document in a fresh LLM session and compare the recreated app's behavior and architecture against the source. Document meaningful differences.
6. Future work starts from the latest accepted code plus maintenance document. A new master milestone is created only when the user chooses a new feature-complete checkpoint.

---

## Current Status

- **Maintenance document:** v1.3.0.
- **Current application version:** 1.32.0.
- **Current source:** [ChessTheInfinity-Code-v1.32.0.html](https://github.com/zpitchfolo/DVAexe/blob/main/Projects/ChessTheInfinity/ChessTheInfinity-Code-v1.32.0.html).
- **Previous source preserved:** `ChessTheInfinity-Code-v1.31.12.html`; earlier versions remain preserved.
- **Latest user-confirmed feature:** node flagging in v1.32.0. The user tested it and reported that it works pretty well.
- **Implemented flag controls:** Flag/Unflag Position, Previous Flag, Next Flag, and a direct-jump flag list. Navigation follows tree traversal order across all branches and wraps at either end.
- **Implemented visual treatment:** flag icon and distinct gold highlight for flagged nodes in Tree View and Card View.
- **Persistence:** flag state is optional node metadata, defaults to false for older data, and is included with node data for JSON export/import. The user has not reported exhaustive testing of every persistence and deletion edge case.
- **Latest previously user-confirmed view fix:** switching from Card View to Tree View centers the selected node reliably in v1.31.12; v1.32.0 builds on that source.
- **Testing status:** user-confirmed that node flagging works pretty well; source checks were also performed after commit. This is not an automated exhaustive regression suite.
- **Save model:** `ChessTheInfinity` JSON format v1. The optional `flagged` node property preserves backward compatibility with older saves.
- **Existing import/export:** JSON game-tree import/export is implemented. PGN import is not implemented.
- **Next feature:** a goal-focused view showing branch-start nodes and flagged nodes, without duplicate entries for nodes matching both categories.
- **Deferred idea:** White/Black node-tree filtering remains a possible future feature; its exact meaning must be clarified before implementation.
- **Known limitation:** this is a static browser app using external chess.js CDN scripts; online multiplayer requires additional backend infrastructure.
- **Current feature queue:** goal-focused view for branch starts and flagged nodes; in-app PGN-to-JSON import; target-position progression navigation; a computer opponent; and possible multiplayer.
- **Versioning:** new application versions are separate files; previously published versions are preserved. Maintenance documentation is versioned independently from application code.
