<div align="center">

# DenseGPT

**让 ChatGPT 更紧凑、更安静、更好扫读——视觉与语言两端同时收紧。**

[![UserCSS](https://img.shields.io/badge/UserCSS-Stylus-2f6feb?style=flat-square)](https://github.com/openstyles/stylus)
[![Release](https://img.shields.io/github/v/release/yunmin311/DenseGPT?style=flat-square&color=2f6feb)](https://github.com/yunmin311/DenseGPT/releases)
[![License](https://img.shields.io/github/license/yunmin311/DenseGPT?style=flat-square&color=2f6feb)](LICENSE)
[![JavaScript](https://img.shields.io/badge/JavaScript-none-2f6feb?style=flat-square)](densegpt.user.css)

[English](README.md) · **简体中文**

</div>

---

ChatGPT 浪费两种空间。页面太松——标题间距过大、列表松散、阅读列比窗口窄一大截；回答也太松——开场白、结尾总结、一句一段、把 Markdown 当装饰用。

DenseGPT 从两端同时解决。

| 层 | 文件 | 解决什么 |
| :--- | :--- | :--- |
| **视觉密度** | [`densegpt.user.css`](densegpt.user.css) | 一个 UserCSS 文件。收紧阅读排版，并给出真正可用的正文宽度控制。 |
| **回答密度** | [`STYLE.md`](STYLE.md) + [`presets/`](presets) | 可直接粘贴的指令，让模型自己把回答写密。 |

两层可以各用各的，一起用效果叠加。

## 目录

- [安装](#安装)
- [正文宽度](#正文宽度)
- [回答密度](#回答密度)
- [到底改了什么](#到底改了什么)
- [文档](#文档)
- [故障排查](#故障排查)
- [参与改进](#参与改进)

## 安装

1. 安装 [Stylus](https://github.com/openstyles/stylus)——Chrome、Edge 或 Firefox。
2. 打开 **[densegpt.user.css](https://raw.githubusercontent.com/yunmin311/DenseGPT/main/densegpt.user.css)**，Stylus 会提示安装。
3. 刷新 `chatgpt.com`。

> [!IMPORTANT]
> **更新请走 Stylus → 管理 → 检查更新。**
> 不要重新导入。*Write new style → Import* 每次都会**新建一条**样式；两份同时注入
> `!important` 规则，会让你的设置看起来完全没反应，而且新建的那份变量值全部回默认。

除 Stylus 外不需要任何东西：不注入 JavaScript、不发网络请求、无遥测。整个样式表五分钟能读完。

## 正文宽度

用来替代 Widescreen 插件。Stylus → DenseGPT → **Configure**。

| 宽度档 | 宽度 | 适用 |
| :--- | :--- | :--- |
| Reading | `44rem` / 704px | 长篇文字，最窄一档 |
| **Balanced** *(默认)* | `54rem` / 864px | 日常阅读 |
| Wide | `66rem` / 1056px | 代码块和表格 |
| Ultra | `78rem` / 1248px | 大屏、参考资料 |
| Custom | 滑块，36–90rem | 中间任意值 |

滑块**只在**档位选 Custom 时生效，四个推荐档一律忽略它。

每个值都包在 `min(…, calc(100vw - 3rem))` 里，窗口变窄时列宽自动收缩，不会把文字顶到边缘。它接的是 ChatGPT 自己的 `--thread-content-max-width`，所以助手消息、你的消息和输入框始终共用同一条中心轴线；又因为它作用于 `max-width`，值比可用空间大时只是失效，绝不会造成横向溢出。

[`docs/width-preview.html`](docs/width-preview.html) 是一个独立页面——双击直接打开、不用服务器——可以在不碰 ChatGPT 的情况下挑 Custom 值。

## 回答密度

CSS 让页面变密，这一层让**回答本身**变密。

[`STYLE.md`](STYLE.md) 是规范，`presets/` 是按工具编译好的版本：

| 预设 | 放在哪 |
| :--- | :--- |
| [`presets/chatgpt-custom-instructions.md`](presets/chatgpt-custom-instructions.md) | ChatGPT → 设置 → 个性化 → 自定义指令 |
| [`presets/codex.md`](presets/codex.md) | `AGENTS.md` |
| [`presets/claude-code.md`](presets/claude-code.md) | `CLAUDE.md` |

### 直接粘进 ChatGPT

设置 → **个性化** → **自定义指令** → *"你希望 ChatGPT 具备哪些特质？"*。全文 1456 字符，能塞进 1500 字符的输入框。

```text
Answer first: conclusion, then only the reasoning that changes what I do next. Never restate my question or the context I pasted.

Density over packaging. Cut filler, not content. No opening pleasantries, no closing summary, no "let me know if" — end on the last useful line.

Markdown is structure, not decoration. Headings only when the answer has 2+ real parts. Lists only for parallel enumerable items. Tables only for 2+ dimensions over 3+ rows. Bold only for the term being defined or the value being decided. Code formatting for identifiers, paths, flags, commands. Never write consecutive one-sentence paragraphs — join them. Keep visual whitespace under ~40% of the answer.

Length follows the question: a one-line question gets a one-line answer. Don't widen scope for completeness, don't append related-but-unasked knowledge, don't produce a report unless I ask for one.

Assume a university CS background and heavy hands-on AI-tool experience. Don't define standard terms. Take a position: recommend one option and say why in a clause. State uncertainty in a few words ("probably X, unverified") instead of hedging across a paragraph.

Voice: direct, plain, technical. No metaphors, no marketing verbs, no self-assessment, no enthusiasm padding. Active voice, shorter word.

No images or diagrams unless I ask.

On request these override the default: "explain in detail" = full depth; "write a report" = long form; "just the answer" = one line.
```

> 指令用英文写，因为自定义指令字段有字符上限，英文在同样字数下能表达更多规则；它照样会用中文回答你。

另有一个 593 字符的精简版，以及关于 Projects 和项目级指令的说明，都在
[`presets/chatgpt-custom-instructions.md`](presets/chatgpt-custom-instructions.md)。

其中两条起的作用最大：**不写结尾总结**、**不许连续的一句一段**。这一层是为了配合 CSS 设计的——段落节奏收紧之后，模型必须停止灌水才好看；反过来，密集的文字也只有在页面给出清晰层级时才扫得动。

## 到底改了什么

助手 markdown 上九条规则，加上宽度两条。整个样式表就这些。

| 元素 | 值 |
| :--- | :--- |
| `p` | `margin: 0.45em 0` · `line-height: 1.65` |
| `h2` | `margin: 1.15em 0 0.45em` · `font-size: 1.15em` |
| `h3` | `margin: 0.9em 0 0.35em` · `font-size: 1.05em` |
| `ul`、`ol` | `margin: 0.35em 0 0.45em` |
| `li` | `margin: 0.12em 0` |
| 行内代码 | 5% `currentColor` 底色 · `4px` 圆角 · 无边框、无阴影 |
| `blockquote` | 单条 `3px` 左线 · 极淡灰蓝底 · `6px` 圆角 · 文字颜色不变 |
| `em`、`i` | `font-weight: 500`——粗斜体仍然是粗体 |
| `hr` | 18% 的 1px 发丝线 |

**不碰的**：正文和标题颜色、链接、表格、粗体、围栏代码块（背景、语法高亮、字号全部保持 ChatGPT 原样）、侧栏、顶栏、所有消息操作控件。

**没有任何主题判断**：没有 `html.dark`、没有 `prefers-color-scheme`，也没有任何针对 `html`、`body`、`:root` 的规则。明暗主题完全继承 ChatGPT 原样。全文只有四个颜色值。

> **提高信息密度，但不降低语义清晰度。**
>
> 标题、段落、列表、行内代码、代码块、引用块必须一眼可分辨。做不到这一点的改动就是回退，无论它把页面压得多紧。

## 文档

| 文档 | 内容 |
| :--- | :--- |
| [`docs/design-principles.md`](docs/design-principles.md) | 每个数值、为什么是这个数值、哪些东西永远不许压缩 |
| [`docs/content-width.md`](docs/content-width.md) | 宽度设置全流程：档位与滑块如何组合、安装陷阱、控制台自检脚本 |
| [`docs/selectors.md`](docs/selectors.md) | 每个 DOM 钩子、它的稳定性、ChatGPT 改版后怎么修 |
| [`CHANGELOG.md`](CHANGELOG.md) | 版本历史，包括失败过什么、为什么失败 |

## 故障排查

| 现象 | 原因 |
| :--- | :--- |
| 改设置没反应 | 装了不止一份 DenseGPT。Stylus → 管理 → 删掉多余的。 |
| 没有 **Configure** 齿轮 | 元数据块没解析成功。从 raw 链接重装。 |
| 完全没有样式 | ChatGPT 改了作用域属性。见 [`docs/selectors.md`](docs/selectors.md)，改一行即可。 |
| 宽度不生效 | ChatGPT 新增了宽度变量的声明点。用 [`docs/selectors.md`](docs/selectors.md) 里的祖先链探针定位。 |

[`docs/content-width.md`](docs/content-width.md) 里有一段控制台自检脚本，上面这些都能用 computed style 直接判定，不用猜。

## 兼容性

Chrome、Edge、Firefox + Stylus。需要 `color-mix()` 和 `:has()`——Chrome 111+、Firefox 113+、Safari 16.4+。作用域仅限 `chatgpt.com`。

## 参与改进

ChatGPT 的 DOM 一直在变。选择器失效时，最有用的报告是 [`docs/selectors.md`](docs/selectors.md) 里那些探针的输出——computed style 和 bounding rect 比"看起来怎样"的描述有用得多。

间距数值是刻意定的，都在真实输出上肉眼验过。要改它们，理由得比个人偏好更硬。

## 参考

- [openstyles/stylus](https://github.com/openstyles/stylus) —— UserCSS 元数据与变量规范
- [catppuccin/userstyles](https://github.com/catppuccin/userstyles) —— 语义化选择器纪律、明暗主题结构
- [tobimori/awesome-userstyles](https://github.com/tobimori/awesome-userstyles) —— 生态概览

## 许可

[MIT](LICENSE) © yunmin311
