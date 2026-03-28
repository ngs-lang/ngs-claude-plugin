# NGS Claude Code Plugin

*EXPERIMENTAL*

[Claude Code](https://claude.ai/code) skills for [Next Generation Shell (NGS)](https://ngs-lang.org).

## Skills

| Skill | Description |
|-------|-------------|
| `ngs:write` | Write NGS code |
| `ngs:bash2ngs` | Convert Bash to NGS |
| `ngs:explain` | Explain NGS syntax and concepts |

Skills are auto-invoked by Claude when relevant (e.g. when working with `.ngs` files or asking NGS questions). You can also invoke them explicitly via the slash command shown in the skill list.

## Install

```
/plugin marketplace add ngs-lang/ngs-claude-plugin
/plugin install ngs@ngs-claude-plugin
/reload-plugins
```

## Update

```
/plugin marketplace update ngs-claude-plugin
```

Then, if you see that the plugin version is still not what you expect:
```
/plugin uninstall ngs@ngs-claude-plugin
/plugin install ngs@ngs-claude-plugin
/reload-plugins
```
