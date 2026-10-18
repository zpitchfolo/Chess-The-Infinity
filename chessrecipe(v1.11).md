# Chess the Infinity
## Recipe v1.11

### 1. Project Goal

Build a single-file HTML chess application that combines standard chess with a navigable tree of alternate moves and game timelines.

The user can play normal chess, explore alternate moves, return to earlier positions, and branch from any existing position without losing other lines of play.

### 2. Development Rules

- Preserve existing working features and the current visual design.
- Make targeted changes rather than rewriting working systems unnecessarily.
- Deliver complete replacement HTML inline when code is requested, not patches or instructions to edit individual sections.
- The user has authorized direct GitHub updates for this project. Preserve older versioned files. Use a new HTML version for substantial changes, but update the current file in place for small, explicitly requested UI refinements to avoid unnecessary version clutter.
- Keep recipe versions separate and historical: create a new recipe file when updating this guide, leaving older recipe versions untouched.
- Do not mark a feature verified until the user has tested it or the behavior has otherwise been confirmed.
- Maintain compatibility with older saved games and imported data whenever practical.
- Keep the app self-contained in one HTML file, retaining its existing dependencies and approach.
- Treat the latest source-checked HTML as the current source of truth while preserving the user's earlier browser-confirmed baseline: v10 tree-side navigation and draw branching. v13's banner design was accepted. v23's browser behavior was not confirmed; v24 adds appearance customization and must be browser-tested.
- Do not claim source checks are browser testing.
- When updating this recipe, preserve historical incident records and carry unresolved issues forward until they are confirmed fixed.

### 3. Existing Functionality to Preserve

- Standard chess rules and legal moves.
- Piece promotion, checkmate, and draw detection.
- Branching move tree.
- Back and Forward navigation, including the tree-side controls.
- Forward navigation that selects the child with the deepest continuation.
- Move-history chips.
- Board flip and Auto Flip.
- Board orientation banner with top/bottom side labels and a centered side-to-move indicator.
- Tree panning and cursor-centered zoom using the mouse wheel.
- Fit Tree and Current Position for fitting/centering the tree. Do not add separate Zoom In or Zoom Out buttons unless the user explicitly requests them.
- Longest Path navigation from the root to a deepest leaf.
- Four tree layouts: top-down, bottom-up, left-to-right, and right-to-left.
- Selecting a tree node navigates to its position.
- Delete Future and New Game.
- JSON import/export for game trees.
- Local storage saving.
- Global appearance customization independent of game/frame data, with separate board themes, piece sets, and optional combined presets.
- Filled or outline piece styles independently for White and Black, with separate fill and outline colors.
- Contrast validation for appearance settings and portable custom appearance-library import/export.
- Turn-status labels on tree nodes.
- Tree dragging without unwanted text selection.
- Separate terminal branches for resignation and draw, while preserving the original position for continued exploration.

Do not rewrite working features as part of unrelated changes.

### 4. Tree Toolbar and Footer Layout

#### Current v23 tree toolbar order

Keep the tree controls in this sequence:

1. Back
2. Forward
3. Starting Position
4. Current Position
5. Longest Path
6. Fit Tree
7. Tree Layout

Tree Layout is the final control. Keep Back and Forward in the same toolbar as the other tree controls.

#### Zoom and pan

- Do not show separate Zoom In or Zoom Out buttons. They were removed in v23 at the user's request.
- Keep mouse-wheel zoom available in the tree viewport.
- Keep dragging to pan the tree.
- Keep Fit Tree as the control for fitting the whole tree into view.
- Keep Current Position for centering the selected position at a useful zoom.
- Keep the helper text “Drag to pan · Scroll to zoom · Click a node to travel” unless the user asks to change it.
- Avoid reintroducing redundant zoom buttons during unrelated layout changes.

#### Tree panel header and footer

- Do not add a separate small header bar above the tree toolbar just to show the position/move count.
- Show the position and move count in the bottom footer alongside the active position and branch information.
- Keep the saved-in-this-browser status in the footer.
- Keep the count element's unique ID, `treeCount`, exactly once because the app updates it dynamically.
- The tree toolbar should sit at the top of the tree panel, followed by the tree viewport. The footer sits below the viewport.
- Recheck narrow-screen wrapping when changing toolbar or footer contents.

### 5. Back and Forward Navigation

#### Requirements

