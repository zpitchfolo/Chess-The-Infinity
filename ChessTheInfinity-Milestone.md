# Chess the Infinity
## Portable Master Milestone and Reconstruction Specification

> **Purpose:** This document is the complete product and build specification for recreating Chess the Infinity. It is intended to be pasted into a fresh AI chat or handed to an independent builder who has never seen the original project, code, repository, or conversation. The reader must be able to build the game from this document alone.
>
> **Primary success criterion:** Recreate substantially the same game experience and behavior, not merely a chessboard with a few branching controls. Do not require access to any existing source code or ask for the original project as a prerequisite.

---

## 1. Instructions to the builder

Build a complete, usable implementation of the game described below.

1. Treat all requirements labeled **Required** as part of the finished game. Do not silently omit a feature because the implementation is lengthy.
2. Use the **Implementation guidance** as a practical default, not as permission to weaken required behavior.
3. Keep **Ideas outside the required scope** separate. They are not requirements and must not displace required features.
4. If a minor implementation detail is unspecified, make the smallest sensible choice, document the assumption briefly, and continue. Ask questions only when a genuinely consequential product decision cannot be resolved from this specification.
5. Deliver a complete runnable application, not a mock-up, pseudocode, a checklist of tasks, or disconnected snippets. Prefer one complete HTML file with embedded CSS and JavaScript, runnable in a modern desktop browser without a build system.
6. Use a real chess rules library. Do not implement chess legality from scratch.
7. The computer opponent must use a real playable chess engine, not random legal moves disguised as intelligence.
8. Preserve all branches unless the user explicitly invokes a destructive operation and confirms it.
9. Do not claim that a feature has been tested unless it actually has. Separate static/source inspection from browser/runtime testing.
10. After building, run the available checks, report limitations honestly, and provide a short, focused manual test checklist for anything that could not be exercised.

## 2. Product identity

Chess the Infinity is a normal playable chess game joined to a persistent, branching map of chess positions.

In an ordinary chess app, moving back and choosing a different move usually replaces the continuation being played. Here, each move creates a position node in a tree. The user can return to any earlier position, make another legal move, and preserve the other explored lines. The tree is the map of the user's analysis and experiments.

The game combines:
- A fully playable standard chessboard.
- A visual Tree View of the complete branching history.
- A Card View that presents the same history as position cards and branch rows.
- Direct navigation among saved positions.
- Flags/bookmarks for positions worth revisiting.
- Move history, result recording, appearance settings, and reliable local persistence.
- JSON save interchange and PGN game import.
- A real computer opponent that supports adjustable strength and move suggestions while preserving the user's freedom to manually control either side and explore alternatives.

The intended feeling is a practical chess exploration workspace. The board is where moves happen; the tree and cards are different ways to understand, navigate, and preserve the possibilities explored.

### Non-negotiable principles

1. **Branch preservation:** making a move from an earlier node creates a new continuation without deleting siblings or descendants.
2. **One source of truth:** Tree View, Card View, board, move history, flags, results, and navigation all represent the same underlying tree.
3. **Selection means navigation:** selecting a node or card loads that exact position and synchronizes the entire interface.
4. **Results are recorded as branches:** an agreed draw or resignation creates a terminal result child, leaving the parent position available for further exploration.
5. **Predictable navigation:** Back follows the parent relationship. Forward continues along the active branch toward that branch's latest move. Longest Path is a separate command.
6. **Computer assistance is optional:** suggestions and engine moves never remove the ability to select, undo, branch, or manually explore.
7. **Saved work matters:** the full tree, current position, flags, result metadata, and relevant board settings survive reload and export/import.
8. **No fake controls:** every visible control must perform its advertised action or be clearly disabled with an explanation.

---

## 3. Technical and delivery requirements

### Required delivery

- Deliver a complete browser application.
- Preferred implementation: one HTML file with embedded CSS and JavaScript, requiring no build process, package installation, server, or account for ordinary local use.
- Use a compatible chess rules library for legal move generation, FEN loading, move application, promotion, and terminal-state detection.
- Use a real browser-compatible chess engine for computer play. A Stockfish-based integration is a suitable default. Do not write a chess engine from scratch.
- If chess rules or engine assets are external, explain that they require a network connection and handle loading failures gracefully. Do not let a missing dependency leave a blank or unusable interface.
- Keep app data local by default. No account or remote database is required.
- Use browser local storage for autosave, with a visible status indicator and graceful handling of unavailable storage.
- Use DOM elements for interactive nodes/cards and SVG or equivalent vector graphics for tree connectors. Do not render the whole interactive tree as one flat image.
- The app should be comfortable on a desktop browser. Smaller screens should remain usable where practical, but desktop board-and-analysis workflow is the primary layout target.

