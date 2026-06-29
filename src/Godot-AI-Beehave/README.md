# Beehave Tools

AI MCP tools for authoring **Beehave** behaviour trees in the Godot editor.

A **source-only** MCP tool extension for [Godot-MCP / AI Game Developer](https://github.com/IvanMurzak/Godot-MCP).
The package ships C# source (no compiled DLL, no bundled Godot) that compiles inside your Godot project
against your own GodotSharp, so it never locks you to a Godot version.

This is an **addon-dependent** extension: it wraps the third-party
[Beehave](https://github.com/bitbrain/beehave) addon, which it does **not** bundle. Every editor tool is
**presence-gated** — when Beehave is not installed it returns a structured `installed: false` result with
an install hint instead of crashing.

## Required prerequisite — install the Beehave addon yourself

This package drives the **Beehave** addon but does not ship it. Install it into your own project first:

- Install **Beehave** from the **Godot Asset Library**, or download the pinned release `v2.9.2` from
  [`bitbrain/beehave`](https://github.com/bitbrain/beehave/releases/tag/v2.9.2) and unzip it so the addon
  lives at `addons/beehave/`.
- **Enable it** under **Project Settings → Plugins**.

> Beehave is © its authors under the **MIT** license; this extension is **not affiliated** with or
> endorsed by it. The addon is your project's own runtime dependency — this package never vendors,
> downloads, or redistributes it.

## Install

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon **and** the Beehave addon
(above) in your Godot C# project.

```bash
# via the godot-cli (resolves from the shared catalog, edits your .csproj, rebuilds)
godot-cli install-extension com.IvanMurzak.Godot.MCP.Beehave

# …or add the reference manually and rebuild:
#   <PackageReference Include="com.IvanMurzak.Godot.MCP.Beehave" Version="0.1.0" />
```

…or pick it from the **Extensions** dock inside the Godot editor.

After a rebuild, the extension's `[AiToolType]` tool families are auto-discovered — no registry edit.

## Tools

| Tool | Description |
| --- | --- |
| `beehave-defaults` | Pure-managed — a recommended behaviour-tree skeleton description (needs no addon). |
| `beehave-create-tree` | Editor — instantiate a `BeehaveTree` under an actor node and wire `actor`/`blackboard`. |
| `beehave-add-composite` | Editor — add a `SelectorComposite`/`SequenceComposite` under a tree or composite. |
| `beehave-add-leaf` | Editor — add an `ActionLeaf`/`ConditionLeaf` scaffold under a composite. |
| `beehave-get-tree` | Editor (read-only) — dump the structure under a named `BeehaveTree`. |

License: Apache-2.0 (this extension). The wrapped Beehave addon is MIT and is the consumer's own dependency.
