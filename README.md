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

## 闭环示例

```bash
curl http://127.0.0.1:3019/tunes/tune_demo/progress
curl -X POST http://127.0.0.1:3019/issues \
  -H 'Content-Type: application/json' \
  -d '{"tuneId":"tune_demo","sectionId":"section_demo_2","type":"错孔","beat":45,"lane":9,"description":"第45拍第9轨多打孔"}'
```

## 批量迁移问题

`POST /issues/migrate` 将同一曲目内的多条问题一次性迁入目标区间：

```bash
curl -X POST http://127.0.0.1:3019/issues/migrate \
  -H 'Content-Type: application/json' \
  -d '{"issueIds":["issue_demo"],"targetSectionId":"section_demo_2"}'
```

约束：

- 一次提交的所有问题与目标区间必须属于同一曲目；
- 每条问题的拍号必须落在目标区间的 `startBeat`~`endBeat` 范围内；
- 迁移成功后目标区间变为未校对（`checked=false`），原区间的校对状态保持不变；
- 跨曲目、拍号越界、目标区间或问题不存在时整批拒绝，不会产生部分落盘。
