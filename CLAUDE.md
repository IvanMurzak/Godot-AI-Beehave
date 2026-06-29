# CLAUDE.md — Godot-AI-Beehave

An **addon-dependent ("Class B")** Godot-MCP extension: an MCP tool family for authoring
[Beehave](https://github.com/bitbrain/beehave) behaviour trees (`BeehaveTree`, selector/sequence
composites, action/condition leaves) in the Godot editor, shipped as a **source-only NuGet package**
(`com.IvanMurzak.Godot.MCP.Beehave`) that compiles inside a consumer's Godot project against the
consumer's own GodotSharp. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template). The packaging recipe
is the load-bearing detail — read `docs/source-only-nuget-recipe.md`.

## Class B — addon-dependent + presence-gated (the load-bearing distinction)

Unlike a Class-A extension (which wraps a **built-in** Godot type like `GpuParticles3D`, present in
GodotSharp), this extension wraps the **third-party Beehave addon**, whose classes are NOT in GodotSharp:

- **No compile-time dependency on Beehave.** The package declares ONLY the two MCP pins
  (`com.IvanMurzak.McpPlugin`, `com.IvanMurzak.ReflectorNet`) — never GodotSharp, never Beehave. Beehave's
  classes (`BeehaveTree`, `SelectorComposite`, …) are referenced **by string name only**, so
  `dotnet build -c Debug` must exit 0 **with the addon absent** (the Class-B success gate; CI's nuspec
  assert also enforces no addon/GodotSharp dep).
- **`kind: gdscript`** — Beehave registers GDScript `class_name` types in the **global script-class list**
  (NOT in `ClassDB`). Resolve via `ProjectSettings.GetGlobalClassList()` + `GD.Load`, **not**
  `ClassDB.ClassExists` (that returns `false` for a `class_name` type even when installed — the #1 trap).
- **Presence gate first.** Every editor tool's first action inside `MainThread.Instance.Run(...)` checks
  `AddonInterop.GlobalClassExists("BeehaveTree")` and returns a structured `installed: false` result with
  an install hint when Beehave is missing — never a raw throw (which is an opaque HTTP-500 to the LLM).
- **snake_case dynamic calls.** Drive Beehave nodes via `GodotObject.Set/Get/Call` with **GDScript
  snake_case** member names (not the Class-A PascalCase reflex); enums are plain ints. Centralize the
  class/member name + enum-int constants in a pure-managed `Runtime/Beehave/BeehaveEnums.cs` and unit-test
  them — there are no compile-time types to lean on, so the constants ARE the contract.

The wrapped addon is the consumer's own runtime dependency (see the README's **Required prerequisite**);
this package never vendors or downloads it. CI installs the pinned Beehave (`bitbrain/beehave` @ `v2.9.2`,
addon dir `addons/beehave`) into the testbed only for the e2e leg.

## Layout

- `src/Godot-AI-Beehave/` — the source-only package (`Godot.NET.Sdk`).
  - `Runtime/Tools/Tool_Beehave.cs` — the `[AiToolType]` family (one partial class).
  - `Runtime/Tools/Tool_Beehave.Ids.cs` — all tool-id consts (pure-managed; pinned by tests).
  - `Runtime/Tools/Tool_Beehave.Defaults.cs` — `beehave-defaults` (pure-managed tool).
  - `Runtime/Beehave/` — pure-managed support types: `BeehaveEnums` (class/member name + enum-int consts)
    and the addon-gate result shape (`AddonGateResult` + `NotInstalled(...)`), all unit-tested.
  - `Runtime/Interop/AddonInterop.cs` — dynamic name-resolution helper (global-class-list resolve +
    `GD.Load`); behind `#if TOOLS`, E2E-verified (it constructs Nodes, so not unit-testable).
  - `Editor/Tools/Tool_Beehave.{CreateTree,AddComposite,AddLeaf,GetTree}.cs` — editor tools behind
    `#if TOOLS` (presence-gated; touch live nodes; main-thread-marshalled; E2E-verified).
  - `build/com.IvanMurzak.Godot.MCP.Beehave.props` — the source-injection props (auto-imported by NuGet
    in the consumer; MUST stay named `<PackageId>.props`).