### General implementation quality

- Use ordinary, readable HTML/CSS/JavaScript unless another approach is clearly justified.
- Use shared functions for duplicate controls so the board and tree controls cannot drift into inconsistent behavior.
- Keep transient interaction state (selected square, legal destination hints, drag state) separate from saved chess-history data.
- Keep appearance preferences separate from the actual game tree.
- Provide useful labels, focus indicators, disabled states, and accessible names for important controls. Icons alone are not enough for critical actions.
- Prevent a single missing DOM element or failed dependency from crashing all initialization.
- Avoid adding unrelated features before the required game is complete.

---

## 4. Main interface and visual design

Create a dark, compact interface with two primary work areas: the chessboard and the history explorer.

### Visual direction

- Dark application background and slightly lighter panels.
- Readable high-contrast text, restrained borders, rounded controls, and a consistent purple/lavender selection accent.
- Board squares use clearly distinguishable light and dark colors.
- Legal move hints use a contrasting accent such as green.
- Flagged nodes use a gold/yellow flag marker and a distinct highlight.
- Avoid excessive decoration. Keep the board and tree/card history visually dominant.
- Use a coherent, consistent visual language across dialogs, cards, tree nodes, buttons, and settings.

### Main regions

1. **Top bar:** application name, compact status/subtitle if useful, and a save-status indicator.
2. **Board panel:** orientation strip, playable chessboard, and board controls.
3. **History panel:** Tree View / Card View switch, relevant navigation controls, and the selected history visualization.
4. **Dialogs:** themed in-app dialogs for confirmations, prompts, errors, import results, and promotion selection.
5. **Appearance settings:** a user-facing settings surface for adjusting supported visual properties. Do not force users through a collection of preset themes as the only customization method.

Layout may adapt at narrower widths, but no view switch, import, flag, or navigation control may become unreachable.

---

## 5. Chessboard and move interaction

### Board rendering

- Render an 8×8 board with correct starting placement for all 32 pieces.
- Pieces may use chess glyphs or another clear representation that works without installing fonts or assets.
- Show the selected square, legal destinations for the selected piece, and the last move where practical.
- Keep the board correctly synchronized with the selected history node.
- The board must remain usable after navigating to any node, importing a game, changing views, flipping orientation, or loading a saved game.

### Making moves

- The user can select a piece and then select a legal destination.
- Selecting another friendly piece may change the active selection.
- Clicking an invalid destination must not change the chess position or add a node.
- Legal destinations should be indicated clearly.
- Use the chess rules library to validate every move.
- Handle castling, en passant, check, checkmate, stalemate, draw conditions, and promotion correctly.
- Promotion must show a clear piece chooser when a choice is required and apply the selected piece.
- After a legal move, create or select the corresponding child node, update the current node, clear transient selection, update the board/history/status displays, render the tree and cards, and autosave.
- If the same continuation already exists, reusing the matching child is acceptable if done consistently. Do not create accidental duplicate children from repeated clicks or double events.

### Orientation and turn indicator

- Provide **Flip Board** and **Auto Flip** controls.
- The orientation strip shows which side is at the top and bottom and prominently displays whose turn it is.
- The side-to-move indicator must reflect the chess position, not the board orientation.
- Flipping changes board orientation and top/bottom labels only. It must not change the selected node, FEN, legal moves, or tree relationships.
- Flipping twice restores the prior orientation.
- Auto Flip follows the side to move. A manual flip turns Auto Flip off, unless the implementation makes this behavior explicit in another equally predictable way.
- The orientation strip and turn indicator must remain accurate after moves, navigation, imports, engine moves, and reload.

---

## 6. The branching history model

### Concept

Each node represents a position in the game history. A node can have zero, one, or many children. Each child represents either a legal chess move from the parent or a recorded terminal result attached to that position.

The tree is not merely a visualization. It is the canonical record of the game and all explored alternatives. Tree View and Card View are visualizations of this same data.

### Required node information

Each node must store equivalent information to:

```js
{
  id: "unique-node-id",
  parentId: null,       // null only for the root
  move: null,           // move leading here; null for root/result child
  fen: "complete FEN",
  ply: 0,               // half-move depth from root
  children: [],         // ordered child node IDs
  createdAt: 0,         // timestamp
  result: null,         // terminal result metadata, otherwise null
  flagged: false        // optional for backwards-compatible reads
}
```

The exact property names may differ, but all equivalent information must be retained.

