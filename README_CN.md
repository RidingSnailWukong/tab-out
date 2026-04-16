# Tab Out

**掌控你的标签页。**

Tab Out 是一款 Chrome 扩展，用一个清爽的仪表盘替换你的新标签页，展示所有打开的标签页。标签页按域名自动分组，首页类标签（Gmail、X、LinkedIn 等）会被单独归类。关闭标签时还有令人愉悦的音效 + 彩纸特效。

无需服务器。无需账号。无需外部 API 调用。只是一个 Chrome 扩展。

---

## 使用 AI 编程助手安装

将这个仓库发给你的 AI 编程助手（Claude Code、Codex 等），然后说 **"帮我安装这个"**：

```
https://github.com/RidingSnailWukong/tab-out
```

助手会一步步引导你完成安装，大约只需 1 分钟。

---

## 功能特性

- **一眼看清所有标签页** — 整洁的网格布局，按域名自动分组
- **首页分组** — 将 Gmail 收件箱、X 首页、YouTube、LinkedIn、GitHub 首页归入一个卡片
- **关闭标签有仪式感** — 音效 + 彩纸飞舞
- **重复检测** — 发现你打开了相同的页面时会标记提醒，一键去重
- **点击标签直接跳转** — 跨窗口切换，不会打开新标签
- **稍后再看** — 关闭前可以把标签加入待办清单
- **Localhost 分组** — 端口号显示在标签旁边，轻松区分不同的本地项目
- **可展开的分组** — 默认显示前 8 个标签，点击 "+N 个更多" 展开
- **100% 本地运行** — 你的数据永远不会离开你的电脑
- **纯 Chrome 扩展** — 无服务器、无 Node.js、无 npm，加载扩展即可使用

---

## 手动安装

**1. 克隆仓库**

```bash
git clone https://github.com/RidingSnailWukong/tab-out.git
```

**2. 加载 Chrome 扩展**

1. 打开 Chrome，访问 `chrome://extensions`
2. 开启右上角的 **开发者模式**
3. 点击 **加载已解压的扩展程序**
4. 选择克隆仓库中的 `extension/` 文件夹

**3. 打开新标签页**

你会看到 Tab Out。

---

## 工作原理

```
打开一个新标签页
  -> Tab Out 展示你的所有标签页，按域名分组
  -> 首页类标签（Gmail、X 等）在顶部单独成组
  -> 点击任意标签标题跳转过去
  -> 关闭不需要的分组（音效 + 彩纸）
  -> 关闭前可以把标签保存到"稍后再看"
```

所有逻辑都在 Chrome 扩展内部运行。没有外部服务器，没有 API 调用，不会向任何地方发送数据。保存的标签存储在 `chrome.storage.local` 中。

---

## 技术栈

| 类别 | 技术方案 |
|------|---------|
| 扩展 | Chrome Manifest V3 |
| 存储 | chrome.storage.local |
| 音效 | Web Audio API（合成生成，无音频文件） |
| 动画 | CSS 过渡 + JS 彩纸粒子效果 |

---

## 许可证

MIT

---

由 [RidingSnailWukong](https://github.com/RidingSnailWukong) 构建
