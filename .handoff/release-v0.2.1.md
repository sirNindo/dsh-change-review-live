# dsh-change-review-live v0.2.1

> 修复版：diff 面板的语法高亮在遇到「标签内含 `@` 属性」的文件时会死循环，把整个页面卡死（打开 `.vue` / `.html` / `.svelte` 文件的 diff 必现）。

## 本版修复

- **打开 `.vue` / `.html` / `.svelte` 文件的 diff 不再卡死**：`hlScanMarkup`（HTML/Vue/Svelte 高亮）在标签内遇到 `@` 开头的属性（`@click` 等）时，属性名扫描**一个字符都没推进**——`q` 原地不动 + 无限追加空片段 → 内存暴涨、页面失去响应。实测真实 `OutputBlock.ce.vue`（131 行）**8.4 秒打爆 4GB 堆**；抽出源码对单行 `<a @click="x">` 也能在 **195ms 打爆 128MB 堆**。修法：属性名扫描从 `q + 1` 起步、并让 `@` 参与推进（`@click` 现在被正确着色为属性色）
- **加了两处迭代守卫兜底**：这两个循环今后任何一处「索引不推进」都只会被截断，不再把整页拖死（把上限压到极限实测 **0ms 返回受限结果**而不是崩溃）；正常输入不受影响（单行 45000 字符 / 5000 个标签实测 3ms、token 一个不少）

## 影响范围 / 升级

本版**只改浏览器端 bundle（`lib/client.js`）**：Host（记录逻辑、HTTP 路由）与状态文件格式**没有任何变化**，升级后**刷新页面**即可生效，**不需要重启 DSH Desktop**，已有记录也不用重建。

```sh
dsh plugin --profile desktop add github:sujingkpo/dsh-change-review-live
```

装到 `dsh web` 的 profile 时把 `desktop` 换成 `web`。

完整说明见 [README.zh.md](https://github.com/sujingkpo/dsh-change-review-live/blob/main/README.zh.md) / [README.md](https://github.com/sujingkpo/dsh-change-review-live/blob/main/README.md)。