- The root is the standard starting position for a new game, or the imported game's starting position when representing an imported game.
- Each non-root node has exactly one parent.
- A parent's children list contains the IDs of its direct children in a stable order.
- The child's parent reference must point back to that parent.
- FEN records the complete chess position, including side to move, castling rights, en passant square, and move counters.
- Ply is the number of half-moves from the root in that tree.
- Result nodes share their parent's FEN and ply and contain explicit result metadata.
- Flagging is metadata only. It must not alter the chess position or branch relationships.
- Imported-game metadata such as PGN headers may be stored at the game level or associated with the imported root.

Maintain a top-level state containing at least the node dictionary, root ID, current ID, and a safe unique-ID generator/counter.

### Branch creation

When a move is made from the selected node:

1. Load the selected node's FEN into the chess rules library.
2. Validate and apply the move.
3. If illegal, do not modify the tree.
4. If legal, create a child node containing the resulting FEN, move metadata, parent ID, and parent ply + 1.
5. Append the new child ID to the parent's children without deleting any existing children.
6. Select the new node and update the entire UI.
7. Save the resulting state.

If the user navigates to an earlier position and makes a different move, create another child. Existing branches remain accessible until the user explicitly chooses Delete Future.

### Invariants

The application must keep these rules true after every operation:
- Root exists and has no parent.
- Current node exists.
- Every non-root node has an existing parent.
- Every listed child exists and points to the listing parent.
- No cycles exist.
- Every stored node is reachable from the root, unless the import validator deliberately rejects the file.
- Move nodes contain valid resulting positions.
- Result nodes are terminal and cannot receive moves.
- A node's flag does not affect navigation or chess legality.
- Deleting descendants never deletes the selected node or its ancestors.
- Changing a view never changes game data.

---

## 7. Tree View

Tree View displays the complete branching history as connected position nodes.

### Node content and styling

Each node should display:
- The move that led to the position, or a clear start-position label for the root.
- Side to move.
- Ply/depth or equivalent move-depth information.
- Number of child branches, where useful.
- Relevant chess status such as check, checkmate, stalemate, or draw.
- A clear marker for recorded draw/resignation result nodes.
- A visible flag icon/highlight if the node is flagged.

The current node must be visually distinct. The active path from the root to the current node should also be identifiable without making other branches hard to read. Nodes on alternate branches remain selectable.

Selecting a node navigates to that exact position and synchronizes the board, move history, turn indicator, flags/status, and card view.

### Layout directions

Support four tree directions:
- Top-down.
- Bottom-up.
- Left-to-right.
- Right-to-left.

A **Tree Layout** control cycles among them. Switching layout must preserve all nodes, branches, current selection, and game state. Recalculate coordinates and edges, update the visible layout label, and fit or sensibly reframe the tree.

### Tree navigation controls

Provide these controls in this order:
1. Starting Position
2. Current Position
3. Longest Path
4. Fit Tree
5. Zoom Out
6. Zoom In
7. Tree Layout

Tree Layout must remain the final control in this sequence. Back and Forward controls may also be present in the tree area, but must not disturb the relative order of these seven controls.

The board panel also has Back and Forward. Both copies must call the same shared navigation functions.

### Pan, zoom, and framing

- Dragging empty viewport space pans the tree.
- Dragging/selecting a node selects it; it must not accidentally pan the canvas.
- Panning must not select text or drag page content.
- Mouse-wheel zoom is centered on the cursor.
- Zoom Out and Zoom In scale the tree around the viewport center.
- Fit Tree scales and centers the entire tree.
- Current Position frames the selected node and nearby context, including parent/siblings and children where present.
- Starting Position selects the root and frames it usefully.
- Layout changes and view changes must not make the tree blank or lose the selected node.
- Compute one consistent set of actual rendered bounds for canvas sizing and Fit Tree. Do not apply coordinate offsets twice.
- Nodes should remain separated enough to be selectable and legible for ordinary analysis trees.

Implementation guidance: use a recursive or otherwise robust layout algorithm based on parent-child relationships, with SVG connectors and a transformable canvas. Handle negative coordinates consistently and use the same computed bounds for fitting and canvas dimensions.

---

## 8. Card View

Card View is an alternate visualization of the same tree, not a second history.

### Layout and behavior

- Show position cards grouped into rows that make alternate branches understandable.
- Each card shows the chess position, the move that led to it or a root label, side to move, and useful status/flag information.
- Represent branching continuations in separate rows or an equally clear branch-aware arrangement.
- Cards within a row can be browsed horizontally with left/right controls or a similarly obvious horizontal scrolling mechanism.
- The overall Card View supports vertical scrolling when many branch rows exist. Vertical scrolling must work reliably and must not be trapped by horizontal card carousels.
- Card content must not overlap or become inaccessible when many branches are present.
- The selected card is visually distinct.
- Selecting a card selects the corresponding real node and updates the main board and every other relevant UI component.
- A flagged card uses the same recognizable flag marker and gold/yellow highlight as the flagged tree node.

