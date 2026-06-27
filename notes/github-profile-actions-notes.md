# GitHub 个人主页 Actions 维护笔记

> 记录 `NOBB2333/NOBB2333` 这个 GitHub Special Repository 中 Actions 的坑、配置路径和改造思路。

---

## 仓库定位

- `NOBB2333/NOBB2333` 是 GitHub 个人主页仓库。
- 主页内容主要靠 `README.md` + 几个 Actions 自动生成 SVG 图表。
- 仓库本身以后不常手动修改，主要展示其他项目的活跃度。

---

## 现有 Actions

| Action | 文件 | 运行频率 | 生成物 | 提交目标 |
|--------|------|----------|--------|----------|
| Metrics | `.github/workflows/Metrics.yml` | 每天 1 次（UTC 00:00） | `metrics/github-metrics.svg` | `output` 分支 |
| GitHub-Profile-3D-Contrib | `.github/workflows/profile-3d.yml` | 每天 1 次（UTC 18:00） | `profile-3d-contrib/*.svg` | `output` 分支 |
| Generate Snake | `.github/workflows/Generate Snake.yml` | 每天 1 次（UTC 00:00） | `github-contribution-grid-snake*.svg` | `output` 分支 |

---

## 关键配置：Actions 写权限

`profile-3d.yml` 报错 403 的根本原因是：

> `GITHUB_TOKEN` 默认只有读权限，推代码会失败。

### 解决路径

1. 打开仓库设置：
   ```
   https://github.com/NOBB2333/NOBB2333/settings/actions
   ```

2. 找到 **Workflow permissions**。

3. 选择：
   > ✅ **Read and write permissions**

4. 点击 **Save**。

### 注意

- 不是改个人 `Settings -> Developer settings -> Personal access tokens`。
- 也不是提升某个 PAT 的 scope。
- 是改仓库级别的 Actions 默认权限。
- 另外要在 workflow 文件里声明写权限：
  ```yaml
  permissions:
    contents: write
  ```
- 两个条件同时满足，自动提交才不会 403。

---

## Git 历史污染问题

### 现状

- `Metrics` 和 `profile-3d` 每次运行都会在 `main` 分支新增一条 `generated` commit。
- 一天 2 条，一年就是 700 多条 bot commit。
- `Generate Snake` 推到 `output` 分支，不污染 `main`。

### 可选改造方案

| 方案 | 做法 | 效果 |
|------|------|------|
| A | 保持现状，每天跑一次 | `main` 每天多 2 条 commit |
| B | 把生成物也改推送到 `output` 分支 | `main` 完全干净，只有手动改动 |
| C | 建立独立仓库 `NOBB2333/NOBB2333-assets` | 主页仓库完全不参与生成 |

### 方案 B 的限制

- 改到 `output` 分支后，`main` 只能看到当前引用的最新 SVG。
- 历史 SVG 需要通过 `output` 分支的 commit 历史查看。
- `lowlighter/metrics` 默认提交到当前分支，改目标分支需要额外配置。

---

## README 引用方式

目前 snake 动画已经在用 `output` 分支引用：

```markdown
![NOBB2333](https://raw.githubusercontent.com/NOBB2333/NOBB2333/output/github-contribution-grid-snake.svg)
```

如果后续把 `github-metrics.svg` 和 3D 图也放到 `output` 分支，README 里统一改成这种 `raw.githubusercontent.com` 外链即可。

---

## 时区说明

- GitHub Actions 的 `schedule` 按 UTC 执行。
- `0 0 * * *` = UTC 00:00 = 北京时间 08:00。
- `0 18 * * *` = UTC 18:00 = 北京时间次日 02:00。

---

## 待决策

- [x] 是否把 `Metrics` 和 `profile-3d` 的生成目标也改到 `output` 分支？—— 已完成
- [x] 是否保留 `Metrics.yml` 里的 `push` 触发器？—— 已移除
- [ ] 等 Actions 自动运行后，验证 `output` 分支是否正确生成文件

---

*Created: 2026-06-27*
