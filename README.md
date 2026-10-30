# linxh-skills

琳小航AI 自己日常在用的 Claude Code skill，整理出来分享。

## 里面有什么

| skill | 用途 |
|---|---|
| [linxh-extension-review](skills/linxh-extension-review) | 想做浏览器插件、油猴脚本时，先帮你把想法评估一遍再动手写代码。分五段走：先问清需求，再从功能、做法、难点、风险、有没有更省事的替代这五个方面评估，你拍板后出执行方案，确认了才开始写。适合不会编程的人用，每段都有大白话解释。 |

## 安装

把 `skills/` 下面想要的文件夹复制到 `~/.claude/skills/` 里：

```bash
git clone https://github.com/jedeeai/linxh-skills.git
cp -r linxh-skills/skills/linxh-extension-review ~/.claude/skills/
```

重开一个 Claude Code 窗口，跟它说「我想做个 Chrome 插件，功能是……」就会自动触发。

## 用得更顺手

`linxh-extension-review/references/reusable-assets.md` 是一张空的登记表。把你以前做过的插件项目填进去（路径、架构、能复用什么、踩过什么坑），评估新插件时它会优先建议复用旧项目，能省不少时间。

不想改这份公开模板，可以在同一个文件夹里新建 `_personal-assets.md`，按同样格式写你自己的清单，skill 会优先读它。

## 许可

MIT
