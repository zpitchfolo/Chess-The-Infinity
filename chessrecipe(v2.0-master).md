# Chess the Infinity
## Master Build Recipe v2.0

**Purpose:** This document is a standalone specification for rebuilding Chess the Infinity in a fresh chat. It must contain enough information to implement the app without access to the original conversation, repository, or previous HTML. It is both a build recipe and an acceptance contract.

**Target artifact:** One complete, self-contained HTML file with embedded CSS and JavaScript. The chess rules library may be loaded from its existing external CDN dependency. Do not split the app into a framework project or multiple files unless the user explicitly changes this requirement.

**Reference implementation:** The existing app reached v14 at `Projects/ChessTheInfinity/chess8v14.html`. It is useful for the second experiment, but this recipe must not depend on it.

---

## 1. How to use this recipe

This recipe is designed for two separate rebuild experiments.

### Experiment A: Recipe-only reconstruction
Start a new chat and provide only this document. Ask the assistant to build Chess the Infinity from the specification. The assistant must not request the old HTML or repository as a prerequisite. If a detail is not specified, it should make the smallest sensible choice, label the assumption, and continue.

### Experiment B: Recipe plus reference
Start another fresh chat and provide this document plus the v14 HTML. Ask the assistant to reconstruct the app using the recipe as the requirements and v14 as the visual/behavioral reference wherever the recipe leaves an implementation detail open.

Keep the results separate. Do not silently add information from the reference implementation into the recipe-only experiment.

### Rules for either experiment
- Deliver a complete replacement HTML file in one code block, not patches or instructions to manually edit sections.
- Preserve every required feature in this recipe. Do not simplify the app by dropping controls or behavior.
- Prefer clear, ordinary JavaScript and CSS over frameworks or a build process.
- Keep the app in one HTML file with embedded styles and script.
- The result must run by opening the HTML file in a modern desktop browser, subject to the chess-library CDN being reachable.
- Explain which checks were actually performed. Static inspection is not browser testing.
- If you cannot run a browser, say so clearly and give a focused test checklist. Never claim that an untested feature works.
- Do not add optional features before the required app is complete.
- If the response is too large for one message, say so and continue with complete, coherent sections. Do not omit code silently.
- Treat a new build as an experiment. Do not overwrite or replace the historical reference file.

---

## 2. Product idea

Chess the Infinity is a chessboard joined to a branching map of possible games.

It supports ordinary chess play, but the user is not trapped on one linear move sequence. Every position can have multiple child positions. The user can return to an earlier position, choose another legal move, and preserve all existing variations. The tree makes the alternatives visible as a navigable structure.

The app should feel like a practical exploration workspace, not a tournament website. The board is the immediate play surface; the move tree is the map of explored possibilities.

### Core principles
1. **Branches are preserved.** Making an alternate move from an earlier position creates another branch. It must not erase siblings or descendants.
2. **Position selection is navigation.** Selecting a tree node loads that position onto the board and synchronizes all relevant status displays.
3. **Terminal results are branches.** Agreeing to a draw or recording a resignation adds a terminal result child to the selected position, leaving the original position available for continued exploration.
4. **Navigation is predictable.** Back follows the parent relationship. Forward chooses the child with the deepest continuation. Longest Path is a separate root-to-deepest-leaf shortcut.
5. **Saved work matters.** The current tree is autosaved locally and can be exported/imported as JSON.
6. **Controls must remain real.** Never remove an intended control merely to silence a missing-element error. Fix the wiring or restore the element.

---

## 3. Technical constraints and dependencies

- Use HTML, CSS, and plain JavaScript in a single file.
- Embed all app-specific CSS and JavaScript in the document.
- Use the same browser chess-rules library as the reference app, exposed as global `Chess` and compatible with the API used below. The original implementation uses chess.js-style methods such as `load`, `fen`, `turn`, `moves`, `move`, `undo`, `reset`, `in_check`, `in_checkmate`, `in_stalemate`, `in_draw`, and `game_over`. Use a compatible version and verify its API.
- If the library cannot load, display a clear status message and alert the user that an internet connection or reload may be needed. Do not let the rest of initialization fail with a confusing blank page.
- Use localStorage for automatic recovery. Use a stable, app-specific storage key.
- Use SVG paths for tree connectors and normal DOM elements for tree nodes. Do not draw the entire tree as a single flat image.
- No server, account, database, package installation, or build step is required.
- The target is a desktop browser. Responsive behavior is desirable, but do not sacrifice the desktop board/tree workflow.

