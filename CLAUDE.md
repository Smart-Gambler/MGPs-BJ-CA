# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

MGP's Blackjack Combinatorial Analyzer (BJ CA): a legacy VB.NET Windows Forms application that computes exact blackjack expected values — including splits and insurance — for a configurable rule set. There are no tests and no lint setup.

## Build

- Solution: `BJ CA.sln` (Visual Studio 2008 format, .NET Framework 3.5, Windows-only). Contains the app project `BJ CA/BJ CA.vbproj` and a VS installer project `BJCA Setup/BJCA Setup.vdproj` (requires VS installer-project support; not buildable with plain MSBuild).
- This repo is currently checked out on macOS, where the app cannot be built or run natively; building requires Visual Studio (or MSBuild targeting .NET 3.5) on Windows.
- A prebuilt executable ships in `BJ CA/bin/` along with its runtime data files.
- Nearly every path contains spaces (`BJ CA/...`) — always quote paths in shell commands.

## Architecture

- **Only files listed as `<Compile Include=...>` in `BJ CA/BJ CA.vbproj` are built.** The repo also holds uncompiled working copies/snapshots of the engine kept side by side in the same directory: `Current BJCA.vb`, `Modified Original BJCA.vb`, `Modifying BJCA.vb`, `Current BJCASplitClasses.vb`, `Modified Original BJCASplitClasses.vb`, and `TempStorage.vb`. The live engine is `BJCA.vb` (+ `BJCASplitClasses.vb`); edits to the "Current/Modified/Modifying" variants have no effect on the build.
- **Engine**: `BJ CA/BJCA.vb` — the `BJCA` class runs the whole combinatorial analysis. Entry point is the `BJCA(...)` sub taking a `BJCARulesClass`. It enumerates all player hands (~3084) and dealer hands (~1740), then computes EVs in `#Region`-organized passes (Stand, Blackjack, Double, Surrender, Bonus, splits, etc.). It maintains four strategies as `BJCAStrategyClass` instances: `Opt` (optimal/composition-dependent), `TD`, `TC`, and `Forced`.
- **UI**: `BJCAMainForm.vb` is the startup form (`StartupObject` in the vbproj) where rules are configured; `BJCAResultsForm.vb` (largest file, ~17k lines) displays results and exports to Excel via `Interop.Excel.dll`; `BJCARealtimeForm.vb` / `BJCASmallRealTimeForm.vb` provide real-time play analysis against a live shoe.
- **Rules/state classes**: `BJCARulesClass` (engine input), `BJCAFormRulesClass` (UI-level rule set, binary-serialized to `bin/BJCA.ini` as defaults), plus general/bonus/forced-rule classes, `BJCAShoeClass`, and hand classes in `BJCAHandClasses.vb`. File extensions for saved rule sets (`.rs`, `.set`, `.gen`, `.br`, `.fr`, `.shu`, etc.) are defined in `BJCAFileSetClass.vb`.
- **Dealer-probability caches**: expensive dealer-probability computations are cached as `DealerProbs * .dat` files in `BJ CA/bin/`, loaded/saved via `LoadDealerProbsFile`/`SaveDealerProbsFile` in `BJCA.vb`.
- **`BJ CA/ManualNotes.vb`**: not code — comment-only notes documenting rule-interaction caveats (bonus rule ordering, surrender semantics, split-aces assumptions). Read it before changing rule handling.
- The `.xls` workbooks at the repo root and in `BJ CA/` are the author's verification spreadsheets (counting calculations, hand insurance details, results); the user help file is `BJ CA/MGP's BJ CA Help.doc`.
