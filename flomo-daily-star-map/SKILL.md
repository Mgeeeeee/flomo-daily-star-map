---
name: flomo-daily-star-map
description: >
  Generate an interactive force-directed star map from flomo notes.
  Trigger when user mentions: "flomo星图", "每日星图", "笔记星图",
  "flomo每日回顾星图", "今日flomo星图", "flomo force graph",
  "flomo可视化", "看看笔记之间的关联", "把flomo笔记画成图".
  Requires flomo MCP connection.
---

# flomo 每日星图

你是一位读者，也是一位制图师。

读取用户的 flomo 笔记，理解它们，为它们命名主题、发现关联，然后把这一切画成一张星图。

## 你的工作

### 1. 采集

通过 flomo MCP 获取原始数据：

- 调用 `memo_search`（不带关键词，`limit=20`）获取最近 20 条笔记，作为 **seed**
- 对每条 seed 调用 `memo_recommended(id, limit=10)`，获取相关笔记
- 合并、按 id 去重

你手上现在有大约 80–160 条笔记。

### 2. 理解

这是你作为模型最重要的工作，没有脚本替你做这件事。

**聚类**：读完所有笔记后，给它们分组。

- 以标签为线索，但不要只按标签机械分组——同一个 `#随想` 下可能有哲学思辨、也有生活碎片，你要读出区别
- 聚类名称像人会起的主题名，不是标签的罗列
- 通常会形成 5–10 个簇
- 每个簇给一个颜色

**节点判断**：

- seed 节点更大、更醒目
- 被多个 seed 同时推荐的 memo 是桥接节点，也应该稍大
- 其余是普通叶子节点

**连线**：

- `recommendation` 边：seed → 推荐 memo 的真实关系，全部保留
- `semantic` 边：你认为语义相近但没有推荐关系的节点之间，可以少量补充（不超过 recommendation 边的 20%）
- 克制。不要画成毛线球

### 3. 制图

读取 `templates/viewer.html` 模板。

这是一个完整的 HTML 页面，只有两个占位符需要你替换：

| 占位符 | 替换为 |
|--------|--------|
| `__PAGE_TITLE__` | 页面标题，如 `flomo 每日星图` |
| `__GRAPH_DATA__` | 你构造的完整 JSON 数据 |

模板里已经包含了所有样式、交互逻辑、力导向算法。你不需要写 CSS、不需要写 D3 代码、不需要改模板的任何其他部分。你唯一的工作就是把数据填进去。

### 4. 交付

把替换完成的 HTML 交给用户。怎么交付取决于你的运行环境——写文件、放到 artifact、直接输出代码块——skill 不管这个。

附一句简短说明：节点数、连线数、聚类数，以及你觉得值得提一句的发现。不需要长篇大论。

## 数据结构

```json
{
  "meta": {
    "title": "笔 记 星 图",
    "subtitle": "flomo · 每日回顾 · 语义星图",
    "page_title": "flomo 每日星图",
    "generated_at": "2026-03-31T22:00:00"
  },
  "clusters": [
    {
      "id": "c1",
      "label": "深夜随想",
      "color": "#6ec8c8"
    }
  ],
  "nodes": [
    {
      "id": "abc123",
      "type": "seed",
      "cluster": "c1",
      "cluster_label": "深夜随想",
      "content": "今天突然想到一个问题，为什么越忙越不想动？\n\n#随想",
      "tags": ["随想"],
      "created_at": "2026-03-26"
    }
  ],
  "links": [
    {
      "source": "abc123",
      "target": "def456",
      "kind": "recommendation"
    }
  ]
}
```

**字段说明**：

- `id`：使用 flomo 返回的原始 memo id
- `type`：`"seed"` 或 `"recommended"`
- `content`：笔记完整正文，显示在悬浮提示中
- `kind`：`"recommendation"` 或 `"semantic"`
- 不需要填 `size`、`color` 等字段，模板会自动根据 type、cluster、被推荐次数计算

**clusters 的颜色**：自己选。配色建议：

- 深度思考 / 长文反思类 → 暖金 `#e8a838`
- 碎片随想 / 日常感悟 → 青蓝 `#6ec8c8`
- 工具 / 方法论 → 紫 `#b07ee8`
- 生活记录 / 瞬间捕捉 → 绿 `#5aad5a`
- 阅读 / 摘录 → 珊瑚 `#e87070`
- 情绪 / 自我觉察 → 粉 `#e88fc0`
- 创意 / 待办 → 蓝 `#50b0e0`
- 其他 → 灰 `#888`

以上只是示例色板，请根据用户笔记的实际主题自行命名和配色。

## 硬约束

- `nodes` 和 `links` 必须非空
- recommendation 边全部保留
- semantic 边 ≤ recommendation 边的 20%
- 不要编造不存在的笔记
- 不要生成空白页面
- 如果 flomo MCP 不可用，直接告知用户，不要变通

## 你不需要做的

- 不需要写任何前端代码
- 不需要处理 D3 / 力模拟 / 样式 / 交互
- 不需要关心用户用什么设备、什么模型
- 模板已经处理了一切渲染逻辑
