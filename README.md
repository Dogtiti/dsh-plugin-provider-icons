# dsh-plugin-provider-icons

给 DeepSeek Harness 的模型选择器加上**供应商标识**：模型名左边显示该供应商的品牌 logo。

模型菜单、输入框里的当前模型、每一项可选模型都会显示，**不改动任何原生交互**——点击、下拉、推理等级全按官方行为走。

## 效果

```
newapi
  🎨 glm-5.3
  🎨 deepseek-v4.1-flash
newapi (responses)
  🎨 gpt-6-luna
zai-coding-cn
  Z  GLM-5.3
google-vertex
  G  Gemini 3.8 Flash
opencode-go
  ▪  DeepSeek V4.1 Flash      ← 单色品牌保持单色
```

## 安装

```bash
dsh plugin install dsh-plugin-provider-icons
```

或在 Plugins 页安装后刷新页面。

纯客户端插件，无需重启 Host。

## 覆盖的供应商

24 个品牌标全部内联为 data URI，装完即可用，不请求任何外部资源：

`deepseek` `openai` `anthropic` `google` `azure` `zai` `moonshot` `xai` `groq` `mistral` `qwen` `minimax` `together` `openrouter` `cerebras` `nvidia` `github-copilot` `huggingface` `fireworks` `baseten` `cloudflare` `vercel` `opencode` `ant-ling`，外加 `new-api`（取自其开源仓库的 logo）。

图标来源：[lobehub/lobe-icons](https://github.com/lobehub/lobe-icons)（MIT）。优先使用各品牌的彩色版本以保留品牌色；只有单色版本品牌（如 OpenAI、Anthropic、xAI、Moonshot、Z.ai）保持单色。

## 匹配顺序

```
① 精确匹配 provider id   → newapi / deepseek-account / zai-coding-cn …
② 别名表                → newapi-responses → newapi
③ 前缀匹配              → newapi-*、deepseek-*、zai-* …（拆协议或分环境时不用改表）
④ 都没命中              → 灰色圆点
```

所以你把同一个网关按协议拆成 `newapi` 和 `newapi-responses` 时，两边都会自动带上 new-api 的标。

## 实现方式

模型名是一个**由原生插件自己声明的内联槽位**（`conversation.input.model.label`），外部插件注入不进去。但包裹它的 `<span>` 上带着 `data-provider-id`，所以本插件改走 CSS：

```css
[data-provider-id="newapi"]::before {
  background-image: url("data:image/png;base64,…");
  width: 13px; height: 13px;
}
```

不替换任何组件、不碰 React 树，因此不会和 Harness 后续版本的选择器改动冲突。

## 已知限制

- 只能加"装饰"，改不了标签文字；
- 单色品牌的 logo 用中性灰（`#6b7280`），深色模式下同样可见；
- 自定义网关如果不在表里，显示灰色圆点——可以在 `client.js` 的 `LOGOS` 里加一条。

## 相关插件

| 插件 | 说明 |
| --- | --- |
| [dsh-plugin-composer-predict](https://github.com/Dogtiti/dsh-plugin-composer-predict) | 预测下一条指令并显示为占位文字，`Tab` 采纳 |
| [dsh-plugin-composer-history](https://github.com/Dogtiti/dsh-plugin-composer-history) | 输入框中按 `↑` / `↓` 回填历史消息 |
| [dsh-plugin-subagent-bridge](https://github.com/Dogtiti/dsh-plugin-subagent-bridge) | 让 Codex 通过本机 MCP 把 DSH 当 subagent 用（spawn / wait / followup），反向也能把任务派给 Codex |
| [dsh-plugin-usage-panel](https://github.com/Dogtiti/dsh-plugin-usage-panel) | 在模型选择器旁显示供应商套餐额度余量（目前 GLM Coding Plan），点开是明细浮层 |
| **dsh-plugin-provider-icons**（本仓库） | 在模型名旁显示供应商品牌 logo |