- `tests/Godot-AI-Beehave.Tests/` — xUnit specs for the pure-managed sources only (no Godot binary).
- `testbed/Beehave-Testbed.csproj` — a consumer `Godot.NET.Sdk` project that restores the local-packed
  package; `dotnet build` of it is the source-injection proof.

> The tool files above are authored in iteration 02 of the `create-extension` pipeline; this scaffold
> ships the template's sample tools until then.

## Tools (planned family)

| Tool | Kind | File |
| --- | --- | --- |
| `beehave-defaults` | pure-managed | `Runtime/Tools/Tool_Beehave.Defaults.cs` |
| `beehave-create-tree` | editor | `Editor/Tools/Tool_Beehave.CreateTree.cs` |
| `beehave-add-composite` | editor | `Editor/Tools/Tool_Beehave.AddComposite.cs` |
| `beehave-add-leaf` | editor | `Editor/Tools/Tool_Beehave.AddLeaf.cs` |
| `beehave-get-tree` | editor | `Editor/Tools/Tool_Beehave.GetTree.cs` |

Beehave leaves are **abstract** — `beehave-add-leaf` creates a scaffold node and documents that the user
attaches the behaviour GDScript; the tools only **author** tree structure (the tree ticks in-game).

## Build / test (no Godot binary)

```bash
dotnet build src/Godot-AI-Beehave/Godot-AI-Beehave.csproj   # source-only package compiles tools (Beehave absent is OK)
dotnet test  tests/Godot-AI-Beehave.Tests/Godot-AI-Beehave.Tests.csproj
dotnet pack  src/Godot-AI-Beehave/Godot-AI-Beehave.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Beehave-Testbed.csproj                 # consumes the local package (injection proof)
```

`Godot.NET.Sdk` supplies GodotSharp from NuGet, so no Godot install is needed to build/test/pack or to
prove the source-injection recipe (the testbed build is a faithful proxy for `godot --build-solutions`).
When proving locally, note `dotnet pack` re-uses the **global NuGet cache** for an already-cached version:
if you re-pack the same `Version`, clear `~/.nuget/packages/com.ivanmurzak.godot.mcp.beehave/<ver>` (or
pack a unique version) before re-restoring the testbed, or you'll silently build the stale cached source.

## Conventions

- Root namespace `com.IvanMurzak.Godot.MCP.Beehave`. Every `.cs` starts with the Apache-2.0 header.
- Pure-managed cores + tool-id consts → `Runtime/` (outside `#if TOOLS`, unit-testable); editor-driving +
  addon-interop tools → `Editor/`/`#if TOOLS` (every Godot call via `MainThread.Instance.Run(...)`,
  E2E-verified). At least one pure-managed tool (`beehave-defaults`) so the e2e is never vacuous.
- The package declares ONLY the `com.IvanMurzak.McpPlugin` / `com.IvanMurzak.ReflectorNet` min-version
  deps; **GodotSharp and the Beehave addon must never become a package dependency** (CI asserts the
  nuspec; a `grep` confirms no compile-time reference to Beehave's classes — only string names). Keep the
  MCP pins in lockstep with the core Godot-MCP addon; bump with `commands/update-core.ps1`.
- One `[AiToolType] partial class Tool_Beehave`; one `[AiTool]` method per partial-class file. New
  pure-managed sources must be added to the test csproj `<Compile Include>` list to be unit-tested.

## Find detail in

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece) + the consumer story.
- `docs/ci.md` — workflows, the version gate, multi-Godot matrix, the NuGet publishing setup.
- `class-b-addon-guide.md` (in the create-extension pipeline) — the addon-interop + presence-gate mechanism.