---

## 4. Interface and visual organization

Create a dark, compact application interface with clearly separated board and move-tree work areas. Follow the reference HTML if supplied for exact colors, spacing, and small visual details. If it is not supplied, choose a coherent dark theme with readable contrast, restrained borders, and a purple/lavender accent for the selected path.

### Main work areas
- A chessboard with an orientation/status area.
- Board controls for navigation, orientation, results, game reset, and data transfer.
- A move-tree area showing every stored position and its parent-child relationships.
- Tree controls grouped in the required order below.
- A small save-status indicator that can show states such as “Autosaved,” “Saved game restored,” or “Local save unavailable.”

Use real HTML buttons for controls. Give buttons readable labels, visible focus states, and appropriate disabled states where relevant. Do not rely only on icons for important actions.

### Chessboard
- Render an 8-by-8 board with standard alternating light/dark squares and all 32 starting pieces.
- Use readable chess-piece glyphs or another locally available representation. Piece placement and legal-move handling must be correct.
- Clicking a piece selects it and shows legal destinations. Clicking a legal destination makes the move. Clicking a different friendly piece may change selection. Clicking elsewhere should clear or update selection sensibly.
- Highlight the selected square, legal destinations, and the most recent move where practical.
- Promotion must work. If the chess library requires a promotion choice, present a clear choice and pass the chosen piece to the library.
- Illegal moves must not alter the tree or chess position.
- When a position is check, checkmate, stalemate, or otherwise drawn according to the chess library, reflect the status in the board/tree interface as appropriate.
- Keep the board usable after navigating to any stored position.

### Orientation banner
Place an orientation/status strip directly around the board, using the app panel's background rather than an unrelated decorative banner color.

- Show labels that identify the sides at the top and bottom of the board.
- Put the side-to-move indicator in the center, for example “WHITE TO MOVE” or “BLACK TO MOVE.”
- Center that indicator in its available space.
- When White is to move, style the center indicator with a white background and dark text. When Black is to move, use a black background and white text.
- The center indicator reflects the actual chess position's side to move, not merely board orientation.
- Flip Board swaps which side labels appear at the top and bottom.
- Flipping twice returns the labels to their original arrangement.
- Auto Flip updates the orientation according to the side to move. A manual flip turns Auto Flip off, matching the existing interaction.
- Orientation changes must not change the chess position, legal moves, or selected tree node.

### Board controls
Provide controls for:
- Back
- Forward
- Flip Board
- Auto Flip (visibly shows On/Off)
- Draw
- Resign
- Delete Future
- New Game
- Export JSON
- Import JSON

Back and Forward must also be available in the tree control area. The board-side and tree-side versions must call the same navigation functions and remain synchronized.

---

## 5. Move-tree design and controls

The tree is a spatial view of the stored game graph. Each position is a node. Parent-child edges show how a move or terminal result branches from a position.

### Required tree controls, in this exact order
1. Starting Position
2. Current Position
3. Longest Path
4. Fit Tree
5. Zoom Out
6. Zoom In
7. Tree Layout

Tree Layout must be the final control in this sequence. Keep Zoom Out and Zoom In clearly labeled. If Back/Forward controls are shown alongside the tree, preserve their placement as established by the reference, but do not disturb the seven-control order above.

### Tree nodes
Each node should show:
- The move that led to the position, or a start label for the root.
- Side to move (“White to move” or “Black to move”).
- Ply number or equivalent position depth.
- Number of child branches.
- A visible status marker when the position is check, checkmate, stalemate, or draw, when detected.
- A visible result marker for recorded resignation or agreed draw nodes.

Selecting a node navigates to it. Highlight the selected node and the path from the root to the current node. Make the active path visually distinct from other edges/nodes. Use a useful accessible label or title containing the position and relevant status.

### Tree layouts
Support four layouts, cycled by Tree Layout:
- Top-down
- Bottom-up
- Left-to-right
- Right-to-left

