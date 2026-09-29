# 运行记录与快照

所有时间为带时区的真实 ISO 8601 时间。示例中的尖括号是说明，实际记录必须替换为真实值；不可获得的字段使用 JSON `null`，不得保留占位符。

## 运行记录

保存为工作区 `state/runs/<run_id>.json`：

```json
{
  "run_id": "<本次唯一标识>",
  "started_at": "<开始时间>",
  "finished_at": "<结束时间>",
  "status": "completed",
  "profile_path": "learning-profile.md",
  "report_path": "reports/<run_id>.md",
  "snapshot_path": "state/stars/<run_id>.json",
  "candidate_count": 0,
  "selected_repositories": [],
  "repository_code_executed": false,
  "limitations": []
}
```

状态含义：`completed` 为本次约定范围完成；`partial` 为已有产物但仍有获取、核查或交付缺口；`failed` 为无法形成所需产物；`skipped` 为无新相关发现。不存在的 `report_path`、`profile_path` 或 `snapshot_path` 使用 `null`，不能指向不存在的文件。

选中仓库使用对象记录 `full_name`、`commit_sha`、`local_path`、`source_mode`。`source_mode` 可以是 `clone_no_checkout`、`existing_checkout` 或 `docs_only`；未读源码时 commit 不明则为 `null`。所有本地路径尽量使用工作区相对路径。

## Star 快照

保存为工作区 `state/stars/<run_id>.json`：

```json
{
  "run_id": "<与运行记录一致>",
  "repositories": [
    {
      "repository_id": null,
      "full_name": "<已确认的 owner/repo>",
      "source_url": "<实际读取的官方 URL>",
      "source_kind": "github_api",
      "observed_at": "<该条数据的实际获取时间>",
      "stars_total": null,
      "previous_observed_at": null,
      "stars_delta": null,
      "interval_hours": null
    }
  ]
}
```

`source_kind` 区分 `github_api`、`github_html`、`trending`。Trending 的页面增量另存 `trending_delta` 和 `trending_window`，不得覆盖自己的 `stars_delta`。

计算差分前确认同一仓库身份、当前与历史都取得真实累计数、前次时间早于本次。优先按稳定 ID 匹配；只有 ID 不可得且身份仍可确认时才按规范仓库名匹配，并记录此限制。遇到重命名不能只按名字把同一 ID 当成新仓库。

无有效基线时 `previous_observed_at`、`stars_delta`、`interval_hours` 均为 `null`。有基线时按实际时间计算间隔，负数差分合法。不得把候选集合的差分称为全站日增统计。

每次创建新文件，不覆盖历史记录。不将合成测试数据、文章示例中的数字或未经重新核实的历史网页数字写入真实采集快照。

## 图文、网页与定时阶段

原研究记录格式继续有效；只有执行相关阶段才增加字段。仅将旧报告整理成文章时不创建虚假的新 Star 快照，`snapshot_path` 为 `null`，`source_report_paths` 指向本次实际采用的既有报告。

```json
{
  "requested_stages": ["article"],
  "source_report_paths": [],
  "article_paths": [],
  "diagram_paths": [],
  "web_delivery": {
    "status": "not_requested",
    "url": null,
    "screenshot_path": null,
    "confirmation_evidence": null
  },
  "automation": {
    "status": "not_requested",
    "id": null,
    "schedule": null,
    "timezone": null
  }
}
```

示例数组须替换为实际产物；路径不存在就不写入数组。`confirmation_evidence` 只记保存/发布的最小界面证据，不记录敏感页面全文。

网页状态区分 `not_requested`、`prepared`、`filled`、`draft_saved`、`published`、`saved_unknown`、`blocked`。只有页面确认的状态才记为保存/发布；看见文章 ID 不足以判断可见范围。

自动化状态区分 `not_requested`、`prepared`、`created`、`updated`、`blocked`。只有工具确认创建/更新且返回真实 ID，才使用对应成功状态；`schedule`、`timezone` 记录实际计划，未能核实的字段为 `null` 并说明限制。

不同阶段分别记账：图文完成不代表网页已保存，研究成功不代表定时任务已建立。真实运行记录留在个人工作区，不进入 Skill 分享包。
