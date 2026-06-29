<h1 align="center">Godot AI Beehave</h1>

<p align="center">
  AI <b>MCP tools</b> for authoring <b>Beehave</b> behaviour trees in the Godot editor — an
  <b>addon-dependent</b> extension for
  <a href="https://github.com/IvanMurzak/Godot-MCP">Godot-MCP / AI Game Developer</a>.
</p>

`Godot-AI-Beehave` adds a focused MCP tool family for building [Beehave](https://github.com/bitbrain/beehave)
behaviour trees (`BeehaveTree`, selector/sequence composites, action/condition leaves) from an AI client.
It is authored in C# with `[AiToolType]` / `[AiTool]` (the same model as Unity-MCP and the core Godot-MCP
addon) and shipped as a **source-only NuGet package** that compiles inside any consumer's Godot project
against the consumer's own GodotSharp — no bundled Godot, no version lock. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template).

**This is an addon-dependent ("Class B") extension.** It wraps the third-party Beehave addon, which it
does **not** bundle and does **not** declare as a package dependency — Beehave's classes are referenced
only by name and resolved at runtime. Every editor tool is **presence-gated**: when Beehave is not
installed, the tool returns a clean structured `installed: false` result (with an install hint) instead of
crashing. See the prerequisite below.

## Required prerequisite — install the Beehave addon yourself

This extension drives the **Beehave** addon but does **not** ship it. Before the editor tools can do
anything, install Beehave into your own project:

- Install **Beehave** from the **Godot Asset Library**, or download the pinned release `v2.9.2` from the
  upstream repo [`bitbrain/beehave`](https://github.com/bitbrain/beehave/releases/tag/v2.9.2) and unzip it
  so the addon lives at `addons/beehave/`.
- **Enable it** under **Project Settings → Plugins**. Beehave registers GDScript `class_name` types
  (e.g. `BeehaveTree`) in the global script-class list; the tools detect that to gate themselves.

> **Attribution & disclaimer.** Beehave is © its authors under the **MIT** license; this extension is
> **not affiliated** with or endorsed by it. The addon is your project's own runtime dependency — this
> package never vendors, downloads, or redistributes it.

## Tools

| Tool | Kind | Description |
| --- | --- | --- |
| `beehave-defaults` | pure-managed | Return a recommended behaviour-tree skeleton description (composite/leaf taxonomy + wiring tips); needs no addon. |
| `beehave-create-tree` | editor (`#if TOOLS`) | Instantiate a `BeehaveTree` node under an actor node in the edited scene and wire its `actor`/`blackboard`. |
| `beehave-add-composite` | editor (`#if TOOLS`) | Add a `SelectorComposite`/`SequenceComposite` (chosen by arg) under a tree or composite node. |
| `beehave-add-leaf` | editor (`#if TOOLS`) | Add an `ActionLeaf`/`ConditionLeaf` placeholder under a composite (a scaffold — you attach the behaviour script). |
| `beehave-get-tree` | editor (`#if TOOLS`) | Dump the structure (node types + parent/child) under a named `BeehaveTree` (read-only). |

Pure-managed tools (no Godot native API) live under `src/Godot-AI-Beehave/Runtime/` and are CI-unit-tested;
editor-driving tools live under `Editor/` behind `#if TOOLS`, presence-gate Beehave, and marshal every
Godot call onto the editor main thread via `MainThread.Instance.Run(...)`.

## Install (in a consumer Godot project)

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon **and** the Beehave addon
(see the prerequisite above). Then add this extension by any of:

- **Extensions dock** — pick it inside the Godot editor (Install → adds the `<PackageReference>` → rebuild).
- **CLI** — `godot-cli install-extension com.IvanMurzak.Godot.MCP.Beehave`.
- **By hand** — add `<PackageReference Include="com.IvanMurzak.Godot.MCP.Beehave" Version="x.y.z" />`
  to the consumer `.csproj` and rebuild.

After a rebuild the `[AiToolType]` tool family is auto-discovered — no registry edit.

## Build & test (no Godot binary needed)

`Godot.NET.Sdk` pulls GodotSharp from NuGet, so the package builds and unit-tests headless — and, because
Beehave is referenced only by name, it **builds with the addon absent**:

```bash
dotnet build src/Godot-AI-Beehave/Godot-AI-Beehave.csproj            # compiles tools (Godot API resolves; Beehave NOT needed)
dotnet test  tests/Godot-AI-Beehave.Tests/Godot-AI-Beehave.Tests.csproj   # pure-managed unit tests
dotnet pack  src/Godot-AI-Beehave/Godot-AI-Beehave.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Beehave-Testbed.csproj                         # consumer build = source-injection proof
```

The testbed build proves the source-injection recipe: the package's `.cs` are injected as `<Compile>`
items into the consumer and compile against the consumer's own GodotSharp. CI runs this across a
multi-Godot-version matrix; an end-to-end leg additionally boots real headless Godot, installs the core
addon **plus the pinned Beehave addon**, and calls each tool — asserting the presence gate reports
`installed: true` once Beehave is present.

## Docs

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece).
- `docs/ci.md` — workflows, the version gate, the multi-Godot matrix, required secrets.
- `CLAUDE.md` — maintainer notes (addon-interop + presence-gate model).

## Publish

Source-only, version-gated release (see `docs/ci.md`): configure NuGet trusted publishing (OIDC), bump
`<Version>` (`commands/bump-version.ps1 -NewVersion x.y.z`), merge to `main`; `release.yml` runs the
full matrix, publishes the package to NuGet, and cuts an atomic GitHub Release.

License: **Apache-2.0** (this extension). The wrapped Beehave addon is MIT and is the consumer's own dependency.
