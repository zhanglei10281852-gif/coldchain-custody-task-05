# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

运输单办理发出后自身状态已变为运输中，保温箱也已占用，但关联样本仍停留在已预留状态。请修复跨实体状态传播，保证发出操作原子地推进所有关联对象。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-05
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-05.git
- parent SHA：22d16637ad69e0d0a1f2974af7950eaf2a928860

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-05.git bug-repro
cd bug-repro
git checkout --detach 22d16637ad69e0d0a1f2974af7950eaf2a928860
go test ./internal/service -run "^TestPlanningLifecycleMovesSamplesAndContainer$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestPlanningLifecycleMovesSamplesAndContainer$" -count=1
--- FAIL: TestPlanningLifecycleMovesSamplesAndContainer (0.60s)
    service_test.go:189: in transit batch = {ID:sample_c5cd14b14d831ef0e3661824 StudyID:study_e8b89e8cc482448d28dfa482 OriginSiteID:site_bfc340420ec57566c5890800 ExternalRef:EXT-1 SpecimenType:plasma VialCount:2 VolumeMilliLit:100 State:reserved ExpiresAt:2026-08-20 08:00:00 +0000 UTC ShipmentID:ship_fd60fe4f08b3e6a343ccd8ad QuarantineNote: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:4}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	0.602s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestPlanningLifecycleMovesSamplesAndContainer$" -count=1
--- FAIL: TestPlanningLifecycleMovesSamplesAndContainer (1.33s)
    service_test.go:189: in transit batch = {ID:sample_5b6ba3808b66801b0f0ac9e3 StudyID:study_b30c77264bce9c0eddc850f2 OriginSiteID:site_11935484fa1ddb5400c81b31 ExternalRef:EXT-1 SpecimenType:plasma VialCount:2 VolumeMilliLit:100 State:reserved ExpiresAt:2026-08-20 08:00:00 +0000 UTC ShipmentID:ship_c2c08169dfcb2d2ec9422cf0 QuarantineNote: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:4}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	1.532s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令 go test ./internal/service -run ^TestPlanningLifecycleMovesSamplesAndContainer$ -count=1 必须由修复前失败变为修复后通过；相关包与 go test ./... -count=1 全量回归通过，回退 gold 关键修改后定向命令重新失败。
