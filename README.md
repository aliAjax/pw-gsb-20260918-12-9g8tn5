# 手摇风琴纸带打孔API

纯后端零依赖Node服务，使用 `data/db.json` 持久化曲目、纸带区间和试奏问题。

## 启动

```bash
PORT=3019 node server.js
```

## 主要接口

- `GET /health`
- `GET /tunes`
- `POST /tunes`
- `GET /tunes/:id/progress`
- `GET /tunes/:id/sections`
- `POST /tunes/:id/sections`
- `GET /tunes/:id/unchecked-sections`
- `PATCH /sections/:id/check`
- `GET /issues?tuneId=&status=`
- `POST /issues`
- `POST /issues/migrate`
- `PATCH /issues/:id/status`

## 批量迁移问题

`POST /issues/migrate`，一次提交多条迁移，规则：

- 请求体：`{"items":[{"issueId":"...","targetSectionId":"..."}]}`，`items` 必须是非空数组
- 整批所有问题必须属于**同一曲目**，且每条的目标区间必须存在并属于该曲目
- 问题拍号必须落在目标区间的 `[startBeat, endBeat]` 内
- 跨曲目、拍号越界、目标/问题不存在、同一批次重复迁移同一条问题时，**整批拒绝且不写盘**
- 迁移只修改问题的 `sectionId`（移动而非复制）；目标区间无论此前是否已校对都会变为未校对
- 原区间迁出后不会被自动标记为已校对（即使已无未解决问题）
- 写接口整体串行执行，并发迁移不会产生重复或丢失

响应：`{"data":{"tuneId","migratedCount","migratedIssues":[...],"targetSections":[...]}}`。

## 闭环示例

```bash
curl http://127.0.0.1:3019/tunes/tune_demo/progress
curl -X POST http://127.0.0.1:3019/issues \
  -H 'Content-Type: application/json' \
  -d '{"tuneId":"tune_demo","sectionId":"section_demo_2","type":"错孔","beat":45,"lane":9,"description":"第45拍第9轨多打孔"}'
```
