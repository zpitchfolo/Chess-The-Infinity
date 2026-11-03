# Chess the Infinity

Chess the Infinity is maintained in this standalone repository.

## Current files

- [index.html](./index.html) — current browser application (v1.32.0).
- [ChessTheInfinity-Maintenance.md](./ChessTheInfinity-Maintenance.md) — current implementation notes, verified behavior, feature queue, and acceptance checks (document version 1.4.3).
- [ChessTheInfinity-Milestone.md](./ChessTheInfinity-Milestone.md) — reconstruction-oriented project specification.
- [Development-State-Machine-Workflow-v1.2.0.md](./Development-State-Machine-Workflow-v1.2.0.md) — reusable development workflow adopted by this repository.
- [testgameV03.json](./testgameV03.json) — example game file for checking JSON import.

## File and version convention

Keep one current file per living project document. Update documents in place and record their independent SemVer version inside the document; Git history preserves previous revisions. Do not create a new version-suffixed document file for routine edits.

Application code follows a separate versioning process: preserve the last accepted baseline and keep implementation candidates recoverable. The maintenance document explains this distinction.
