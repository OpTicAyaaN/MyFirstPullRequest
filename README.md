# MyFirstPullRequest

## Overview

This repository is used for assignments in `COGS 108`.

## Repository contents

- `README.md`: Repository overview.
- `AG24.txt`: Text file included at the repository root.

## Recent development summary (last 5 commits)

- `c84548d6` — Speed improvements & partial clue progress (2025-05-08)
  - Updates `api/crud/clue.ts` and `api/crud/move.ts`, and regenerates Prisma client artifacts under `generated/prisma/`.
- `e4bee6ab` — Better clues (2025-05-07)
  - Updates clue and game logic in `api/crud/clue.ts` and `api/crud/game.ts`, adds Prisma migrations, and regenerates `generated/prisma/`.
- `fbac57f0` — Added player display name to clues (2025-05-02)
  - Adds required `player_display_name` fields in the Prisma schema and updates the move route payload in `api/routes/moves.ts`.
- `c30394be` — Added move correct column (2025-05-02)
  - Adds nullable `correct` field to the Prisma `Move` model and updates move/game logic in `api/crud/move.ts` and `api/crud/game.ts`.
- `595bc4ba` — Regenerate prisma (2025-05-01)
  - Regenerates Prisma client artifacts under `generated/prisma/` and updates Prisma schema definition files.

## Author

- Ayaan Gill