Switching layouts must preserve the tree and selected position. Recompute node and edge positions, update the direction control label, and fit the tree to the viewport. Do not change the move relationships.

### Pan and zoom
- Dragging empty tree-viewport space pans the tree.
- Dragging a node should select the node rather than pan the whole viewport.
- Panning must not accidentally select text.
- Mouse-wheel zoom is centered on the cursor.
- Zoom Out and Zoom In change scale around the viewport center.
- Fit Tree scales and centers the full rendered tree in the viewport.
- Current Position frames the selected node and its immediate context, including its parent/siblings and children where present.
- Starting Position navigates to the root and frames it usefully.
- Navigation should keep the selected node visible without unexpectedly changing zoom in an aggressive or disorienting way.
- All four layouts must work with pan, zoom, and Fit Tree.
- Avoid applying coordinate offsets twice when calculating the canvas bounds or fitting the view. Use one consistent definition of the actual rendered canvas dimensions.

### Tree layout implementation guidance
Use a recursive tree layout based on node relationships. Compute coordinates for each node according to the selected direction, with enough horizontal/vertical separation to avoid overlapping nodes. Draw connectors between parent and child positions using SVG paths. Compute the actual min/max coordinate bounds, offset negative coordinates into the canvas, and use the same bounds for canvas sizing and Fit Tree. Apply pan/zoom with a transform to the canvas container.

The tree may become large. Prefer a straightforward implementation that works reliably for ordinary analysis trees over premature optimization.

---

## 6. Data model: positions, moves, branches, and results

Represent the tree with a dictionary/object keyed by unique node IDs. Maintain identifiers for the root node, current node, and next ID.

Each node should contain the equivalent of:

```js
{
  id: "n1",
  parentId: null,       // null for root
  move: null,           // null for root or a terminal result child
  fen: "...",           // complete chess position
  ply: 0,               // depth from root
  children: [],         // ordered child node IDs
  createdAt: 0,         // timestamp
  result: null          // optional terminal-result metadata
}
```

Use a unique ID strategy that avoids collisions when creating nodes after importing a game. Keep each parent's `children` array in creation order. That order is meaningful for deterministic tie-breaking in Forward navigation.

### Legal move creation
1. Read the currently selected node and load its FEN into the chess engine.
2. Ask the chess engine to make the requested move.
3. If the move is illegal, do not create a node.
4. If legal, create a new node with the move details, resulting FEN, parent ID, and parent ply + 1.
5. Add the new node ID to the parent's children without deleting existing children.
6. Set the new node as current, clear transient square-selection state, render the board and tree, update the move-history display, and schedule an autosave.

If the same move has already been explored from the same position, it is acceptable to select/reuse the existing matching child rather than duplicate it, provided the behavior is consistent. The reference implementation may create a separate child for a newly played variation; follow the reference when available.

### Moving from an earlier node
Navigating to any earlier position and making a legal move adds a child to that position. It must not delete its existing child branches. Existing descendants remain accessible until the user explicitly chooses Delete Future.

### Terminal result metadata
A result child represents a recorded game result without changing the chess position itself. It shares the parent's FEN and ply and has no move. Store explicit result metadata, for example:

```js
{ type: "draw", recordedAt: 0 }
```

or

```js
{ type: "resignation", resigning: "w", recordedAt: 0 }
```

Use `"w"` for White and `"b"` for Black. For resignation, the side whose turn it is resigns and the opponent wins.

A terminal result child cannot receive further moves. The original parent position remains nonterminal and can still be selected to continue play or create another variation.

---

## 7. Navigation rules

### Back
- If the current node has a parent, navigate to that parent.
- If already at the root, do nothing.
- Do not remove any node or branch.

### Forward: deepest continuation
Forward is a **local choice from the current node**, not the same as Longest Path.

1. Get the current node's ordered list of children.
2. If there are no children, do nothing.
3. For each child, calculate the maximum number of descendant edges reachable below that child.
4. Select the child with the greatest descendant depth.
5. If multiple children tie, select the first in the existing child order.
6. Navigate to that child.
7. Both board-side and tree-side Forward buttons call the same `goForward()` function.