- Back navigates to the current node's parent.
- Forward chooses the child with the deepest continuation, rather than always choosing the first child.
- Both board-side and tree-side Back/Forward controls stay synchronized.
- The selected node should remain visible and the tree view should follow navigation appropriately.
- Avoid aggressive zoom changes that make it difficult to maintain a sense of the overall tree.
- Keep Fit Tree, Current Position, Starting Position, and Longest Path usable after one another.
- Repeated navigation must not make the tree blank or corrupt the move tree or chess state.
- If multiple children have equally deep continuations, choose the first child in the existing child order for deterministic behavior.
- Terminal result branches count as descendants, but a short result branch should not beat a longer continuation.

The tree-side Back and Forward buttons were restored in v10, and the user reported that they work. v14 changes the shared Forward behavior to choose the deepest continuation. Browser testing of that change remains outstanding.

### 6. Board Orientation Banner

#### Accepted design

- Use the app's existing panel background rather than a separate colored banner background.
- Show labels identifying the sides at the top and bottom of the board.
- Center the side-to-move indicator between the orientation information.
- The center indicator states whose turn it is, such as “BLACK TO MOVE” or “WHITE TO MOVE.”
- Use a black background with white text when Black moves, and a white background with dark text when White moves.
- Keep the center indicator centered and the overall design readable.

#### Behavior

- Flip Board updates the top and bottom labels.
- Flipping twice returns the labels to their original arrangement.
- The center side-to-move label tracks the actual chess turn, not merely the board orientation.
- Auto Flip continues to work without making the banner misleading.
- Visual changes must not affect chess rules, piece positions, or move legality.

The user accepted the v13 banner design. Static source checks passed, but the latest browser behavior has not been confirmed.

### 7. Resignation Creates a Separate Branch

#### Requirements

- Recording a resignation creates a separate terminal branch/result from the selected position rather than making the original position permanently terminal.
- The original position and its existing continuations remain available.
- The user can continue playing from the original nonterminal position after exploring or recording the resignation branch.
- Preserve the existing confirmation step before resigning.
- Record the correct winner: the side whose turn it is resigns, and the opponent wins.
- Store resignation result metadata with the appropriate branch/node and preserve it through local saving and JSON export/import.
- Continue to support older saved games that lack result metadata.
- A terminal resignation branch cannot receive further moves, while other nonterminal branches remain playable.

The user previously reported that the resignation branch works. Keep this as reported working behavior, while retaining the detailed metadata and compatibility checks as regression requirements.

Implement future fixes with the smallest safe change to the existing tree model. Do not make unrelated changes to draw behavior unless required for compatibility.

### 8. Longest Path and Deepest-Continuation Forward

These are two separate navigation behaviors.

**Forward**
- Inspect the current position's children.
- Recursively determine the greatest descendant depth reachable through each child.
- Navigate to the child with the greatest depth.
- Break equal-depth ties by choosing the first child in the existing child order.
- Both Forward buttons use the same `goForward()` function.
- Back remains ordinary parent navigation.

**Longest Path**
- Remains a separate shortcut that follows a deepest route from the root to a deepest leaf.
- Do not replace it with Forward's local, current-position-based behavior.

In v14, `deepestContinuationLength(id)` calculates descendant depth, and `goForward()` uses it to choose a child. This changes branch selection without changing the tree structure or saved-game format.

Static checks passed for the depth helper, both Forward button bindings, removal of first-child-only navigation, preservation of draw branching, the Longest Path button, and orientation styling.

Browser-test this behavior at a branching position with continuations of different lengths. Confirm both Forward buttons choose the deeper continuation and Back returns to the parent.

### 9. Current Work Order

**Latest HTML:** v24, `chess8v24.html`.

https://github.com/zpitchfolo/DVAexe/blob/main/Projects/ChessTheInfinity/chess8v28.html

**Latest interface changes:** v15-v23 refine the orientation banner, tree-node turn styling and selected-node visibility, tree toolbar, and footer. v23 keeps Back/Forward in the main tree toolbar, removes the extra title/subtitle and mini bar, and places the position/move count in the footer. v24 adds global appearance settings, quick access to recent appearance presets, custom board/piece/combined presets, color controls, piece style/design controls, contrast validation, and custom appearance-library export/import. Game-tree JSON and appearance-library JSON remain separate.

**Appearance architecture:** appearance is saved separately in local storage and does not belong to individual positions or branches. Board themes and piece sets apply independently; combined presets replace the entire appearance configuration. Invalid draft combinations may be previewed but cannot be applied or saved. Invalid imported or combined presets are rejected before application.

**Next step:** Browser-test v23, especially tree navigation, the mouse-wheel zoom/pan interactions, the footer count, responsive toolbar layout, and regressions in previously working features.

Do not add unrelated features or visual refinements unless testing reveals a concrete problem or the user asks.