### Centering and view switching

- When a card is selected, position it near the center of its row's visible card area when practical, so the user can see the selection immediately.
- Switching from Card View to Tree View should frame the selected node in the center of the tree viewport after the view has updated.
- Switching views must not change the selected node or the chess position.
- Returning to Card View should retain sensible selected-card visibility/centering.
- View changes must reset or reconcile stale viewport scroll/transform state as needed, rather than applying old scroll offsets to a newly rendered view.
- Selecting a node from one view and switching to the other must show the same node as selected.

---

## 9. Node flags and flag navigation

Flags are user-controlled bookmarks for important positions. They can mark any node, including the root, a branch point, a result node, or a node on an imported line.

### Required controls

- Flag Position / Unflag Position for the current node.
- Previous Flag.
- Next Flag.
- A direct-jump list of flagged nodes.

### Behavior

- Flagging toggles the selected node's flag state.
- Flag state appears in both Tree View and Card View using a recognizable flag icon and distinct gold/yellow highlight.
- The direct-jump list includes flags across all branches in deterministic tree traversal order and lets the user navigate directly to a chosen flag.
- Previous Flag and Next Flag follow the same tree traversal order and wrap around at either end.
- If the current node is not flagged, Next Flag selects the first flagged node and Previous Flag selects the last flagged node.
- If there are no flags, navigation controls are disabled or produce a clear non-disruptive message.
- If the current node is flagged, Previous/Next move to the preceding/following flag rather than selecting the current flag again.
- Unflagging a node updates all views and the flag list immediately.
- Flagging does not change FEN, ply, parent/children, or move legality.
- Flags persist through autosave, reload, JSON export/import, and PGN-related workflows when a user adds flags after import.
- Older saved data without a flag property defaults to unflagged.

---

## 10. Back, Forward, Longest Path, and position framing

These controls have distinct purposes and must not be conflated.

### Back

- Navigate to the current node's parent.
- At the root, do nothing.
- Never delete a node or branch.
- Synchronize all views after navigation.

### Forward: continue along the active branch toward its latest move

Forward is local navigation along the current exploration line. It is **not** a command to find the deepest branch in the entire tree.

- If a continuation from the current node has already been explicitly selected or visited as the active continuation, follow that continuation.
- When continuing along that branch, move toward its latest stored move rather than switching to a different branch because that branch is longer.
- At a node with multiple children and no remembered active continuation, choose a predictable child, preferably the most recently created child, and remember it as the active continuation.
- If the selected continuation has no child, do nothing.
- Forward must never delete or reorder branches.
- The board-side and tree-side Forward controls call the same function.
- Keep the active-continuation rule stable across view changes and save/reload if the required state can be stored safely. If not persisted, initialize a deterministic default without changing the tree.

This behavior makes Back and Forward useful for moving through the line the user is actually exploring. It prevents Forward from silently jumping into another variation merely because that variation has more descendants.

### Longest Path

Longest Path is a separate explicit shortcut:
- Start from the root, regardless of the currently selected node.
- Find a leaf at the greatest depth.
- Navigate to that leaf and frame it usefully.
- If several leaves tie, use a deterministic tie-break.
- Do not implement this by repeatedly calling Forward from the current node.

### Starting Position, Current Position, Fit Tree

- Starting Position selects the root and frames it.
- Current Position keeps the current node selected and frames it with nearby context.
- Fit Tree frames the complete tree.
- These actions are navigation/framing operations, not tree mutations.

### Move history

Show the sequence of moves along the path from root to the selected node, rather than a session-only log of moves the user has clicked. Selecting a node or navigating must update the displayed line accordingly. A terminal result node should display the recorded result without pretending a chess move occurred.

---

## 11. Draws, resignation, and terminal results

Draw and resignation are recorded as separate terminal child nodes. They do not mutate the parent into a terminal node.

### Agreed draw

1. If the current node is already a result node or the chess rules library says the position is already over, do not create another result.
2. Ask for confirmation.
3. On confirmation, create a result child with the same FEN and ply as the parent, no chess move, and draw metadata.
4. Append the result child to the parent's children.
5. Select the result node, display the result, update views, and save.
6. The result node cannot receive further moves.
7. The parent remains available for continued exploration and alternative branches.

### Resignation