A terminal result node is a valid child but has no descendants, so it must not beat a deeper playable continuation.

Example helper:

```js
function deepestContinuationLength(id) {
  const node = nodes[id];
  if (!node || !node.children.length) return 0;
  let deepest = 0;
  for (const childId of node.children) {
    deepest = Math.max(
      deepest,
      1 + deepestContinuationLength(childId)
    );
  }
  return deepest;
}
```

Then compare each current child using that helper. Keep the first child when lengths are equal. Avoid changing the tree or save format merely to implement this navigation behavior.

### Longest Path
Longest Path is a separate shortcut:
- Start at the root.
- Find a leaf with the greatest ply/depth.
- Navigate to that leaf.
- If multiple leaves have equal depth, a deterministic first-encountered leaf is acceptable.
- Frame the resulting node usefully.

Do not implement Longest Path by repeatedly calling Forward from the current position. It must start from the root regardless of the current selection.

### Starting Position and Current Position
- Starting Position selects the root and frames it usefully.
- Current Position keeps the current node selected and frames its local context.
- Fit Tree fits the complete tree.
- These actions should remain usable in succession and must not make the tree blank.

---

## 8. Draw and resignation behavior

### Agreed draw
1. Do nothing if the selected node already represents a recorded terminal result or the chess position is already game-over according to the current implementation's rules.
2. Ask for confirmation before recording an agreed draw.
3. If confirmed, create a new terminal child of the current position with the same FEN and ply, no move, and draw-result metadata.
4. Add it to the parent's children.
5. Select the result child, rerender, keep it visible, and save.
6. The original position and all other branches remain available.
7. The result node cannot receive further moves.

### Resignation
1. Do nothing if the current node is already a recorded terminal result or the chess position is already game-over.
2. Determine the resigning side from the actual side to move.
3. Determine the winner as the opposite side.
4. Ask for confirmation, clearly naming the resigning side and winner.
5. If confirmed, create a separate terminal child of the current position with the same FEN and ply, no move, and resignation metadata containing the resigning side and timestamp.
6. Add it to the parent's children, select it, rerender, keep it visible, and save.
7. Preserve the original position and all other continuations.
8. A result child cannot receive further moves.

Do not turn the original position itself into a terminal node. Draw and resignation must be represented as separate result branches so that the user can return to the parent and continue exploring.

---

## 9. Move history and status synchronization

Provide a move-history display, such as chips or compact entries, derived from the path from root to current node. It must reflect the selected branch rather than only the order in which moves were played during the current session.

When the selected node changes, synchronize:
- Chess engine position.
- Board pieces and orientation banner.
- Current node highlight and active path in the tree.
- Move-history display.
- Side-to-move information.
- Check/checkmate/stalemate/draw indicators.
- Selected-square and legal-destination state.
- Most-recent-move highlighting.
- Save state, where the navigation action changes persisted current position.

Navigating to a terminal result child should show the same board position as its parent while displaying the recorded result status.

---

## 10. Delete Future and New Game

### Delete Future
- If the current node has no children, do nothing.
- Confirm before deleting, explaining that all continuations after the selected position will be removed and the action cannot be undone.
- Recursively remove all descendants of the current node.
- Clear the current node's children array.
- Keep the current node and its ancestors intact.
- Rerender and save.
- Do not accidentally delete the selected node or any ancestor.

### New Game
- Confirm before replacing the current tree.
- Reset the chess engine to the standard starting position.
- Initialize a new root node and reset the node-ID sequence safely.
- Reset board orientation, Auto Flip, and tree layout to their defaults.
- Reset transient board selection and tree view state.
- Rerender and save the new game.
- The old tree is replaced only after confirmation.

---

## 11. Save, restore, export, and import

### Export format
Export a JSON object containing at least:
- A format identifier such as `"ChessTheInfinity"`.
- A format version.
- Export timestamp.
- `rootId`, `currentId`, and `nextId`.
- Board orientation and Auto Flip state.
- Tree layout direction.
- The complete collection of nodes, including child order and optional result metadata.

Offer an Export JSON control that downloads a file named `chess-the-infinity.json` or a similarly clear name.