### 10. Acceptance Tests

Use these checklists as test requirements, not as claims that every test has passed. A checkmark indicates a user-reported or otherwise confirmed result only when explicitly noted in the status sections.

#### Tree controls and navigation

- [ ] Controls appear in the current requested order: Back, Forward, Starting Position, Current Position, Longest Path, Fit Tree, Tree Layout.
- [ ] Separate Zoom In and Zoom Out buttons are absent.
- [ ] Mouse-wheel zoom, tree dragging/panning, Fit Tree, and Current Position remain functional.
- [ ] The position/move count appears in the bottom footer, not in a separate bar above the toolbar.
- [ ] Back selects the correct parent position and keeps it visible in the tree.
- [ ] Forward chooses the child with the deepest continuation.
- [ ] Board-side and tree-side navigation stay synchronized.
- [ ] Equal-depth Forward choices consistently select the first child in existing order.
- [ ] Current Position centers the selected node at a useful zoom.
- [ ] Starting Position returns to the root and keeps the tree visible.
- [ ] Longest Path retains its separate root-to-deepest-leaf behavior.
- [ ] Fit Tree works in all four layouts.
- [ ] Repeatedly switching between navigation, wheel zoom, Fit Tree, and Current Position never leaves the tree blank.
- [ ] Tree dragging continues to pan without accidental text selection.

#### Board orientation banner

- [ ] The banner uses the existing app panel background without a separate banner background color.
- [ ] The top and bottom labels clearly identify the orientation.
- [ ] The center side-to-move indicator stays centered.
- [ ] The center indicator uses black/white styling corresponding to whose turn it is.
- [ ] Flip Board swaps the top and bottom labels.
- [ ] Flipping twice returns the labels to their original positions.
- [ ] The center label tracks the actual side to move.
- [ ] Auto Flip updates orientation information correctly.
- [ ] Orientation cues do not obstruct pieces or legal-move interaction.

#### Draw and resignation branching

- [ ] Recording a draw creates a separate terminal branch from the selected position.
- [ ] The original position and existing continuations remain available after a draw branch is created.
- [ ] The user can return to the earlier position and make another move.
- [ ] The draw branch cannot receive further moves.
- [ ] Resign confirmation appears before the result is recorded.
- [ ] Resignation is represented as a separate terminal branch from the current position.
- [ ] The correct winner is recorded.
- [ ] The original position and its other continuations remain available.
- [ ] The user can resume playing from the original nonterminal position.
- [ ] The resignation branch cannot accept further moves.
- [ ] Result metadata survives saving, reload, export, and import.
- [ ] Older saved games still load.

#### Save and general regression checks

- [ ] The game survives reload with its saved state intact.
- [ ] JSON export and import continue to work.
- [ ] Legal chess moves, promotion, checkmate, and draw detection still work.
- [ ] Board flip and Auto Flip still work.
- [ ] Move history, layouts, Delete Future, New Game, and local saving remain functional.

### 11. Debugging and Recovery Protocol

#### Core principle

The user does not need to know why something broke before debugging can begin. An observation of the failure is enough. Investigate the cause, apply the smallest safe fix, and preserve what was learned.

#### Workflow when a previously working version breaks

1. Record what the user observes and which features fail; do not require the user to diagnose the cause.
2. Establish the latest explicitly user-confirmed working HTML as the baseline.
3. Compare the baseline with the broken candidate before making further changes.
4. Investigate shared startup paths, dependencies, JavaScript errors, missing HTML elements, event-handler registration, state restoration, and other relevant code.
5. Separate confirmed evidence from hypotheses.
6. Make a targeted fix while preserving unrelated working features.
7. Create a new version and leave previous files untouched.
8. Run source checks where possible, but never describe syntax/reference checks as browser testing.
9. Ask the user to test and record exactly what they confirm.
10. Preserve the cause, fix, and reusable lesson in the recipe.

#### Incident log: v07/v08 startup failure

**Symptoms:** The board and tree were absent; New Game and Import JSON did not work; the previously saved game was not appearing.

**Cause found:** v07 and v08 referenced `treeBackBtn` and `treeForwardBtn` in JavaScript even though those HTML elements were absent. Initialization attempted to register handlers on missing elements, causing a startup exception. v06 still contained the elements.

**Fix:** v09 removed the obsolete references and registrations. The user confirmed the board, tree, and buttons work in v09.

**Follow-up resolved in v10:** Removing invalid references prevented the startup crash but also left the tree-side Back and Forward controls absent. v10 restored the actual controls and wired them to the existing navigation functions. The user reported that tree-side Back and Forward work. Do not resolve missing-element errors by silently dropping intended UI controls.