1. Do not record resignation from an already terminal position.
2. Determine the resigning side from the actual side to move at the selected position.
3. Determine the winner as the opposite side.
4. Ask for confirmation and clearly state who resigns and who wins.
5. On confirmation, create a terminal result child with the parent's FEN and ply, no chess move, and resignation metadata including resigning side, winner if stored, and timestamp.
6. Select the result child, update all views, and save.
7. Preserve the parent and all existing branches.
8. A result child cannot receive further moves.

Result nodes should have clear labels and styling in Tree View, Card View, and move history.

---

## 12. Delete Future and New Game

### Delete Future

- If the current node has no children, do nothing.
- Ask for confirmation and explain that all continuations after the selected node will be deleted.
- On confirmation, recursively remove every descendant of the current node and clear its children list.
- Keep the selected node and all its ancestors.
- Remove deleted nodes from every derived UI list, flag list, active-continuation reference, and any target references if such references exist internally.
- Rerender and save.
- Never delete the selected node or an ancestor.
- Do not perform the operation if confirmation is cancelled.

### New Game

- Ask for confirmation before replacing the current game tree.
- On confirmation, initialize a fresh standard chess position and root.
- Reset node IDs safely, current selection, transient piece selection, result state, and active branch continuation.
- Reset board orientation, Auto Flip, and tree layout to sensible defaults.
- Preserve appearance preferences if they are stored separately.
- Rerender all UI and save the new game.
- Cancellation leaves the existing tree untouched.

---

## 13. Saving, JSON export/import, and compatibility

### Autosave

- Autosave after meaningful state changes: legal moves, node selection when current position is persisted, flag changes, results, deletion, imports, orientation/layout changes when included in saved state, and other persisted settings.
- Show a small status such as “Autosaved,” “Restored saved game,” or “Local save unavailable.”
- On startup, restore the most recent valid save when possible.
- If no valid save exists, initialize a fresh game.
- If storage is unavailable or a saved record is corrupt, show a clear status and recover gracefully rather than leaving the interface blank.
- A storage failure must not crash manual play.

### JSON export

Provide Export JSON that downloads the complete tree and persisted state. The exported object must include the equivalent of:
- A format identifier such as `ChessTheInfinity`.
- A save-format version.
- Export timestamp.
- Root ID, current node ID, and safe ID counter/generator state.
- Relevant board orientation and Auto Flip state.
- Tree layout direction and any required navigation context.
- All nodes, child ordering, move metadata, result metadata, flags, and game metadata.

The export must preserve every branch, not just the currently selected line.

### JSON import

- Provide an Import JSON control and file picker.
- Parse and validate the entire file before replacing the current game.
- Validate node IDs, FENs, root/current references, parent/child consistency, cycles, reachability, and supported state fields.
- Normalize optional fields with safe defaults. Unknown optional fields may be ignored if they do not affect required behavior.
- Ask for confirmation only after validation succeeds and before replacement.
- If parsing, validation, or confirmation fails/cancels, leave the current game untouched.
- On success, replace the tree, restore supported state, rerender, frame the selected node, and save.
- Show useful error messages rather than silently failing.

### Backwards compatibility

- Treat optional metadata such as flags, result annotations, imported headers, and UI settings defensively.
- Older saves lacking optional fields should receive safe defaults.
- Do not make a display-only or navigation-only change require a destructive data migration.
- If an incompatible save schema ever becomes necessary, add explicit schema versioning and a migration path.

---

## 14. PGN import

PGN (Portable Game Notation) import is a required part of the intended finished game. It is separate from JSON import: PGN describes chess games and variations, while the app's JSON format stores the full Chess the Infinity tree and its UI/game metadata.

### User experience

- Provide an Import PGN action with file selection and useful feedback.
- Parse the PGN before replacing or adding to the current game.
- If a PGN contains multiple games, let the user choose which game(s) to import or clearly import them as separate games/trees. Never silently discard extra games.
- Preserve available PGN headers such as player names, event, date, site, round, and result as game metadata.
- Show an understandable success summary, including the number of games/positions imported and any variations or annotations that could not be preserved.

### Build the tree

- Replay moves through the chess rules library, validating each move and storing the resulting FEN.
- Convert the main line into a navigable chain of nodes.
- Where the PGN contains recognized variations, represent them as branches from the appropriate position whenever the parser and notation permit. Do not flatten variations into the main line or silently discard them.
- Preserve the recorded game result as metadata. A PGN game result should not be confused with an agreed-draw or resignation action taken interactively unless the PGN specifically identifies that information.
- Imported nodes must follow the same invariants as manually created nodes.
- Imported positions must work in Tree View, Card View, move history, flagging, navigation, saving, and computer play.
- Allow the user to branch from any imported position without modifying the imported line.
- Importing a PGN must not corrupt or partially replace the active tree if parsing fails.
- The user must be able to export the imported tree to JSON and reimport it without losing its branches or supported metadata.