### Autosave
- Save the current tree and UI state to localStorage after meaningful changes.
- Debouncing autosave by a short interval is acceptable.
- Show “Autosaved” after a successful write and “Local save unavailable” if storage fails.
- On startup, attempt to restore the last saved state. If no valid save exists, initialize a new game.
- If a saved state cannot be restored, log a useful diagnostic and recover to a fresh starting game rather than leaving a blank interface.
- A storage failure must not crash the app.

### Import
- Import JSON through a file input.
- Parse and validate the file before replacing the current game.
- Ask for confirmation before replacing the current tree.
- On failure, show a clear message and leave the current game intact.
- Restore all persisted state fields supported by the format, including result metadata when present.
- Rerender, fit/frame the tree, and save after successful import.

### Import validation
Validate the data defensively before applying it:
- The top-level object has the expected format identifier and a node array.
- Each node has a string ID and FEN string.
- IDs are unique.
- Every FEN can be loaded by the chess engine.
- The root exists and has no parent.
- The current node exists.
- Every non-root node has an existing parent.
- Every child ID exists and points back to the correct parent.
- No cycles exist.
- Every node is reachable from the root.
- The tree layout value is one of the supported directions; otherwise use the default.
- Missing optional fields in older saves receive safe defaults.
- Unknown or missing result metadata is normalized to null rather than causing a crash.
- Missing `nextId` receives a safe fallback.

Do not blindly trust imported JSON or replace the current game before validation and user confirmation have succeeded.

### Backward compatibility
Older saved games may lack result metadata or newer UI-state fields. Treat these fields as optional and provide defaults. Keep the core tree readable when possible. Do not make the save format incompatible for a change that only affects display or navigation.

---

## 12. Suggested internal function boundaries

Names may differ, but preserve the responsibilities and keep shared behavior shared.

- `makeNode(...)`: create a node with consistent defaults.
- `initTree()`: initialize the standard root and reset current state.
- `currentNode()`: return the selected node.
- `pathTo(id)`: return the path from root to a node.
- `gotoNode(id)`: select a valid node, load its FEN, synchronize UI, and save as appropriate.
- `renderAll()`: rerender board, move history, and tree.
- `renderBoard()`: render squares and pieces, highlights, orientation labels, and side-to-move indicator.
- `renderTree()`: calculate positions, render SVG edges and DOM nodes, update selection/path styling, and update the layout label.
- `computeTreeLayout()`: return node coordinates and coordinate bounds for the current layout.
- `applyTreeTransform()`: apply pan/zoom transform.
- `fitTree()`: fit the complete tree.
- `focusNodeContext(id)`: frame the selected node and nearby family.
- `zoomTree(factor, x, y)`: zoom around a viewport coordinate.
- `goBack()`: parent navigation.
- `deepestContinuationLength(id)`: recursively calculate maximum descendant depth.
- `goForward()`: select the deepest child of the current node, deterministic first-child tie break.
- `longestLeafFrom(rootId)`: locate a deepest leaf from the root.
- `saveState()`: persist the current exportable state.
- `exportData()`: create the canonical serializable object.
- `validateImport(data)`: validate and normalize imported or saved state.
- `loadSavedState()`: restore a valid saved game.

Use one shared implementation for duplicate controls. In particular, both Forward buttons must call the same `goForward()` function and both Back buttons must call the same `goBack()` function.

---

## 13. Common failure modes to prevent

### Missing DOM elements at startup
A prior version referenced `treeBackBtn` and `treeForwardBtn` in JavaScript when the elements were missing from the HTML. Registering listeners on missing elements caused an initialization exception, leaving the board/tree absent and other controls unusable.

**Rule:** Every element referenced during initialization must exist, or its optional presence must be checked intentionally. If a control is part of the design, restore it and wire it correctly rather than removing the behavior to suppress the error. Check the browser console when multiple interface areas fail together.

### Tree fit and coordinate mismatch
The tree canvas dimensions and Fit Tree calculations must use the same coordinate bounds and offsets. Applying an offset twice can make the tree appear misplaced or poorly fitted.

### Branch destruction
A move made from an earlier node must append a child and preserve siblings. Only Delete Future may remove descendants, and it must do so after confirmation.