**Save-state note:** The earlier saved-game symptom was not independently established as caused by the missing controls. The user later reported that reload and save/import/export work in v10. Continue watching for regressions rather than retroactively claiming the original symptom had only one cause.

**Reusable lesson:** When many interface features fail together, check shared initialization first. Also verify that a missing-element fix preserves the intended control rather than deleting its behavior.

### 12. Working-Baseline Rule

After the user confirms a version works, it becomes the next debugging baseline. If a later version breaks, compare against that confirmed version first. Keep older versions so regressions can be isolated.

Distinguish between:

- **User-confirmed behavior:** the user has tested and reported it working.
- **Source-checked behavior:** the code has been inspected or checked statically, but not tested in a browser.
- **Untested behavior:** no adequate confirmation is available.

Carry unresolved issues forward until explicitly fixed and verified. Do not infer that an issue is fixed merely because another problem was resolved.

### 13. Feature Status and Acceptance-Test History

#### Tree-side navigation controls

- [x] Tree-side Back and Forward buttons were restored in v10.
- [x] The user reports that tree-side navigation works.
- [ ] Reconfirm that navigation never makes the tree blank or causes unwanted zoom changes.
- [ ] Reconfirm the current tree-control order, footer placement, and all four layouts if those areas are changed.
- [ ] Browser-test the v14 deepest-continuation Forward behavior.

#### Draw branching and saving

- [x] The user reports that Draw creates a separate branch.
- [x] The user reports that the game reloads and that save/import/export work.
- [x] The user reports being able to return to an earlier position and make another move after recording a draw.
- [ ] Verify that a terminal draw branch cannot receive further moves.
- [ ] Confirm compatibility with older saves that lack result metadata if relevant to a future change.

#### Resignation branching

- [x] The user previously reported that the resignation branch works.
- [ ] Detailed winner/result metadata and persistence checks remain regression requirements.
- [ ] Reconfirm that original positions and alternate continuations remain playable after exploring resignation if this code is changed.

#### Orientation banner

- [x] The user accepted the v13 design.
- [x] v13 source uses the app's panel background for the banner.
- [x] v13 source centers the side-to-move label and styles it in black/white according to whose turn it is.
- [ ] Browser-test the latest HTML, including flipping once and twice, changing turns, and Auto Flip.

#### General regression

- [ ] Individually reconfirm preserved chess and interface features after any substantial code change.
- [ ] Keep older versions untouched and use a new filename for each new version.

### 14. Appearance Customization (introduced in v24, refined in v1.4)

#### Scope and persistence
- Appearance is global across every game and branch. It is not stored in node/frame data and must not affect chess positions, move legality, tree structure, or game import/export.
- Remember the last-used appearance across browser restarts using a separate local-storage key.
- Custom appearance presets persist locally. The user must export the custom appearance library to back it up or transfer it to another device.
- Provide a global Appearance Settings entry and a quick board-toolbar menu with recently used board themes, piece sets, combined presets, and a link to full settings.

#### Board themes and colors
- Include several distinct built-in board themes, with individual color controls for light squares, dark squares, board border, last-move highlight, selected square, legal-move indicator, check indicator, and coordinates on light/dark squares.
- Offer both a visual color picker and exact six-digit HEX input.
- Board themes and piece sets are independent. Applying a board theme changes board-related colors only and preserves piece appearance.
- Use live previews. Draft changes may temporarily preview invalid combinations, but invalid combinations cannot be applied or saved.
- Contrast validation focuses on square distinction, pieces against both square colors, and highlights against both square colors and one another. Coordinates have a separate text-readability check. The board border is decorative and must not participate in contrast validation.
- Reject a board-theme change if it would make existing pieces or coordinates fail required contrast checks. Explain the conflict and suggest replacement colors; do not automatically recolor.
- If a draft fails validation, the active appearance remains unchanged until a valid configuration is applied.
- Keep saving simple: one Apply appearance button, one Save as preset action that asks for Board Theme, Piece Set, or Combined Preset, and a separate Manage presets action for rename, duplicate, overwrite, and delete. Do not show six category-specific save/overwrite buttons at once.
- Show a live board preview with both square colors, White/Black pieces, last-move, selected-square, legal-move, and check indicators. Contrast failures should use plain language and tell the user what to change. The board border must never block applying a theme.