### Error handling

- Reject malformed PGN with a useful explanation.
- If one game in a multi-game file is invalid, report which game failed and preserve the user's ability to import valid games according to the chosen import policy.
- Handle comments, NAGs/annotations, move numbers, headers, and result markers deliberately. Preserve useful metadata where feasible.
- Do not claim full variation or annotation preservation unless the parser actually supports it.
- If a variation is unsupported, report that limitation rather than misrepresenting the imported history.

---

## 15. Real computer opponent

The finished game must include a genuine playable chess engine with adjustable strength and on-demand move suggestions. A browser-compatible Stockfish integration is the recommended implementation path.

### Playing against the engine

- Let the user choose which side to play: White or Black.
- Let the user enable or disable the computer opponent.
- Provide an adjustable difficulty/strength control with understandable labels or levels.
- Difficulty must change the engine's actual search strength or playing parameters, not merely change a label.
- The engine must return legal moves from the selected position and make its move through the same node-creation logic used by manual moves.
- The engine must not make moves from a stale position if the user navigates, branches, changes mode, or changes side while the engine is thinking.
- Prevent duplicate engine moves from repeated events or overlapping requests.
- Show a thinking/busy state and keep the UI responsive while the engine searches.
- Let the user cancel or supersede a pending engine request by navigating or changing the relevant mode, when technically possible.

### Move suggestions

- Provide an explicit action to ask the engine for a suggested move.
- A suggestion must not automatically play the move.
- Display the suggested move clearly, ideally with source/destination highlighting and notation.
- The user can accept the suggestion by making that move or ignore it and continue exploring.
- Suggestions must be based on the currently selected node and discarded if the position changes before the response arrives.

### Manual control of either side

- The user can manually choose moves for White or Black, including the side that would normally be controlled by the engine.
- A mode or control must make it clear whether the engine is playing automatically, offering suggestions only, or not controlling moves.
- Board perspective can be switched independently of whose move is legal. The user can orient the board for the side they prefer to play.
- Perspective switching must not silently change the chosen playing side or alter the chess position.
- The user can disable the engine temporarily to explore both sides manually.

### Taking back engine moves and exploring alternatives

- Engine moves are ordinary child nodes in the same persistent tree.
- The user can navigate back to the position before an engine move and choose a different legal continuation.
- The original engine move and any descendants must remain preserved as a branch unless the user explicitly deletes future branches.
- The user can manually choose the move for the computer's side and then continue the game or request engine assistance again.
- Do not make “undo engine move” silently delete the branch that contained it.
- When the user switches to another branch, the engine must use that branch's actual FEN, not the previously displayed or analyzed position.

### Engine dependency and failure behavior

- Use an established engine integration rather than writing an engine from scratch.
- Explain if the engine requires network access or separately loaded engine assets.
- If the engine fails to load, timeouts occur, or an invalid response is received, show a useful error and keep manual chess play, navigation, branching, and saves functional.
- Never accept an illegal engine move. Validate the response with the chess rules library before applying it.
- Keep engine analysis separate from persistent move history until the user explicitly plays the move.
- Do not let engine failure corrupt the selected position or saved tree.

---

## 16. Appearance settings and dialogs

### Appearance

Provide an accessible appearance/settings panel for supported visual adjustments, such as board-square colors, accent/highlight colors, and other clearly implemented display options.

- Settings should have immediate, visible effects.
- Keep appearance preferences separate from the game-tree data.
- Persist supported preferences locally.
- Provide a reset-to-defaults option.
- Do not include controls that are decorative or nonfunctional.
- Appearance changes must not alter game data, move legality, tree structure, or current selection.

### Dialogs

Use consistent in-app dialogs for:
- Destructive confirmations.
- Import success/failure and invalid input.
- Resignation and draw confirmation.
- Promotion choice.
- Settings prompts where appropriate.

Dialogs must have clear actions, keyboard/focus behavior, and cancellation semantics. Cancelling a confirmation must not partially mutate state. Avoid browser-native alerts/prompts if they would conflict with the app's visual consistency.

---

## 17. State synchronization requirements

Whenever the current node changes, synchronize all relevant surfaces from the same source of truth:

- Chess rules engine position.
- Board pieces and orientation strip.
- Side-to-move indicator.
- Selected tree node and active path.
- Selected card and card scroll position.
- Move history along the root-to-current path.
- Check, checkmate, stalemate, draw, and recorded-result indicators.
- Last-move highlighting.
- Flag controls and direct-jump flag list.
- Active continuation used by Forward.
- Engine mode, pending search state, and suggestion display as appropriate.
- Save status.