### Result destroys its parent
A draw or resignation must create a terminal child, not mark the parent position terminal. The user must be able to return to the parent and continue play.

### Forward and Longest Path conflated
Forward chooses the deepest child of the current node. Longest Path starts at the root and jumps to a deepest leaf. They are not interchangeable.

### UI state becomes misleading
The side-to-move indicator follows the actual FEN/chess engine turn. Board flip changes orientation labels only. Auto Flip must not make the indicator lie about whose turn it is.

### Import partially overwrites a game
Parse and validate the entire import first, then ask for confirmation, then replace the live state. If anything fails, preserve the existing game.

---

## 14. Acceptance tests

These are required tests, not claims that the implementation has already passed them. Mark a test complete only after actually performing it and recording the result.

### A. Startup and basic chess
- [ ] The HTML opens without a startup exception when the chess library is available.
- [ ] The starting board is correct.
- [ ] Legal moves work.
- [ ] Illegal moves do not alter the position or tree.
- [ ] Piece selection and legal destination hints work.
- [ ] Promotion works.
- [ ] Check, checkmate, stalemate, and draw states are handled.
- [ ] A missing chess library produces a clear failure message rather than a blank, broken interface.

### B. Branching tree
- [ ] Making a move creates/selects the correct child node.
- [ ] Making a different move from an earlier node creates a sibling branch.
- [ ] Existing siblings and descendants remain available.
- [ ] Selecting a tree node loads its exact FEN on the board.
- [ ] The active path and current node are visibly distinct.
- [ ] Node labels show move/start, side to move, depth, and child count.
- [ ] Four tree layouts preserve all branches and the current selection.
- [ ] Tree Layout remains the final control in the required control sequence.

### C. Navigation
- [ ] Back selects the parent; Back at root does nothing.
- [ ] Both Back buttons behave identically.
- [ ] Both Forward buttons behave identically.
- [ ] Forward chooses the child with the deepest continuation.
- [ ] Equal-depth ties select the first child in the parent's existing child order.
- [ ] A short terminal result branch does not beat a deeper playable branch.
- [ ] Forward at a leaf does nothing.
- [ ] Longest Path starts from the root and reaches a deepest leaf regardless of the previously selected node.
- [ ] Starting Position and Current Position work.
- [ ] Fit Tree works in all four layouts.
- [ ] Repeated navigation and fitting do not make the tree blank or corrupt state.

### D. Orientation and board interaction
- [ ] The banner uses the panel background.
- [ ] Top and bottom labels identify the orientation correctly.
- [ ] Center side-to-move indicator is centered.
- [ ] White-to-move uses white/dark styling; Black-to-move uses black/white styling.
- [ ] Flip Board swaps orientation labels without changing the position.
- [ ] Flipping twice restores the original labels.
- [ ] Auto Flip tracks the side to move.
- [ ] Manual flip turns Auto Flip off.
- [ ] Banner remains accurate after move navigation, import, and reload.

### E. Tree viewport
- [ ] Dragging empty space pans the tree.
- [ ] Dragging a node selects it instead of panning.
- [ ] Panning does not select text.
- [ ] Mouse-wheel zoom centers on the cursor.
- [ ] Zoom controls work.
- [ ] Fit Tree uses actual canvas bounds.
- [ ] Current Position frames the selected node and nearby context.
- [ ] Tree nodes do not overlap under ordinary game sizes.
- [ ] All four layouts work with pan and zoom.

### F. Draw and resignation
- [ ] Draw asks for confirmation.
- [ ] Confirmed draw creates a separate terminal child with draw metadata.
- [ ] Cancelled draw leaves the tree unchanged.
- [ ] The original position remains playable after visiting the draw branch.
- [ ] Resign asks for confirmation and names the resigning side and winner.
- [ ] Cancelled resignation leaves the tree unchanged.
- [ ] Confirmed resignation creates a separate terminal child.
- [ ] The side to move is recorded as resigning; the opponent is the winner.
- [ ] Result nodes cannot receive moves.
- [ ] Other continuations remain available.