#### Piece sets and colors
- Include several built-in piece designs, including outline pieces. v24 uses Unicode chess glyphs with classic, serif, minimal, and outline rendering styles; these are font/rendering variations, not imported external artwork.
- White and Black independently choose filled or outline rendering.
- White and Black each have separate fill and outline colors. Preserve fill values when a side is using outline style.
- Piece design and piece appearance are distinct. Selecting a piece set loads its saved design/style/colors; custom recoloring does not modify the built-in preset.
- Piece rendering must remain legible against both square colors. Use a contrasting edge/outline treatment where appropriate.

#### Presets and editing
- Keep board themes, piece sets, and combined presets in separate categories. Include a small initial library of distinct board themes and piece sets, plus a few combined presets.
- Applying a combined preset replaces board colors and all piece appearance settings.
- Applying a preset must validate the entire proposed appearance before committing. Reject the whole preset if invalid; never partially apply it.
- Changes create a modified current appearance. Save as new or explicitly overwrite an existing custom preset. Never silently overwrite saved presets.
- Built-in presets cannot be overwritten. Provide category resets and a reset-everything action; resetting appearance must preserve the custom preset library.
- Custom preset management should preserve originals unless the user explicitly overwrites them.

#### Appearance library export/import
- Export only the complete custom appearance library in a separate, versioned JSON format. Do not mix it into the chess-game JSON export.
- Import validates the incoming library before modifying the local library.
- For name conflicts, offer rename, skip, or explicit overwrite. Built-in presets cannot be overwritten.
- Report import results. Imported invalid combined presets must be rejected.
- Keep the export format versioned for future migrations.

#### v24 implementation status
- Source code includes a global Appearance Settings dialog and a quick appearance dialog.
- v1.4 refinement consolidates the save controls, adds a live board preview and clearer contrast feedback, and excludes board-border color from contrast validation.
- Source code includes built-in board themes and piece styles, custom preset creation/overwrite, local persistence, and separate appearance-library export/import with a contents summary before conflict handling. Built-in Forest, Amethyst, and Ruby board borders were tuned to satisfy the board-border contrast check.
- Contrast validation and preview/apply separation are implemented in source but require browser testing. In particular, test built-in themes/presets against the validator before claiming every preset is valid.
- v24 is a substantial addition and must be browser-tested for UI overflow, preset save/overwrite/import behavior, local persistence, contrast validation, and regressions in game/tree behavior.

### 15. Latest User Notes

- Tree-side Back and Forward navigation works according to the user's previous tests.
- Draw creates a separate branch.
- The game reloads, and the user reports save, import, and export work.
- The user reported that flipping the board updates the banner. Verify the labels swap after one flip and return after a second.
- The top and bottom labels should identify the appropriate sides, while the center states whose turn it is, such as “BLACK TO MOVE.”
- The user prefers the banner to use the app's overall panel background rather than an additional background color.
- The centered side-to-move label uses the color of the side to move.
- The user accepted the v13 design as good enough. Avoid unnecessary styling work.
- v14 changes Forward navigation to choose the deepest continuation, while Longest Path remains a separate shortcut.
- v14 has passed static source checks, but browser testing remains unconfirmed.

### 16. Version History

- **v09:** Removed obsolete references to missing tree-side navigation elements, resolving the reported startup failure. The user confirmed the board, tree, and buttons work.
- **v10:** Restored the actual tree-side Back and Forward controls. The user reported working navigation and draw branching, along with reload/save/import/export behavior.
- **v11:** Added a high-contrast board-orientation banner.
- **v12:** Centered the color-coded side-to-move label.
- **v13:** Removed the separate banner background, retained the centered indicator, and used side-to-move color styling. The user accepted the design; browser testing was not confirmed.
- **v14:** Added deepest-continuation Forward navigation. Both Forward controls share `goForward()`. The depth helper evaluates descendant branches recursively, with equal-depth ties resolved by existing child order. Longest Path remains separate. Static source checks passed; browser testing was not confirmed.
- **v15:** Refined the orientation banner styling.
- **v16:** Reordered the layout so the orientation banner and board appear before Flip/Auto Flip and the move information/controls/history.
- **v17:** Made the banner more compact and enlarged the lower toolbar buttons.
- **v18:** Centered the “WHITE TO MOVE”/side-to-move label and styled tree nodes according to the side to move.
- **v19:** Fixed oversized side chips and improved selected-node highlighting. The user noted that black selected nodes were easy to see while white selected nodes needed stronger contrast.
- **v20:** Improved white selected-node visibility using a dark separation ring, purple border, glow, and corner markers. The user approved the result.
- **v21:** Applied the selected-node highlight layers and corner markers to black nodes as well.
- **v22:** Enlarged tree toolbar buttons while keeping them within the responsive layout.
- **v23:** Moved Back and Forward into the main tree toolbar; removed the “Infinity Tree” title and “Every node is a saved position” subtitle; removed the small bar above the tree toolbar; moved the position/move count into the bottom footer; removed the Zoom In and Zoom Out buttons and their click handlers. Mouse-wheel zoom and the “Drag to pan · Scroll to zoom · Click a node to travel” helper remain. Static source checks confirmed the intended controls, unique `treeCount` element, and footer placement. Browser testing was not confirmed.