Selecting a terminal result child displays the same chessboard as its parent while also displaying the recorded result. A view switch must never change the selected node or position.

---

## 18. Acceptance tests

These tests define required behavior. They are not claims that a particular implementation has already passed them. Mark a test as passed only after actually exercising it.

### A. Startup and basic chess
- [ ] App initializes without an uncaught error when required dependencies load.
- [ ] Starting position and piece placement are correct.
- [ ] Legal moves work; illegal moves do not alter the board or tree.
- [ ] Piece selection and legal destination indicators work.
- [ ] Promotion works with all valid promotion choices.
- [ ] Castling and en passant work when legal.
- [ ] Check, checkmate, stalemate, and draw states are reflected accurately.
- [ ] Missing chess library produces a clear error rather than a blank app.

### B. Branching and node integrity
- [ ] A legal move creates/selects a child with the correct resulting FEN.
- [ ] Returning to an earlier position and making a different move creates a sibling branch.
- [ ] Existing siblings and descendants are preserved.
- [ ] Selecting any node loads its exact position.
- [ ] Parent/child links remain consistent.
- [ ] Every node is reachable from the root; no cycles or orphan nodes are created.
- [ ] A terminal result node cannot receive another move.
- [ ] Flagging does not alter FEN or tree relationships.

### C. Tree View
- [ ] All branches are visible and selectable.
- [ ] Selected node and active path are visually distinct.
- [ ] Node labels/status are legible.
- [ ] All four tree directions work.
- [ ] Layout switching preserves selection and all branches.
- [ ] Panning, cursor-centered wheel zoom, Zoom In/Out, Fit Tree, Current Position, and Starting Position work.
- [ ] Selecting a node does not pan the tree accidentally.
- [ ] Fit Tree uses consistent rendered bounds and does not push the tree off-screen.
- [ ] Required tree controls remain in the specified order.

### D. Card View
- [ ] Card View represents the same nodes and branches as Tree View.
- [ ] Selecting a card updates the main board and selected node.
- [ ] Horizontal card browsing works.
- [ ] Vertical scrolling works across branch rows.
- [ ] Selected-card centering/visibility works.
- [ ] Switching from Card View to Tree View centers the selected node after the new view is rendered.
- [ ] Switching views never changes game state.
- [ ] Flagged cards are clearly marked.

### E. Navigation
- [ ] Back follows the parent; Back at root does nothing.
- [ ] Board and tree copies of Back behave identically.
- [ ] Forward follows the active continuation toward its latest move.
- [ ] Forward does not jump to another branch merely because that branch is longer.
- [ ] At a branch point with no remembered continuation, Forward chooses the documented deterministic default.
- [ ] Forward at a leaf does nothing.
- [ ] Board and tree copies of Forward behave identically.
- [ ] Longest Path starts at the root and reaches a deepest leaf regardless of current selection.
- [ ] Starting Position, Current Position, and Fit Tree work without corrupting or hiding the tree.
- [ ] Move history reflects the selected root-to-current path.

### F. Flags
- [ ] Flag/Unflag Position works.
- [ ] Flag icon/highlight appears in both Tree View and Card View.
- [ ] Direct-jump list includes flags across branches in tree traversal order.
- [ ] Previous Flag and Next Flag navigate in order and wrap around.
- [ ] From an unflagged node, Next Flag starts at the first flag and Previous Flag starts at the last.
- [ ] No-flags behavior is clear and non-disruptive.
- [ ] Flags persist through reload and JSON export/import.

### G. Draw, resignation, deletion, and reset
- [ ] Confirmed draw creates a terminal child without mutating its parent.
- [ ] Confirmed resignation creates a terminal child with the correct resigning side/winner.
- [ ] Cancelling either action changes nothing.
- [ ] Result nodes are visible and cannot receive moves.
- [ ] Returning to a result's parent permits further exploration.
- [ ] Delete Future removes all descendants only after confirmation and preserves the selected node/ancestors.
- [ ] Cancelling Delete Future leaves the tree intact.
- [ ] New Game resets the tree only after confirmation.
- [ ] Cancelling New Game leaves the tree intact.

### H. Persistence and JSON
- [ ] Autosave occurs after all meaningful persisted changes.
- [ ] Reload restores the current tree, selected node, flags, results, and supported UI state.
- [ ] Export includes all branches and relevant metadata.
- [ ] Import validates before replacement and asks for confirmation.
- [ ] Invalid JSON, invalid FEN, duplicate IDs, broken references, cycles, and unreachable nodes are rejected safely.
- [ ] Failed or cancelled import leaves the current game untouched.
- [ ] Older saves without optional metadata load with safe defaults.
- [ ] Local storage failure does not break manual play.