### G. Data safety
- [ ] Autosave works after moves and meaningful navigation/state changes.
- [ ] Reload restores the game and selected position.
- [ ] Export produces valid JSON with all branches and result metadata.
- [ ] Import restores all branches and UI state.
- [ ] Invalid JSON is rejected without losing the current game.
- [ ] Broken node relationships, duplicate IDs, cycles, unreachable nodes, and invalid FENs are rejected.
- [ ] Older saves missing optional metadata still load.
- [ ] LocalStorage failure does not crash the app.
- [ ] Delete Future asks for confirmation and removes only descendants.
- [ ] New Game asks for confirmation and resets to the starting position.

---

## 15. Test-report discipline

For each build, report these categories separately:

- **Implemented:** The code contains the feature.
- **Source-checked:** The relevant source was inspected or checked statically.
- **Browser-tested:** The behavior was exercised in a browser.
- **User-confirmed:** The user tested it and reported the result.
- **Known defect:** A concrete failure has been observed.
- **Untested:** There is not enough evidence to claim it works.
- **Planned:** A future improvement, not required for this baseline unless explicitly promoted to required.

Do not describe source inspection as a successful runtime test. Do not convert an unchecked acceptance item into a checked one just because the code appears plausible.

When testing a rebuilt version, keep a concise table of test, result, and evidence. Record failures as observations first, then investigate. The user should not have to diagnose the cause before debugging can start.

---

## 16. Historical reference and known status

This section preserves project history to help a future chat understand the target. It is not permission to skip the acceptance tests.

### Earlier versions
- **v09:** Removed obsolete references to missing tree-side navigation elements after a startup failure. The user confirmed the board, tree, and buttons worked.
- **v10:** Restored the actual tree-side Back and Forward controls. The user reported that tree-side navigation works, Draw creates a separate branch, the game reloads, and save/import/export work.
- **v11:** Added a high-contrast board-orientation banner.
- **v12:** Centered the side-to-move label.
- **v13:** Used the panel background, centered the indicator, and styled it according to the side to move. The user accepted the design.
- **v14:** Added deepest-continuation Forward navigation. Both Forward controls share `goForward()`. Static source checks passed; browser testing of this change was not confirmed.

### Known user-confirmed reports
- Tree-side Back and Forward worked in v10.
- Draw created a separate branch.
- The user could return to an earlier position and make another move after recording a draw.
- Reload and save/import/export worked in v10.
- The user previously reported that the resignation branch worked.
- The user accepted the v13 orientation-banner design as good enough.

### Unverified or outstanding checks
- Browser-test v14's deepest-continuation Forward behavior with branches of different lengths.
- Confirm both Forward controls select the same deeper child and Back returns to its parent.
- Recheck result metadata persistence and compatibility with older saves when changing that code.
- Browser-test banner behavior across moves, flips, Auto Flip, imports, and reloads.
- Re-run regression checks after a substantial rewrite.

### Important incident lesson
In v07/v08, JavaScript referenced `treeBackBtn` and `treeForwardBtn` even though those elements were absent. The resulting initialization error caused multiple interface features to fail together. The initial v09 fix removed the invalid references but also left intended tree-side controls absent; v10 restored them. The reusable lesson is to check shared initialization when many features fail together and never silently drop intended UI controls to fix a missing-element error.

---

## 17. Nice-to-have features, deliberately out of scope

The baseline is the working chess exploration app described above. Do not let optional ideas delay or destabilize it. Possible future enhancements can be discussed after the required behavior is implemented and tested. Treat them as planned rather than existing functionality unless explicitly built and verified.

Do not invent a new feature list or implement speculative extras in the initial reconstruction.

---

## 18. Definition of success

The recipe succeeds when a fresh chat can use this document to produce a complete single-file HTML app with the required board, branching tree, navigation rules, result branches, orientation behavior, storage/import/export, and viewport controls, without needing the original conversation.

For the two-experiment test:
- Experiment A measures whether this recipe alone is sufficiently complete.
- Experiment B measures how much closer the result gets when the reference HTML is also available.
- Compare the outputs using the same acceptance checklist.
- Any missing requirement discovered in Experiment A should be added to a later recipe version, not silently supplied only to that experiment.

**Keep this document self-contained. When updating it, preserve the build instructions, core behavior, validation rules, acceptance tests, and unresolved status.**