- **v24 appearance refinements:** Added a “Make readable” draft action. It checks square separation and piece visibility against both square colors, attempts outline adjustments first, preserves fill colors unless outline-only changes are insufficient, and previews adjustments before Apply appearance. If square colors themselves are too similar, the action adjusts them to a distinct light/dark pair. Contrast validation no longer checks board border, coordinates, last-move, selected-square, legal-move, or check-indicator colors, and no longer checks piece fill against piece outline. These changes were source-checked; browser behavior remains unconfirmed.

### 17. Simplified Appearance and Unicode Piece Sets (v25)

#### User direction
- The user wants **themes only**, not manual color customization.
- Piece options should focus on **more Unicode/font-based piece sets**, not piece-color controls or custom-drawn artwork.
- Filled and outline styles must be visually distinct. Continue using Unicode chess symbols; do not replace them with SVG or custom-drawn pieces.

#### v25 changes
- New HTML: `chess8v25.html`. v24 remains unchanged.
- The Appearance dialog now presents only Board Theme, Piece Set, a preview, and Apply appearance.
- The quick appearance dialog no longer exposes combined presets or detailed color settings.
- Added eight built-in Unicode piece-set choices: Classic Filled, Classic Outline, Serif Filled, Serif Outline, Minimal Filled, Minimal Outline, Monospace Filled, and Monospace Outline.
- Filled rendering explicitly uses a text fill with a light stroke. Outline rendering explicitly makes the text fill transparent and uses a thicker stroke, instead of relying on drop shadows. The preview uses the same filled/outline distinction.
- Added a monospace font option while retaining the existing font-based Unicode approach. These are font/rendering variations, not custom-drawn silhouettes.
- Existing detailed controls and legacy preset code are retained in a hidden compatibility container to avoid removing older storage/import support during this UI simplification. They are not part of the visible appearance workflow.
- Kept the existing board themes and their built-in colors. Manual color pickers, HEX controls, custom appearance management, and the contrast-adjustment workflow are no longer shown in the normal UI.
- Preserved the legacy piece-set ID `outline` so appearance data saved by v24 can still resolve to the outline set. Older custom appearance-library formats are retained in the underlying code.

#### v25 verification status
- Source was inspected after creation for the simplified controls, eight built-in Unicode piece sets, transparent text fill in outline mode, thicker outline strokes, and legacy outline-ID compatibility.
- No browser test has been performed. The visual difference depends partly on browser/font glyph rendering and must be checked in the actual page. Specifically verify that outline pieces are visibly hollow while filled pieces retain solid interiors, and confirm theme/set selection, preview, Apply appearance, local persistence, and loading older saved appearance data.
- Do not describe source inspection as browser testing. If the user reports the distinction still looks too similar, inspect the actual rendered glyphs before changing the rendering strategy again.

### 18. Board Themes and Highlight Simplification (v26)

#### User direction
- Board themes should change the board palette only, independently from the Unicode piece set.
- Remove the old custom-theme workflow.
- Remove last-move and check highlights because the user dislikes their appearance.
- Provide more built-in board color choices.

#### v26 changes
- New HTML: `chess8v26.html`; v25 remains unchanged.
- Expanded built-in board themes to 18 options: Slate Classic, Forest, Amethyst, Ruby Twilight, Deep Ocean, Jade Garden, Dusty Rose, Lavender Mist, Ember, Desert Sand, Midnight, Monochrome, Arctic Ice, Copper, Black Plum, Teal Neon, Royal Blue, and Charcoal.
- Built-in themes now define only board palette properties: light-square color, dark-square color, board border, and coordinate colors. They no longer change piece colors or move/check highlight colors.
- Removed custom board themes from the available theme list. Legacy saved custom-theme data may remain in local storage for compatibility but is no longer offered as selectable themes.
- Disabled the last-move square overlay and check-square border in the actual board and appearance preview. Selection and legal-move indicators remain unchanged.
- Updated the Appearance note to explain the separation between board themes and Unicode piece sets.