### I. PGN import
- [ ] A standard single-game PGN imports into a valid move tree.
- [ ] Headers and game result are preserved where supported.
- [ ] Variations become branches when supported by the parser.
- [ ] Multiple-game PGNs are handled by an explicit selection/import policy, not silently truncated.
- [ ] Malformed PGN reports a useful error and does not destroy the current game.
- [ ] An invalid game in a multi-game file is reported clearly.
- [ ] Imported positions work with navigation, both views, flags, and engine play.
- [ ] A user can branch from any imported node without altering the imported line.
- [ ] Exporting the imported tree to JSON and reimporting it preserves its structure and supported metadata.

### J. Computer opponent
- [ ] Engine loads and produces legal moves.
- [ ] Difficulty changes actual engine strength/search settings.
- [ ] The user can play as White or Black.
- [ ] The board perspective can be switched independently of the side being controlled.
- [ ] Suggestions display without automatically playing a move.
- [ ] A stale suggestion is discarded when the selected node changes.
- [ ] Engine moves enter the same branching tree as manual moves.
- [ ] The user can navigate back before an engine move and explore an alternative without deleting the original line.
- [ ] The user can manually choose the computer-side move.
- [ ] Pending engine searches do not apply moves to the wrong branch or duplicate a move.
- [ ] Engine loading/failure leaves manual play and saved data usable.

### K. Dialogs, settings, and honest verification
- [ ] Settings visibly affect the intended appearance and persist.
- [ ] Reset settings works without deleting game history.
- [ ] Every visible button performs its advertised action.
- [ ] Confirmations can be cancelled without partial changes.
- [ ] UI remains usable at the intended desktop size and reasonable narrower widths.
- [ ] The final report clearly separates static checks, actual browser/runtime tests, and untested items.

---

## 19. Implementation order

The builder may choose the internal coding order, but the finished app must meet every required acceptance test. A practical order is:

1. Core chessboard, rules-library integration, and the node/tree data model.
2. Branching moves, navigation, move history, and terminal result children.
3. Tree View, layout, pan/zoom, and framing.
4. Card View with horizontal card browsing, vertical scrolling, selection synchronization, and centering.
5. Flags and flag navigation.
6. Autosave, JSON import/export, validation, and backward compatibility.
7. Appearance settings, dialogs, and robust error handling.
8. PGN import with variations and game metadata.
9. Computer engine, strength controls, suggestions, manual side control, and alternative-line exploration.
10. Full regression and acceptance testing.

This order is guidance, not permission to leave later features out of the finished delivery.

---

## 20. Ideas explicitly outside the required scope

These concepts may be interesting future additions, but they are **not required to satisfy this specification**:

- A goal-focused view combining branch-start nodes and flagged nodes.
- Goal-driven exploration toward a desired board position or chess objective.
- Special navigation that follows a target goal or target-position progression beyond the defined Back, Forward, Current Position, Starting Position, Fit Tree, and Longest Path behavior.
- White/Black filtering of tree nodes.
- Online multiplayer.

Do not build these features preemptively at the expense of the required game. The user may choose to promote any idea into the required scope later.

---

## 21. Definition of done

The reconstruction is complete when:

1. The app runs as a usable browser game, not a mock-up.
2. Standard chess play and every required history/navigation feature work.
3. Tree View and Card View remain synchronized representations of the same branching history.
4. Branches, flags, results, and imported lines are preserved correctly.
5. Save/reload and JSON export/import work reliably.
6. PGN import creates a navigable tree and handles variations and multiple games deliberately.
7. A real computer engine provides adjustable-strength play and suggestions without compromising manual control or branch preservation.
8. The acceptance tests have been exercised as far as the environment permits, and untested items are clearly identified.
9. The builder provides the complete runnable artifact and a concise, honest test report.

The target is a substantially equivalent game experience and behavior. Byte-for-byte identical source code is not required, but a builder must not claim equivalence if important controls, branches, navigation behavior, import/export, or engine features are missing.

---

## 22. Self-contained build request

If this document is pasted into a new chat, the instruction to the assistant is:

**Build Chess the Infinity according to this specification. Do not ask for the original code, repository, or prior conversation. Deliver the complete runnable application, preserve every required feature, and use reasonable documented assumptions for minor unspecified details. After implementation, test what you can, state honestly what you could not test, and provide a focused manual acceptance checklist.**
