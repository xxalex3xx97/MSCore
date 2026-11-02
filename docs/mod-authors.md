---
title: Mod authors
nav_order: 3
has_children: true
---

# Mod authors

MSCore gives your UE4SS Lua mod and your native mod shared helpers, so every mod uses one copy.

- [Quickstart]({{ site.baseurl }}/mod-authors/quickstart.html): the first working mod.
- [Lua API reference]({{ site.baseurl }}/mod-authors/lua-api.html): every Lua function.
- [Native C API reference]({{ site.baseurl }}/mod-authors/native-api.html): every `MSCore_*` export.
- [Publishing a mod]({{ site.baseurl }}/mod-authors/publishing.html): zip layout, manifest, release checklist.

Each UE4SS mod has its own Lua state. Each Lua mod must `require` its own copy of `mscore.lua` from `Mods\MSCore\Scripts`. Do not share Lua tables between mods.