#### v26 verification status
- Source inspection confirms 18 built-in themes, no custom themes in the board-theme selector source, and CSS overrides disabling last-move and check highlights.
- No browser test has been performed. Test all themes, ensure switching a theme only changes board palette, confirm last-move/check highlights are absent while selection/legal-move feedback remains, and test piece-set selection and Apply appearance.
- Local storage may retain old custom theme entries from v24/v25; those entries are not included in the built-in theme selector after this change.

### 19. Single Piece Style and Appearance Cleanup (v27)

#### User direction
- Remove the Quick Appearance button and dialog.
- Remove the piece-set selector because the available choices looked the same.
- Keep one Unicode piece style that works across all board palettes.
- Ensure board themes cannot switch the white and black piece colors, including Charcoal.
- Applying appearance should save and close the dialog.
- Remove duplicate theme entries and the old custom-theme controls.

#### v27 changes
- New HTML: `chess8v27.html`; v26 remains unchanged.
- Removed the Quick Appearance button/dialog and the visible piece-set selector.
- Kept one built-in piece set: Classic Filled. The fixed piece palette is white pieces `#FFFFFF` with dark outline `#18202D`, and black pieces `#202534` with light outline `#FFFFFF`.
- Normalized loaded legacy appearance data to the fixed classic filled piece style and colors, so old saved outline/color settings cannot invert or swap the piece sides.
- Board theme selection changes only the board palette. The selector deduplicates themes by their actual palette values.
- Removed the hidden old color controls, piece-style controls, custom preset buttons, and their event listeners from the appearance dialog.
- Apply appearance now saves the chosen board theme, reapplies the fixed piece palette, and closes the Appearance dialog.

#### v27 verification status
- Source inspection confirms no Quick Appearance button/dialog, no piece-set selector, one built-in piece set, a fixed white/black palette, deduplicated board-theme population, and the Apply handler closing the dialog.
- No browser test has been performed. Test Charcoal and every other board theme, verify white remains white and black remains dark, verify Apply closes the dialog, and confirm the game still initializes without console errors.
- If the browser has old appearance data saved, the load normalization should reset legacy piece styles/colors to the fixed palette while preserving the board theme when valid.

### 20. Board Palette Fixes and Turn Indicator Test Controls (v28)

#### User direction
- Desert Sand and Arctic Ice still did not work because old contrast checks/settings were still active.
- Add Test White to Move / Test Black to Move controls in a centered box alongside the Flip Board and Auto Flip controls.

#### v28 changes
- New HTML: `chess8v28.html`; v27 remains unchanged.
- Removed the old contrast-ratio validation and readability helper logic. Board themes are accepted when the required board palette values are valid six-digit HEX colors; piece-to-square contrast checks no longer block applying a theme.
- Updated Desert Sand to a clearer warm sand palette and Arctic Ice to a more distinct pale-ice / deeper blue-teal palette.
- Added a centered `TEST TURN` box in the board-controls header, with White to Move and Black to Move buttons.
- The test buttons only override the visible turn indicator and turn label for visual testing; they do not alter the chess position or actual side to move. A normal board/tree render clears the test override.
- Removed stale contrast-status wording and unused legacy contrast helper functions.

#### v28 verification status
- Source inspection confirms the contrast-ratio helpers and validation are removed, the two named palettes were updated, and both turn-test buttons are wired to a display-only override.
- No browser test has been performed. Verify Desert Sand and Arctic Ice can be previewed and applied, the old contrast blocker no longer appears, the test box is centered between the title and Flip/Auto Flip controls, and each test button changes only the visible indicator.
- Regression-test actual move logic, Auto Flip, and board orientation after using the test buttons.

### 22. Current Status

- Recipe version: **v1.10**
- Latest HTML: `chess8v29.html`
- Latest HTML URL: https://github.com/zpitchfolo/DVAexe/blob/main/Projects/ChessTheInfinity/chess8v29.html
- Previous HTML: `chess8v28.html`
- Previous recipe: `chessrecipe(v1.9).md`
- Turn label: moved to the Chessboard header, centered between the “Chessboard” title and “Flip Board” button; displays the actual side to move. Source-checked, not browser-tested.
- Back/Forward/Resign controls remain in their existing toolbar; the side-to-move label is no longer inside that row.
- Tree-side Back and Forward: restored in v10; the user reports they work. Both are in the main tree toolbar in v23.
- Forward navigation: v14 implements deepest-continuation selection, with equal-depth ties resolved by existing child order. Browser testing remains unconfirmed.
- Draw branching: the user reports it works and that they can return to an earlier position and make another move.
- Reload/save/import/export: the user reports these work.
- Resignation branch: the user previously reported it works; detailed metadata and persistence checks remain.
- Tree-node selection: v20 improved white selected-node visibility and the user approved it; v21 applied matching highlight layers to black nodes.
- Tree toolbar: v23 order is Back, Forward, Starting Position, Current Position, Longest Path, Fit Tree, Tree Layout.
- Zoom: standalone Zoom In and Zoom Out buttons are intentionally absent in v23. Mouse-wheel zoom, panning, Fit Tree, and Current Position remain required.
- Tree footer: the position/move count is in the bottom footer with the active position and branch text. The extra bar above the toolbar is removed.
- Full chess regression suite, all layouts, responsive layout, and tree navigation/zoom behavior: preserve as requirements; not all have been reconfirmed on v23.
- Next step: browser-test v29. Verify the side-to-move label is centered in the requested header location at desktop and narrow widths; verify it tracks actual moves and branch navigation; confirm the Back/Forward/Resign toolbar has no turn label; regression-test palettes, Auto Flip, orientation, persistence, tree navigation, zoom/panning, and previously working chess features.

**Status placement rule:** Keep the Current Status section at the very bottom of every future recipe version. Refresh it with the latest user-confirmed state, carry unresolved issues forward, and preserve historical incident records. Do not infer that an issue is fixed unless it has been confirmed.


### 23. Shared Dialog and Promotion Overhaul (v30)

#### User-approved direction
- Replace browser-native alerts, confirmations, and prompts with a shared custom dialog system.
- Use one generic dialog layout with configurable title, message, input, and buttons.
- Center dialogs in the full viewport, match the current theme colors, use polished panels with borders and spacing, size to content with sensible width limits, and use subtle fade/scale animation.
- Backdrop clicks and Escape cancel confirmation dialogs and dismiss ordinary dialogs when appropriate. Neither action can confirm a destructive action.
- Keep temporary success/error notifications in their existing style.
- Pawn promotion uses four large existing Unicode chess pieces in one horizontal row, preserving existing glyphs and white/black colors. Selecting a piece immediately completes promotion. Show the destination coordinate and highlight that exact square on the currently displayed board without switching the selected frame or branch.
- Color presets have been removed. Do not restore color preset UI or add preset workflows as part of this overhaul.

#### v30 implementation
- New HTML: chess8v30.html; v29 remains unchanged.
- Added a shared native dialog shell with theme-aware CSS, viewport-centered layout, responsive width/height limits, backdrop blur, and subtle fade/scale entry animation.
- Added shared promise-based helpers for alerts, confirmations, and text inputs. Browser-native alert, confirm, and prompt calls are replaced in source.
- Confirmation backdrop clicks and Escape resolve as Cancel. The dialog buttons are the only way to explicitly confirm a destructive operation.
- Added a four-choice promotion dialog using the existing Unicode glyphs and existing piece-color classes. The target square is highlighted while the promotion chooser is open. Choosing a piece proceeds with that move; cancelling leaves the move unmade.
- Added backdrop/Escape dismissal for the existing Appearance dialog. Its Apply button and appearance behavior remain intact.
- Kept temporary status messages and notifications unchanged. No color preset controls are reintroduced.

#### v30 verification status
- Static source checks confirm no direct browser-native alert(), confirm(), or prompt() calls remain.
- Browser testing has not been performed. Test dialogs at desktop and narrow widths, Escape/backdrop cancellation, all destructive confirmation flows, text input, import success/error paths, promotion as both colors on both board orientations, target-square highlighting, and confirm no frame/branch navigation occurs during promotion.
- Because the app relies on external chess.js CDNs, verify startup when the library is available and its error path when it is not.

### 24. Current Status

- Recipe version: v1.11
- Latest HTML: chess8v30.html
- Latest HTML URL: https://github.com/zpitchfolo/DVAexe/blob/main/Projects/ChessTheInfinity/chess8v30.html
- Previous HTML: chess8v29.html
- Previous recipe: chessrecipe(v1.10).md
- v30 adds a shared theme-aware dialog system and custom pawn promotion chooser. Source-level checks only; browser behavior is not yet confirmed.
- The promotion chooser uses four large Unicode piece glyphs in one row, highlights the destination square, and leaves the current frame/branch selected.
- Confirmations cancel safely on backdrop click or Escape.
- Temporary notifications remain unchanged.
- Color presets remain removed and must not be reintroduced.
- All existing chess/tree functionality remains a regression-test requirement. Browser testing and user confirmation are still pending.

**Status placement rule:** Keep the Current Status section at the very bottom of every future recipe version. Refresh it with the latest user-confirmed state, carry unresolved issues forward, and preserve historical incident records. Do not infer that an issue is fixed unless it has been confirmed.
