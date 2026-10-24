# 实验 1 记录：第 8 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 8 组 |
| 仓库地址 | <https://github.com/aCannedFish/ops-lab1-8> |
| 代码载体 | 统一示例 `enroll-conflict`（Go） |
| runner 与语言版本 | `ubuntu-24.04`，Go `1.26.9`；[ci 第 3 次运行](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38353330057)成功，生成了 artifact `enroll-conflict-d24ae3fadbb46bceed0a8d8556e2f8a598109187-3` |
| 合并方式 | squash（仓库仅允许此方式；[PR #1](https://github.com/aCannedFish/ops-lab1-8/pull/1) 合并后对应提交 `d24ae3fadbb46bceed0a8d8556e2f8a598109187`） |

## 2. 任务 0：本地基线

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | 在 Linux 主机 `ecs-249f` 的 `/home/ubuntu/lab1-local` 中克隆[课程模板仓库](https://github.com/zilinglius/operations-2026-lab1-sample)，执行 `make package VERSION=1.0.0 BUILD_NO=1`。两次构建均设置 `GOMAXPROCS=1`、`GOFLAGS=-p=1`、`GOMEMLIMIT=256MiB`。实际输出：<br>`name=enroll-conflict`<br>`version=1.0.0`<br>`build_no=1`<br>`run_id=local`<br>`commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`go=go1.26.9` |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | 将 `conflict.go` 第 1 行改为 `package main // task0 baseline observation`，不提交，以相同参数再次打包。`BUILD_INFO` 没有变化，二进制 sha256 发生变化。两次 `sha256sum` 的实际输出：<br>第一次：`f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e  dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict`<br>第二次：`fd9e4cdfb5dba94309d1eb422110fa4e4217bce6cd13a5c8e22048f5ca1fa098  dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict` |
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | 第一次：<br>`build vcs=git`<br>`build vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`build vcs.time=2026-10-12T16:26:07Z`<br>`build vcs.modified=false`<br>第二次：<br>`build vcs=git`<br>`build vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`build vcs.time=2026-10-12T16:26:07Z`<br>`build vcs.modified=true`<br>仅 `vcs.modified` 从 `false` 变为 `true`，表示第二次构建包含未提交的工作区改动。虽然只修改了注释，这项内嵌构建信息仍使二进制内容及哈希发生变化。 |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | 不能完整回答。`name`、`version` 能标识包及版本，`go` 记录工具链版本；其中没有记录构建主机和构建时间，`go version -m` 中的 `vcs.time` 记录的也是提交时间。`commit` 取自本地 HEAD，第二次构建时仍为 `7b63b41c13f061439997b0e6e4eaa2af922a7aa1`，没有反映未提交的注释改动。因此，仅凭该字段会误以为构建时使用的源码与该提交完全一致；`BUILD_INFO` 本身没有标记工作区是否有未提交的改动。 |

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | [main-merge-gate](https://github.com/aCannedFish/ops-lab1-8/rules/24975603)：状态为 Active，作用于默认分支 `main`，Bypass list 为空，管理员也不能绕过。启用 Restrict deletions、Block force pushes、Require a pull request before merging（Required approvals：1）、Require status checks to pass（必需检查：`ci`，来源为 GitHub Actions；未要求分支在合并前更新到最新主干）。 |
| 组长直接 `git push origin main` 被拒的终端输出 | 使用组长 `aCannedFish` 账号（仓库管理员），创建一个空提交后执行 `git push origin main`，退出码为 1。实际输出：<br>`remote: error: GH013: Repository rule violations found for refs/heads/main.`<br>`remote: Review all repository rules at https://github.com/aCannedFish/ops-lab1-8/rules?ref=refs%2Fheads%2Fmain`<br>`remote:`<br>`remote: - Changes must be made through a pull request.`<br>`remote:`<br>`remote: - Required status check "ci" is expected.`<br>`remote:`<br>`To github.com:aCannedFish/ops-lab1-8.git`<br>` ! [remote rejected] main -> main (push declined due to repository rule violations)`<br>`error: failed to push some refs to 'github.com:aCannedFish/ops-lab1-8.git'` |
| 第一轮：失败的 PR 运行 | [PR #4](https://github.com/aCannedFish/ops-lab1-8/pull/4) 的 [ci #8](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38568957560)：格式检查和 vet 通过，`TestConflicts/单周对双周不冲突` 与 `TestConflictEndpoint` 失败；当时必需检查 `ci` 为 `FAILURE`，PR 的 `mergeStateStatus` 为 `BLOCKED`，同时尚缺成员审批。 |
| 第一轮：修复后通过的 PR 运行 | [ci #9](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38569113229)：追加修复提交 `3cfa07fc4239769d5a96ba9c1dde77578aec6623`，恢复 `weeksOverlap` 的周次判断，并保留新增的双周重叠测试；格式检查、vet 和测试全部通过。 |
| 第一轮：合入 main 后的运行（含制品） | [PR #4](https://github.com/aCannedFish/ops-lab1-8/pull/4) 经 VigilLover 审批后 squash 合入，主干提交为 162fef9f00ef878e1f31409c8d78c81b040d748d。[ci #10](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38570502600) 全部通过，产出 [artifact enroll-conflict-162fef9f00ef878e1f31409c8d78c81b040d748d-10](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38570502600/artifacts/11897147070)，包含 tar.gz 与对应 .sha256。 |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | `Section.Valid` 的范围判断被改成 `s.End <= s.Start`，把 start 与 end 相同的单节次时段判为非法。当时 `TestValid` 的合法用例只有 `{Weekday: 3, Start: 1, End: 2}`，关于节次范围的非法用例覆盖了 `start > end`，两类用例都未覆盖 `start == end` 的边界，因此该改动不会让 `make test` 失败。 |
| 第二轮：仅含缺陷时仍然通过的运行 | [PR #6](https://github.com/aCannedFish/ops-lab1-8/pull/6) 的 [ci #13](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38600811332)：分支上只有缺陷提交 `340d8bd`，格式检查、vet 和测试全部通过，gate 没有发现该缺陷。 |
| 第二轮：补测试后失败的运行 | [ci #14](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38600929615)：追加测试提交 `7e55b98`，新增的单节次合法用例在 `TestValid` 中失败；格式检查与 vet 通过。 |
| 第二轮：撤销缺陷后通过的运行 | [ci #15](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38601111763)：提交 `279f221` 恢复 `s.End < s.Start`，保留新增用例，检查全部通过。 |
| 第二轮：合入 main 后的运行（含制品） | [PR #6](https://github.com/aCannedFish/ops-lab1-8/pull/6) 经 1JI0O 审批后 squash 合入，主干提交为 9364840e2faec689ef86d47b489809f37724e90e，main 上只新增了这一条测试用例。[ci #16](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38688536767) 全部通过，产出 [artifact enroll-conflict-9364840e2faec689ef86d47b489809f37724e90e-16](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38688536767/artifacts/11951360180)，包含 tar.gz 与对应 .sha256。 |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | [release #2](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709690916)：在临时分支 `scratch/gate-test` 的未合入提交 `301b6eb26b55dd1a90dfb5244fe6432c621f9bf1` 上创建测试 tag `v0.0.1`。[Release gate 步骤](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709690916/job/116337716093)输出 `Tag commit 301b6eb26b55dd1a90dfb5244fe6432c621f9bf1 is not in origin/main history.`，以退出码 1 失败；工具链安装、检查、打包和创建 Release 均跳过，未生成测试 Release。测试 tag 和临时分支已从远程及本地删除。 |
| Release v1.0.0 | [v1.0.0](https://github.com/aCannedFish/ops-lab1-8/releases/tag/v1.0.0)：在 [PR #8](https://github.com/aCannedFish/ops-lab1-8/pull/8) 合入 main 的提交 `3fdf50a724b1206ca62c8d602b7ceaa534f257a7` 上创建 tag，[release #1](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709389752) 全部通过。已下载 `enroll-conflict_1.0.0_linux_amd64.tar.gz` 和对应 `.sha256`，执行 `sha256sum -c` 得到 `enroll-conflict_1.0.0_linux_amd64.tar.gz: OK`；包内 `BUILD_INFO` 的版本、提交和 `run_id=38709389752` 与本次发布一致。 |
| Release v1.0.1 | [v1.0.1](https://github.com/aCannedFish/ops-lab1-8/releases/tag/v1.0.1)：在 [PR #10](https://github.com/aCannedFish/ops-lab1-8/pull/10) 合入 main 的提交 `2918346e168cf7ece3be02d8b466a92cc4bf253a` 上创建 tag，[release #3](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38713497689) 全部通过。已下载 `enroll-conflict_1.0.1_linux_amd64.tar.gz` 和对应 `.sha256`，使用 Python hashlib 计算并核对 SHA-256 一致（`8e9207e59d3585f128e4680e71bf93916c32acf9654de83d2cda1b77854318b0`）；包内 `BUILD_INFO` 的 `version=1.0.1`、`build_no=3`、提交号和 `run_id=38713497689` 与本次发布一致。 |

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | 1.0.0 | 1.0.1 |
| 构建号 | 1 | 3 |
| 提交号（SHA） | 3fdf50a724b1206ca62c8d602b7ceaa534f257a7 | 2918346e168cf7ece3be02d8b466a92cc4bf253a |
| 引入该提交的 PR | [PR #8](https://github.com/aCannedFish/ops-lab1-8/pull/8) | [PR #10](https://github.com/aCannedFish/ops-lab1-8/pull/10) |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709389752 | https://github.com/aCannedFish/ops-lab1-8/actions/runs/38713497689 |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | `fc05bdfecbfe374e4569eaa9df4618c0355ebb69a3497fbfe21795f2b3f2683e` | `8e9207e59d3585f128e4680e71bf93916c32acf9654de83d2cda1b77854318b0` |

仅凭构建号能否唯一确定是哪次运行？v1.0.0 的构建号在 `ci` 和 `release` 的运行列表中各对应哪次运行？不能。`build_no` 取自 `github.run_number`，各工作流独立计数。v1.0.0 的构建号为 `1`，在 `ci` 列表中对应 [ci #1](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38352773742)（PR 触发），在 `release` 列表中对应 [release #1](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709389752)（tag 推送触发）。实际构建 v1.0.0 制品的是 release #1，可通过 `BUILD_INFO` 中的 `run_id=38709389752` 唯一确定这次运行。

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：`fc05bdfecbfe374e4569eaa9df4618c0355ebb69a3497fbfe21795f2b3f2683e`。与上表中发布当时的值是否一致？一致。本次重新下载附件后使用 Python hashlib 计算 SHA-256，同时核对对应 `.sha256` 文件及上表中发布当时记录的值，三者相同，说明本次复核时 v1.0.0 附件内容与原记录一致。

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=1
run_id=38709389752
commit=3fdf50a724b1206ca62c8d602b7ceaa534f257a7
go=go1.26.9
== 3. 提交 3fdf50a724b1206ca62c8d602b7ceaa534f257a7
作者: Guanting Yu <yuguanting@sjtu.edu.cn>
时间: 2026-10-15T16:54:47+08:00
标题: feat: add tag-triggered release workflow (#8)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 1
提交页面（含关联的 PR）: https://github.com/aCannedFish/ops-lab1-8/commit/3fdf50a724b1206ca62c8d602b7ceaa534f257a7
构建它的运行: https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709389752
```

`trace.sh` 输出（v1.0.1）：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=3
run_id=38713497689
commit=2918346e168cf7ece3be02d8b466a92cc4bf253a
go=go1.26.9
== 3. 提交 2918346e168cf7ece3be02d8b466a92cc4bf253a
作者: wolf_lumine <hateful_mouse@outlook.com>
时间: 2026-10-15T17:26:53+08:00
标题: test: 增加同一单节次教学班冲突用例 (#10)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.1
版本号: 1.0.1    构建号: 3
提交页面（含关联的 PR）: https://github.com/aCannedFish/ops-lab1-8/commit/2918346e168cf7ece3be02d8b466a92cc4bf253a
构建它的运行: https://github.com/aCannedFish/ops-lab1-8/actions/runs/38713497689
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
{"version":"1.0.1","commit":"2918346e168cf7ece3be02d8b466a92cc4bf253a","build_no":"3","go":"go1.26.9"}

{"version":"1.0.0","commit":"3fdf50a724b1206ca62c8d602b7ceaa534f257a7","build_no":"1","go":"go1.26.9"}
```

## 5. 数据

时间均从 GitHub 页面读取，并注明时区。

第一轮 gate 演练的时间线（北京时间，UTC+8；CI 结束时间取工作流中唯一的 `ci` job 的 `completed_at`，修复推送时间取 GitHub 仓库 Activity 中的 push 事件）：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | 2026-10-14 21:05:38 | [PR #4](https://github.com/aCannedFish/ops-lab1-8/pull/4) |
| 第一次 `ci` 失败（运行结束） | 2026-10-14 21:06:01 | [ci #8](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38568957560/job/115868783325) |
| 推送修复提交 | 2026-10-14 21:06:46 | [修复提交](https://github.com/aCannedFish/ops-lab1-8/commit/3cfa07fc4239769d5a96ba9c1dde77578aec6623)、[推送记录](https://github.com/aCannedFish/ops-lab1-8/activity?ref=lab1%2Fgate-round1) |
| `ci` 通过（运行结束） | 2026-10-14 21:07:09 | [ci #9](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38569113229/job/115869315003) |
| 合入 main | 2026-10-14 21:17:02 | [PR #4](https://github.com/aCannedFish/ops-lab1-8/pull/4)、[squash 提交](https://github.com/aCannedFish/ops-lab1-8/commit/162fef9f00ef878e1f31409c8d78c81b040d748d) |

提交到发布的间隔，至少记录三个合入的 PR：

以下时间均为北京时间（UTC+8）。第一个提交的时间取该 PR 提交列表首项的 `commit.committer.date`，合入时间取 `merged_at`，发布时间取 Release 的 `published_at`。对应 Release 选取首次包含该 PR 的 squash 提交的正式版本。PR #8 的首个提交经过 cherry-pick，作者时间为 16:01:12，提交者时间为 16:45:07，本表统一使用提交者时间计算。

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| [#4](https://github.com/aCannedFish/ops-lab1-8/pull/4) | [2026-10-14 21:05:19](https://github.com/aCannedFish/ops-lab1-8/commit/42c17fd47f984a5372d8f53cf57c43ceb73d7797) | 2026-10-14 21:17:02 | [v1.0.0](https://github.com/aCannedFish/ops-lab1-8/releases/tag/v1.0.0)，2026-10-15 16:57:49 | 19 小时 52 分 30 秒（71550 s） |
| [#8](https://github.com/aCannedFish/ops-lab1-8/pull/8) | [2026-10-15 16:45:07](https://github.com/aCannedFish/ops-lab1-8/commit/e5426d4354b0ee05d9afea8c5de779830935305f) | 2026-10-15 16:54:48 | [v1.0.0](https://github.com/aCannedFish/ops-lab1-8/releases/tag/v1.0.0)，2026-10-15 16:57:49 | 12 分 42 秒（762 s） |
| [#10](https://github.com/aCannedFish/ops-lab1-8/pull/10) | [2026-10-15 17:17:56](https://github.com/aCannedFish/ops-lab1-8/commit/9ac36d7d6d6e27f9b232cce8b9f61b6334f2162d) | 2026-10-15 17:26:54 | [v1.0.1](https://github.com/aCannedFish/ops-lab1-8/releases/tag/v1.0.1)，2026-10-15 17:31:17 | 13 分 21 秒（801 s） |

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

以下为两轮 gate 演练的 7 次运行，均为 attempt 1，采用默认缓存。工作流时长按 GitHub 对应运行的 `updated_at - run_started_at` 计算，包含调度等待和收尾时间；备注另列 Actions job 页面中 `ci` job 的执行时长，每次运行均只有这一个 job。第一张时间线使用 job 的结束时刻，与工作流的结束时刻可能相差约 1 秒。

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| [ci #8](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38568957560/attempts/1) | PR #4 | 20 s | 第一轮缺陷被已有测试发现；[job](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38568957560/job/115868783325) 执行 15 s，失败。 |
| [ci #9](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38569113229/attempts/1) | PR #4 | 19 s | 修复后通过；[job](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38569113229/job/115869315003) 执行 14 s。 |
| [ci #10](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38570502600/attempts/1) | push main | 20 s | 第一轮合入后通过，含打包及上传；[job](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38570502600/job/115874122671) 执行 16 s。 |
| [ci #13](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38600811332/attempts/1) | PR #6 | 21 s | 仅有边界缺陷时仍然通过；[job](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38600811332/job/115979273271) 执行 16 s。 |
| [ci #14](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38600929615/attempts/1) | PR #6 | 16 s | 补充测试后失败；[job](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38600929615/job/115979681400) 执行 12 s。 |
| [ci #15](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38601111763/attempts/1) | PR #6 | 23 s | 撤销缺陷、保留测试后通过；[job](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38601111763/job/115980301302) 执行 18 s。 |
| [ci #16](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38688536767/attempts/1) | push main | 14 s | 第二轮合入后通过，含打包及上传；[job](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38688536767/job/116268858971) 执行 10 s。 |

上述工作流时长合计 133 s，均值 19 s，中位数 20 s，范围为 14～23 s。统计范围为本次两轮演练，表中已分别标明 PR 检查和 main 上包含打包、上传步骤的运行。

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

所用数据：第 3 节两轮演练的运行结果、第 5 节第一轮时间线和 7 次 CI 时长，以及 PR #4、#8、#10 的提交到发布间隔。第一轮从发起 PR 到检查通过共用时 `21:07:09 - 21:05:38 = 91 s`，两次等待检查结果各用时 23 s，从检查失败到推送修复提交用时 45 s。

在两轮演练中，merge gate 成功拦住了第一轮已被现有单元测试覆盖的周次冲突逻辑错误，但在第二轮起初未能拦住单节次课程（start == end）的边界缺陷，直到补充对应测试用例后才将其拦截，这说明自动化门禁的拦截能力完全取决于测试覆盖率。从时间成本来看，第一轮从发起 PR 到修复后 CI 通过共耗时 91 s，若扣除正常一次通过所需的 23 s，因缺陷失败实际多花了 68 s。在这多花的 68 s 中，额外等待一次 CI 结果占 23 s（约 33.8%），人工定位、修改代码并推送占 45 s（约 66.2%）；而在整个 91 s 周期内，两次等待 CI 共 46 s（约 50.5%），人工处理占 45 s（约 49.5%）。本组 7 次 CI 工作流平均耗时仅 19 s，在 PR #4、#8、#10 的“提交到发布间隔”中占比分别仅为 0.027%、2.49% 和 2.37%，可见自动化检查本身的时间成本极低，交付周期的主要耗时在于人工审批等待（如 PR #4 通过检查后等待了 593 s 才合入）和发版排期。即使补充了边界单元测试，诸如需求理解偏差、未编写用例的复杂业务分支、高并发性能瓶颈、安全漏洞以及生产环境配置差异等问题，依然无法仅靠这套单元测试发现。上述耗时与占比结论仅在当前小规模无外部依赖的 Go 项目、开启默认编译缓存、使用 GitHub 共享 Runner 的条件下成立，且 7 次运行和 3 个 PR 的小样本不足以代表长期工程交付效率。

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

所用数据：[PR #4 修复后的 ci #9](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38569113229/job/115869315003) 的事件日志如下；第 3 节记录了 squash 后的主干提交。本次核对以 `main` 提交 `03e760869488565a01888a4c7fdf551ad31f1ddd`（PR #14 合入后）为截止点，检查其提交历史，确认下面两个 SHA 均不在主干历史中，合入后的 `162fef9f00ef878e1f31409c8d78c81b040d748d` 在其中。

```text
github.event_name=pull_request
github.ref=refs/pull/4/merge
github.sha=5ca54507e620208172ed476797eec67d37e54234
github.event.pull_request.head.sha=3cfa07fc4239769d5a96ba9c1dde77578aec6623
```

在 PR 触发的运行日志中，github.sha 与 pull_request.head.sha 并不相同。这是因为 pull_request.head.sha（3cfa07f...）指向开发分支上最新推送的提交，而 github.sha（5ca5450...）指向 GitHub 为测试合并兼容性而在后台自动生成的临时预合并提交（refs/pull/4/merge）。由于本组仓库设置了仅允许 Squash merge（压缩合并），这两者在 main 的提交历史中都找不到；PR #4 合入后，分支上的多次提交被压缩为一个全新的提交 162fef9f00ef878e1f31409c8d78c81b040d748d 写入 main 历史，原分支提交仅保留在 PR 页面中供查阅。正因如此，待发布的正式制品只能从 main 上的提交或指向 main 的 Tag 构建：PR 阶段的构建产物对应的是未经最终审批的代码和随时会随主干更新而改变的临时合并提交，其 SHA 既不能代表最终获批合入主干的代码状态，也无法在 main 历史中通过 trace.sh 完成主干溯源。上述“两个原 SHA 均不在 main 历史中”的结论，以本组采用 Squash 合并方式为成立前提（若采用保留分支历史的普通 Merge commit 模式，分支的 head.sha 会成为 main 的祖先提交）。

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

所用数据：在 Linux 主机 `ecs-249f` 上临时克隆一份独立仓库，固定使用 v1.0.0 的源提交 `3fdf50a724b1206ca62c8d602b7ceaa534f257a7`。工具链为 Go `1.26.9`、GNU tar `1.35`、gzip `1.12`，构建目标为 `linux/amd64`，设置 `CGO_ENABLED=0`，并使用 Makefile 中的 `-trimpath`。①②④⑤沿用原发布参数：`VERSION=1.0.0 BUILD_NO=1 RUN_ID=38709389752 COMMIT=3fdf50a724b1206ca62c8d602b7ceaa534f257a7`；③仅将 `BUILD_NO` 改为 `2`。五次构建均使用该主机的 `umask 0002`，并设置 `GOMAXPROCS=1`、`GOFLAGS=-p=1`、`GOMEMLIMIT=256MiB`；④⑤之间等待 3 秒，结束后已执行 `git checkout -- Makefile`，工作区已无改动。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，`VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 均指定固定值 | `abe945029c49c5a9012ed371016d3dd21679892b8294b47d01140210ede49b0f` | `bd2e33b2757451c97eac5f9ceec08516e1f019f6d1347a9d6818d9083a32e40c` |
| ② | 与 ① 完全相同，再构建一次 | `abe945029c49c5a9012ed371016d3dd21679892b8294b47d01140210ede49b0f` | `bd2e33b2757451c97eac5f9ceec08516e1f019f6d1347a9d6818d9083a32e40c` |
| ③ | 仅将 `BUILD_NO` 换成另一个值 | `64c0bfe4f32bd008a3e683ebb28415c768675f7c158a9638e613ba5b94abd921` | `fa5f5d848322a56e9816fd5965fb9f179998c3d0dd35858ed5e93798253913b3` |
| ④ | 去掉 `--mtime=@0`，其余同 ① | `9b060e925dc61996847d91ce98838ef0bc0e00759a24a0165b987b10d535f440` | `02682f67d1c34ef310e42e610da0f9fdcdbc1be2f576ee60d2ee3c2f1ef3948e` |
| ⑤ | 同 ④，间隔几秒后再构建一次 | `9b060e925dc61996847d91ce98838ef0bc0e00759a24a0165b987b10d535f440` | `c9a8c5ff0cdcd68236bbfc5c222c3c1332a68a573a2ab88f4832178122bb36c5` |
| Actions | 对 main 上的一次 push 运行执行 Re-run all jobs，比较两次尝试中 package 步骤打印的值 | attempt 1：`5306adeb2e2961f9235ec1750bf7ae793e661b33f63d5588d49920556303dc5a`<br>attempt 2：`5306adeb2e2961f9235ec1750bf7ae793e661b33f63d5588d49920556303dc5a` | attempt 1：`48fd47309894b59d0c58fda5160840668396f4a3f8ad3da4d80cc4893c7ff79f`<br>attempt 2：`48fd47309894b59d0c58fda5160840668396f4a3f8ad3da4d80cc4893c7ff79f` |

Actions 对照使用同一主干提交的 [ci #21 / attempt 1](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709075443/attempts/1) 和 [attempt 2](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709075443/attempts/2)，第二次通过 `gh run rerun 38709075443` 重新运行全部 job，两次均成功。构建参数固定为 `VERSION=0.0.0-main BUILD_NO=21 RUN_ID=38709075443 COMMIT=3fdf50a724b1206ca62c8d602b7ceaa534f257a7`；attempt 从 1 变为 2，该值未写入本项目制品。重跑沿用原事件的 SHA 与 ref，机制参见 [GitHub 重跑说明](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/re-run-workflows-and-jobs)。

补充权限对照：①的二进制与正式 v1.0.0 附件内的二进制相同，但归档哈希不同。检查发现，本机目录和二进制的权限为 `0775`，`BUILD_INFO` 的权限为 `0664`，正式附件中对应的权限分别为 `0755`、`0644`。随后在同一源提交、工作区无改动的条件下，仅对本次构建进程设置 `umask 0022`，使用相同的追溯参数重新构建，将结果写入新的 `dist/permission-control/` 子目录。二进制哈希仍为 `abe945029c49c5a9012ed371016d3dd21679892b8294b47d01140210ede49b0f`，归档哈希变为 `fc05bdfecbfe374e4569eaa9df4618c0355ebb69a3497fbfe21795f2b3f2683e`，与第 4 节的原发布附件一致。

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

实验结果显示，①与②、以及 Actions 的两次重跑在二进制和归档哈希上均完全相同，因为编译参数固定、工作区干净且 Re-run 保持 run_number 与 run_id 不变；而 ③ 修改 BUILD_NO 后，由于该值通过 -ldflags 编译进二进制并写入 BUILD_INFO，导致两项哈希均改变。④ 的二进制哈希与 ① 不同，原因是临时修改 Makefile 未提交导致 Git 工作区变脏，Go 编译器自动将内嵌的 vcs.modified 标记从 false 改为 true（与任务 0 现象一致）；④ 与 ⑤ 二进制相同但归档哈希不同，则是因为去掉 --mtime=@0 后，间隔 3 秒生成的不同文件修改时间被记录进了 tar 归档头中（此外本组补充实验发现，本地默认 umask 0002 与云端 0022 造成的文件权限差异也会改变 tar.gz 哈希，统一为 0022 后本地归档哈希与正式 Release 完全一致）。由此可见，制品中应当包含可执行文件及版本、提交号、构建号、运行号等固定追溯元数据，而不应包含未提交的本地改动、宿主机绝对路径、动态构建时间戳以及未受控的文件权限/属主信息。本实验中 release 工作流在 main 同一提交上重新编译，是因为需要将 ci 中的占位版本号 0.0.0-main 替换为 Tag 对应的正式版本号（如 1.0.0）写入二进制文件。这种做法与严格的“一次构建、多处部署”原则存在冲突，代价是增加了重复编译测试的开销，并引入了两次构建间环境发生微小变化的风险。上述可重复构建结论成立的条件是：操作系统与 CPU 架构、Go 编译器补丁版本、源码树干净状态、注入的版本构建参数，以及打包时的时间戳、文件顺序和 umask 权限等元数据均严格固定。

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

所用数据：第 4 节回退前后的 `/version` 分别为 `1.0.1 / 2918346e168cf7ece3be02d8b466a92cc4bf253a / build_no=3` 和 `1.0.0 / 3fdf50a724b1206ca62c8d602b7ceaa534f257a7 / build_no=1`，重新下载的 v1.0.0 归档哈希仍为 `fc05bdfecbfe374e4569eaa9df4618c0355ebb69a3497fbfe21795f2b3f2683e`。问题 3 的①与权限对照提供了同一源提交重建的比较；本次读取的 [v1.0.0 元数据](https://api.github.com/repos/aCannedFish/ops-lab1-8/releases/tags/v1.0.0)和 [v1.0.1 元数据](https://api.github.com/repos/aCannedFish/ops-lab1-8/releases/tags/v1.0.1)中，`immutable` 均为 `false`。

若 v1.0.1 发布后发现缺陷，应立即将线上运行的服务切回已校验过的 v1.0.0 制品完成回退，随后在 main 分支修复缺陷并打新 Tag（如 v1.0.2）发布新版本。绝不应删除 v1.0.1 的 Tag 后复用同一版本号重发，因为制品一旦发布就应保持不可变，复用版本号会导致已下载或已部署该版本的环境、缓存与审计记录出现“同名不同物”的混乱，彻底破坏可追溯性。在任务 6 的回退操作中，直接启动了事先从 v1.0.0 Release 解压出的原二进制文件，无需重新构建；两次 /version 的输出显示版本号、提交号和构建号直接从 1.0.1 / 2918346e... / 3 切回了 1.0.0 / 3fdf50a7... / 1，证明服务已直接恢复为旧版原制品。如果改为从源码临时重新构建 v1.0.0，正如问题 3 所示，一旦构建号、运行号、工作区状态或宿主机 umask 权限存在任何差异，生成的归档哈希就会与原附件不同，既耗时又无法保证与原线上版本完全一致。倘若有人恶意删除 v1.0.0 的附件并同时替换为篡改后的新压缩包、匹配的 .sha256 以及伪造包含合法主干提交号的 BUILD_INFO，trace.sh 将无法发现异常（因为自洽校验和与 Git 历史检查均能通过）；但对比本组在第 4 节中独立保存的发布初始哈希值（fc05bdfe...），就能立刻发现文件被掉包。为从平台层面根除这种篡改风险，可启用 GitHub 的 Immutable releases（不可变发布）功能，在 Release 发布后永久锁定其 Tag 与附件禁止删改（本仓库当前查询值为 false）；此外，上述回退机制能够顺利生效的前提是旧版制品完整可用，且旧版程序与当前生产环境的配置、数据结构保持向前/向后兼容。
## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | 已完成。[v1.0.2](https://github.com/aCannedFish/ops-lab1-8/releases/tag/v1.0.2) 的 [release #4](https://github.com/aCannedFish/ops-lab1-8/actions/runs/39539696505) 中，release 与 image 两个 job 均成功；镜像已推送 GHCR，按 digest 运行及匿名拉取均通过，实际摘要和主机架构见下文。 |
| O2 缓存对反馈时间的影响 | 已完成两组各 5 次运行，全部成功；[PR #11](https://github.com/aCannedFish/ops-lab1-8/pull/11) 使用默认缓存，[PR #12](https://github.com/aCannedFish/ops-lab1-8/pull/12) 用作无缓存对照，已关闭且未合并。预期、逐次数据、波动与局限见下文。 |
| O3 迁移到小组项目 | 以 [CharonRyui/rshx](https://github.com/CharonRyui/rshx) 为对比对象，补充其现有 Rust gate、发布流水线与统一示例的差异，以及完整迁移仍需补齐的内容。已有 CI/Release 证据与本次迁移验收分开记录，不将已有流水线等同于 O3 已全部完成，详见下文。 |

### O1：Release 附件构建镜像

[PR #11](https://github.com/aCannedFish/ops-lab1-8/pull/11) 已在 `release.yml` 中新增 `image` job，通过 `needs: release` 等待发布 job 完成。镜像 job 下载该 tag 对应的 `enroll-conflict_<版本>_linux_amd64.tar.gz` 和 `.sha256`，校验后解压，将解压目录作为 Docker 构建上下文。仓库的 `Dockerfile` 直接复制 `enroll-conflict` 和 `BUILD_INFO`。工作流通过 `${GITHUB_REPOSITORY,,}` 将镜像名称转为小写，发布地址为 `ghcr.io/acannedfish/ops-lab1-8:<tag>`，平台为 `linux/amd64`；只有镜像 job 获得 `packages: write` 权限。

本地已下载 v1.0.0 附件，`sha256sum -c` 返回 `OK`，随后成功构建 Docker 镜像，并运行容器查看 `-version` 输出。运行容器的本地主机 `uname -m` 为 `x86_64`，Docker 服务端为 `linux/x86_64`，镜像平台为 `linux/amd64`，版本输出如下：

```text
enroll-conflict version=1.0.0 commit=3fdf50a724b1206ca62c8d602b7ceaa534f257a7 build_no=1 go=go1.26.9
```

本地镜像摘要为 `sha256:460bffc7330ad7885a538b07e07d0c50c20f552c47a8d73534f76c899bf5adaf`，已使用摘要引用 `ops-lab1-o1@sha256:460bffc7330ad7885a538b07e07d0c50c20f552c47a8d73534f76c899bf5adaf` 运行容器，`/healthz` 返回 `ok`，`/version` 与上述版本一致。另外，从容器中复制二进制文件，通过 `cmp` 与附件解压得到的二进制文件比较，结果相同，二进制 SHA-256 为 `abe945029c49c5a9012ed371016d3dd21679892b8294b47d01140210ede49b0f`。

2026-10-21 11:39:09（UTC+8），在已合入 main 的提交 `a9ce09ea64afadc77f52947559c71a534956710e` 上发布 [v1.0.2](https://github.com/aCannedFish/ops-lab1-8/releases/tag/v1.0.2)。[release #4](https://github.com/aCannedFish/ops-lab1-8/actions/runs/39539696505) 的 [release job](https://github.com/aCannedFish/ops-lab1-8/actions/runs/39539696505/job/119024366023) 和 [image job](https://github.com/aCannedFish/ops-lab1-8/actions/runs/39539696505/job/119024522478) 均成功。image job 下载本次 Release 的附件后，`sha256sum -c` 返回 `enroll-conflict_1.0.2_linux_amd64.tar.gz: OK`，随后构建并推送镜像；以下为此次正式 GHCR 发布的结果。

| 项 | 实际结果 |
|---|---|
| 镜像 tag | `ghcr.io/acannedfish/ops-lab1-8:v1.0.2` |
| GHCR digest | `sha256:1de57e4fb3e72c92af564e97e53f5b9420c93e04d25b50e9d34c4cabd8f0b65b` |
| 运行容器的 Docker 主机 | `linux/x86_64`（image job 的 Docker info 输出） |
| 镜像平台 | `linux/amd64`（匿名读取 OCI config 核对） |
| 匿名拉取 | 通过，使用空 Docker 配置拉取同一 digest；另以匿名请求成功读取 manifest 和 config。 |

image job 使用摘要而非 tag 运行镜像，命令及实际输出如下：

```bash
docker run --rm --platform linux/amd64 \
  ghcr.io/acannedfish/ops-lab1-8@sha256:1de57e4fb3e72c92af564e97e53f5b9420c93e04d25b50e9d34c4cabd8f0b65b -version
```

```text
enroll-conflict version=1.0.2 commit=a9ce09ea64afadc77f52947559c71a534956710e build_no=4 go=go1.26.9
```

镜像中的二进制仅支持 `linux/amd64`。在 Apple 芯片等 arm64 主机上运行时，需要加上 `--platform linux/amd64`，并依赖 Docker 的模拟支持，运行速度可能降低。

**为什么不拆成另一个 Release 事件工作流？** 发布步骤使用仓库内置的 `GITHUB_TOKEN` 创建 Release。为防止工作流之间循环触发，使用该 token 产生的事件通常不会启动新工作流（`workflow_dispatch` 和 `repository_dispatch` 是例外），因此这里创建 Release 后不会触发另一个监听 `release: published` 的工作流。将镜像 job 放在同一工作流中，并设置为依赖发布 job，即可使用本次运行的信息和已生成的 Release 附件。参见 [Triggering a workflow](https://docs.github.com/en/actions/how-tos/writing-workflows/choosing-when-your-workflow-runs/triggering-a-workflow#triggering-a-workflow-from-a-workflow)。

**匿名拉取检查。** 本次 image job 的 `Check anonymous pull` 步骤使用临时空 Docker 配置拉取上述 digest，日志返回相同的 `Digest` 和 `Status: Image is up to date`，未出现匿名拉取失败的 warning。另以未登录身份取得仅用于匿名读取的 registry token，成功读取该 digest 的 OCI manifest 和 config，确认平台为 `linux/amd64`，版本标签为 `1.0.2`，提交标签与本次主干提交一致。因此，本次镜像已验证可匿名访问；这一判断来自实际请求结果，未仅凭仓库公开或工作流绿色状态推断。包权限设置说明参见 [Configuring package access](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)。

### O2：缓存实验

**实验前预期（在创建无缓存 PR 和重跑之前记录）。** 本项目的 `go.mod` 没有第三方依赖，所以预计节省的依赖下载时间接近 0 秒。`setup-go` 同时缓存 Go 编译产物，执行 `vet`、`test` 时仍可使用标准库及项目代码的编译缓存。因此，预计命中已有缓存时检查耗时会缩短，可能节省数十秒，实际节省的时间还需扣除缓存下载、解压和保存的开销。实验前查询仓库 Actions Cache，main 已有约 45 MB 缓存，缓存键中的 Go 版本为 `1.26.9`。当时计划使用相同的 `ubuntu-24.04`、Go `1.26.9` 和 Go 源码，比较两个 PR 的 `ci` 时长，以排除 main 推送后额外打包和上传步骤的影响。

**实验方法。** 预期先记录在本地文档中，并在创建 [PR #12](https://github.com/aCannedFish/ops-lab1-8/pull/12) 时写入初始说明（09:35:12 UTC，早于无缓存组首次运行的 09:35:16 UTC）；文档随后通过[提交 d7fb433](https://github.com/aCannedFish/ops-lab1-8/commit/d7fb4335ffc09cb5d1d459c1d5f15e17e859969b) 推送。对照 PR 相对当时 main 的唯一改动是增加 `cache: false`，该 PR 现已关闭，未合并。缓存开启组使用 [PR #11](https://github.com/aCannedFish/ops-lab1-8/pull/11) 的 `ci #26`，关闭组使用 `ci #27`；两组的 Go 源码、测试、Makefile、Go 版本和 ci 检查步骤完全相同，PR #11 中用于实验的提交仅修改了 tag 触发的 release 工作流。每组均保留首次运行，再通过 Re-run all jobs（`gh run rerun <run_id>`，未指定 `--failed` 或 job）重跑 4 次，重跑期间没有修改提交。两组首次运行依次启动，随后大致交错重跑，部分运行时间重叠，未控制分配到的物理主机。

**全部数据。** 日期均为 2026-10-15，时间为 UTC。数据来自 GitHub Actions 对应的 attempt/job 页面，以及 API 中的 `run_started_at`、`updated_at` 和 job/step 的 `started_at`、`completed_at` 字段。主要比较 job 时长，该值不含排队时间；反馈时长取本次运行的 `updated_at - run_started_at`，包含调度等待及工作流收尾时间。API 时间精度为秒，表中 0 秒的步骤仍有执行，只是耗时不足约 1 秒。

| 缓存 | 运行 / attempt（证据） | job 开始（UTC） | job 时长 | 反馈时长 | setup-go | vet | test | 工具链来源 / runner 镜像版本 |
|---|---|---|---:|---:|---:|---:|---:|---|
| 开 | [ci #26 / 1](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38713827030/job/116351527993) | 09:33:38 | 17 s | 20 s | 3 s | 6 s | 1 s | 预装 / `20261012.334.1` |
| 开 | [ci #26 / 2](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38713827030/job/116352668245) | 09:36:27 | 17 s | 30 s | 9 s | 0 s | 0 s | 下载 / `20261004.327.1` |
| 开 | [ci #26 / 3](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38713827030/job/116352919869) | 09:37:02 | 16 s | 21 s | 10 s | 0 s | 0 s | 下载 / `20261004.327.1` |
| 开 | [ci #26 / 4](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38713827030/job/116353156390) | 09:37:38 | 20 s | 26 s | 4 s | 8 s | 2 s | 预装 / `20261012.334.1` |
| 开 | [ci #26 / 5](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38713827030/job/116353396137) | 09:38:12 | 13 s | 18 s | 2 s | 5 s | 1 s | 预装 / `20261012.334.1` |
| 关 | [ci #27 / 1](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38714036018/job/116352222696) | 09:35:19 | 33 s | 36 s | 9 s | 14 s | 3 s | 下载 / `20261004.327.1` |
| 关 | [ci #27 / 2](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38714036018/job/116352640455) | 09:36:21 | 24 s | 30 s | 1 s | 13 s | 2 s | 预装 / `20261012.334.1` |
| 关 | [ci #27 / 3](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38714036018/job/116352932883) | 09:37:05 | 30 s | 36 s | 2 s | 19 s | 3 s | 预装 / `20261012.334.1` |
| 关 | [ci #27 / 4](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38714036018/job/116353250767) | 09:37:51 | 23 s | 28 s | 2 s | 14 s | 2 s | 预装 / `20261012.334.1` |
| 关 | [ci #27 / 5](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38714036018/job/116353549151) | 09:38:35 | 37 s | 42 s | 11 s | 14 s | 3 s | 下载 / `20261004.327.1` |

| 指标 | 默认缓存（5 次） | 关闭缓存（5 次） |
|---|---:|---:|
| job 平均时长 | 16.6 s | 29.4 s |
| job 中位数 | 17 s | 30 s |
| job 范围 | 13～20 s | 23～37 s |
| 反馈平均时长 | 23.0 s | 34.4 s |
| 反馈范围 | 18～30 s | 28～42 s |

**观察与结论。** 默认缓存组的 5 次日志均出现 `Cache restored successfully`，使用同一个缓存键恢复了约 43 MiB 的缓存；无缓存组均设置 `cache: false`，没有恢复该编译缓存。默认缓存组的 job 平均时长少 12.8 秒，减少量约占关闭组的 43.5%，反馈平均时长少 11.4 秒，以上均为本次样本的统计结果。`vet` 与 `test` 的耗时缩短，符合标准库及项目编译缓存能够节省时间的预期；`make test` 使用 `go test -count=1`，每次均执行测试。runner 预装的 Go 工具链独立于 Actions 恢复的 `GOCACHE` 和 `GOMODCACHE`，因此即使设置 `cache: false`，日志中仍可能出现 `Found in cache @ /opt/hostedtoolcache/...`。两组运行分配到了两个版本的 runner 镜像，需要下载工具链的运行中，setup-go 耗时明显增加。缓存下载、网络状况、共享主机的 CPU 和磁盘负载，以及排队时间也会影响耗时，例如缓存开启组 #26/2 与关闭组 #27/2 的反馈时长均为 30 秒。本次数据支持保留默认缓存，具体节省的时间会随运行条件变化。

**局限。** 每组仅运行 5 次，集中于同一时间段，执行顺序未随机化，runner 镜像、硬件和负载也未固定。默认缓存组测量的是命中已有缓存时的耗时，未测量首次缓存未命中和上传缓存的开销；源码版本变化时，可能只能复用部分编译结果。无缓存组仅关闭 setup-go 的模块和编译缓存，runner 的其他系统缓存及预装工具链仍然保留。本项目没有第三方依赖，PR 检查也不执行打包和上传步骤，因此结论适用于本次实验条件；对于依赖较多的项目、main 的完整制品流水线或首次未命中缓存的情况，还需增加样本、跨时段重复实验，并控制 runner 镜像等因素后再作判断。

### O3：与 CharonRyui/rshx 的迁移对比

**对比对象与范围。** 选取 [CharonRyui/rshx](https://github.com/CharonRyui/rshx) 作为目标项目：它是通过 SSH 在多台机器上执行操作的 Rust 命令行工具，与本实验的 Go HTTP 服务 `enroll-conflict` 不同。以下核对日期为 2026-10-21，源码固定到 main 的[提交 `9f83f35`](https://github.com/CharonRyui/rshx/tree/9f83f35dec47b0bc66da44e935723a426ee22c3d)，避免后续仓库变化影响对比。本次只补充文档，不修改目标仓库；其已有 CI 和 Release 可以作为方案参考与运行证据，但不能据此声称已完成本实验的全部迁移验收。

**核对依据。**

- [Cargo.toml](https://github.com/CharonRyui/rshx/blob/9f83f35dec47b0bc66da44e935723a426ee22c3d/Cargo.toml) 与 [rust-toolchain.toml](https://github.com/CharonRyui/rshx/blob/9f83f35dec47b0bc66da44e935723a426ee22c3d/rust-toolchain.toml)：包版本为 `0.5.0`，使用 Rust edition `2024`，工具链固定为 `1.97.0`，包含 `clippy`、`rustfmt`。仓库提交了 `Cargo.lock`。
- [CI 配置](https://github.com/CharonRyui/rshx/blob/9f83f35dec47b0bc66da44e935723a426ee22c3d/.github/workflows/ci.yml)与 [CI 运行](https://github.com/CharonRyui/rshx/actions/runs/36875823355)：2026-10-01 对上述 main 提交的检查成功，`lint`、`test` 两个 job 均通过。
- [Release 配置](https://github.com/CharonRyui/rshx/blob/9f83f35dec47b0bc66da44e935723a426ee22c3d/.github/workflows/release.yml)、[Release 运行](https://github.com/CharonRyui/rshx/actions/runs/36875860713)与 [v0.5.0](https://github.com/CharonRyui/rshx/releases/tag/v0.5.0)：同一提交完成验证、创建草稿、四个目标构建及最终发布；[Release 元数据](https://api.github.com/repos/CharonRyui/rshx/releases/tags/v0.5.0)的 `immutable` 为 `true`。这些运行发生在本次实验之前，属于目标项目原有能力，不是本次迁移新增的 PR 或演练记录。

| 对比项 | 统一示例 enroll-conflict | rshx 的实际情况与迁移差异 |
|---|---|---|
| 工具链、依赖与缓存 | 固定 Go `1.26.9`，仅依赖标准库，通过 `make` 调用检查与构建。 | 使用 Cargo 管理多个第三方依赖；通过工具链文件固定 Rust 版本，并在 Clippy、测试和构建时使用 `--locked`，不允许静默修改锁文件。迁移应保留 Rust 工具链 action 的缓存，并关注依赖下载与 `target` 编译缓存，不能直接套用 O2 中无第三方依赖的耗时结论。 |
| 质量检查入口 | 单个 `ci` job 顺序执行 `make fmt-check`、`make vet`、`make test`。 | `lint` 执行 `cargo fmt --all -- --check` 和 `cargo clippy --all-targets --locked -- -D warnings`；`test` 执行 `cargo test --locked` 和 `cargo build --release --locked`。迁移后必需检查应对应实际的 `lint`、`test`，不能仍填写示例的 `ci`；有检查工作流不等于已配置强制合并 gate。 |
| 测试环境与运行方式 | 测试本地冲突规则及 HTTP 接口；运行时通过 `/healthz`、`/version` 检查服务。 | [测试辅助模块](https://github.com/CharonRyui/rshx/blob/9f83f35dec47b0bc66da44e935723a426ee22c3d/tests/support/mod.rs)将假的 `ssh` 放在测试进程的 `PATH` 前部，记录参数并模拟响应，不需要网络、真实 SSH 服务器或私钥。迁移应保留这种隔离；真实主机连通性、认证和部署行为仍需另行验证，不能由桩测试通过推断。CLI 运行检查应使用 `rshx --version` 及命令退出码，而非 HTTP 路径。 |
| runner 与触发条件 | 固定 `ubuntu-24.04`，PR 和 main push 执行检查，main push 另外打包并上传 artifact。 | 使用 `ubuntu-latest`，还支持手动触发 CI；相同 ref 的新运行会取消旧运行。main CI 只检查和编译，没有上传实验所需的主干归档。若按实验条件复现，应固定到 `ubuntu-24.04`，补充 main 制品上传，并将被取消的运行与真正失败区分。 |
| 发布 gate 与版本 | tag 发布前检查提交是否在 `origin/main` 历史中，再执行质量检查；版本从 tag 写入程序。 | `verify` 重跑格式、Clippy 和测试，并要求去掉 `v` 的 tag 与 `Cargo.toml` 版本一致，避免 CLI 与 Release 版本不一致；但当前没有 main 祖先检查。重新测试通过只说明该 tag 的代码通过检查，不代表已获准合入 main，迁移仍需补上完整历史获取及祖先检查。 |
| 目标平台与链接方式 | `CGO_ENABLED=0` 构建一个 `linux/amd64` 程序。 | 发布 `x86_64`、`aarch64` 各自的 `gnu`、`musl` 四种 Linux 包；aarch64 使用 `cargo-zigbuild` 和 Zig 交叉编译。GNU 包动态链接 glibc，musl 包按项目说明为静态构建，回退必须选择相同架构及适配运行环境的包，不能只按版本号选附件。 |
| 制品格式与追溯 | 一个顶层目录中包含程序和 `BUILD_INFO`，另附 `.sha256`；记录版本、构建号、提交号和运行号，可由 `trace.sh` 追溯。 | 现有包名形如 `rshx-v0.5.0-x86_64-unknown-linux-musl.tar.gz`，打包步骤只将 `rshx` 放在归档根部，没有顶层目录、`BUILD_INFO` 或单独的 `.sha256`；CLI 版本也未提供实验所需的提交与构建号。因此现有包不能直接交给模板 `trace.sh` 完成同样的追溯，必须先补齐包结构与元数据。 |
| 归档与可重复构建 | 通过固定 tar 排序、属主、时间和 `gzip -n` 减少归档差异；权限仍需控制，见问题 3。 | 当前直接执行 `tar -czf`，没有示例中的元数据归一化处理。`Cargo.lock` 和工具链固定不能单独证明归档可重复，还应控制打包元数据、构建路径及交叉编译工具版本，并进行两次实际构建的哈希对照。 |
| 发布编排与回退 | 一个 job 构建并发布；O1 在其后下载附件构建镜像，不重新编译。 | 按 `verify → release（草稿）→ build（四目标矩阵）→ publish` 编排，全部目标成功后才公开 Release；构建失败会留下未发布草稿。v0.5.0 的不可变发布状态与该流程相符，草稿阶段可补齐附件，不应覆盖已正式发布的版本。回退应取已发布的旧版对应平台附件，不重新构建；实际下载、校验和回退仍需记录证据。 |

**迁移时仍需补齐的内容。**

1. **让检查成为强制 gate。** 在 rshx 配置 PR、审批和必需检查 `lint`、`test`，固定实验 runner，并通过失败、修复、审批、合入的 PR 验证。本次查询[main 分支保护 API](https://api.github.com/repos/CharonRyui/rshx/branches/main/protection)返回 `Branch not protected`，[仓库 rulesets API](https://api.github.com/repos/CharonRyui/rshx/rulesets)返回空列表；不能将现有绿色 CI 作为已配置与本实验相同保护规则的证据。
2. **补齐主干归档与追溯。** 在 main push 后生成并上传带顶层目录的归档和同名 `.sha256`；`BUILD_INFO` 至少包含 `name=rshx`、`version`、`build_no`、`commit`，并增加 `run_id`、Rust 工具链及 target。其中提交号取 main/tag 运行的 `github.sha`，构建号和运行号分别取 `github.run_number`、`github.run_id`，不能手工填写。保持模板所需字段后再引入 `scripts/trace.sh`，并保留各平台目标的区别。
3. **补齐发布来源约束。** 在现有版本一致性检查之外，加入 tag 提交属于 main 历史的验证；保留草稿、矩阵构建、最终发布的顺序，在不可变发布之前一次上传完整归档与校验文件。多架构是 rshx 的已有能力，满足实验基本要求可以先验证一个 Linux 目标，不必把四目标全部视为新增要求。
4. **记录迁移验收证据。** 提供实际迁移 PR、失败与修复运行、未合入提交的 tag 被拒记录，以及两个新版本的附件、`trace.sh` 输出和 CLI 回退结果；独立保存发布时哈希。不能把 2026-10-01 已有的成功运行写成本次新完成的 gate 演练或回退实验。

**结论。** 可以迁移的是“PR 检查与审批、可信主干来源、tag 发布、制品追溯、复用旧制品回退”的原则，而不是照搬 Go 命令和包名。rshx 已有适配 Rust 的检查、隔离测试、多目标构建和不可变 Release 发布流程，但实验要求的强制合并保护、main 来源校验、归档追溯格式及迁移演练证据仍需补齐。因此，本节完成的是基于实际仓库的差异说明，不将 O3 标记为已全部完成。

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

以下记录截至 2026-10-15、[PR #14](https://github.com/aCannedFish/ops-lab1-8/pull/14) 合入时，依据 GitHub 上各 PR 的作者、合并状态和 `APPROVED` 审批记录填写。[PR #12](https://github.com/aCannedFish/ops-lab1-8/pull/12) 为缓存实验的对照 PR，已关闭且未合并，未计入“发起并合入”的数量。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| aCannedFish | [#2](https://github.com/aCannedFish/ops-lab1-8/pull/2)、[#8](https://github.com/aCannedFish/ops-lab1-8/pull/8)、[#9](https://github.com/aCannedFish/ops-lab1-8/pull/9)、[#14](https://github.com/aCannedFish/ops-lab1-8/pull/14) | [#3](https://github.com/aCannedFish/ops-lab1-8/pull/3)、[#5](https://github.com/aCannedFish/ops-lab1-8/pull/5)、[#7](https://github.com/aCannedFish/ops-lab1-8/pull/7)、[#10](https://github.com/aCannedFish/ops-lab1-8/pull/10)、[#13](https://github.com/aCannedFish/ops-lab1-8/pull/13) |
| CharonRyui | [#1](https://github.com/aCannedFish/ops-lab1-8/pull/1)、[#11](https://github.com/aCannedFish/ops-lab1-8/pull/11) | [#2](https://github.com/aCannedFish/ops-lab1-8/pull/2) |
| 1JI0O | [#3](https://github.com/aCannedFish/ops-lab1-8/pull/3)、[#4](https://github.com/aCannedFish/ops-lab1-8/pull/4)、[#5](https://github.com/aCannedFish/ops-lab1-8/pull/5) | [#6](https://github.com/aCannedFish/ops-lab1-8/pull/6)、[#9](https://github.com/aCannedFish/ops-lab1-8/pull/9)、[#11](https://github.com/aCannedFish/ops-lab1-8/pull/11)、[#14](https://github.com/aCannedFish/ops-lab1-8/pull/14) |
| VigilLover | [#10](https://github.com/aCannedFish/ops-lab1-8/pull/10)、[#13](https://github.com/aCannedFish/ops-lab1-8/pull/13) | [#4](https://github.com/aCannedFish/ops-lab1-8/pull/4) |
| Mikujoker | [#6](https://github.com/aCannedFish/ops-lab1-8/pull/6)、[#7](https://github.com/aCannedFish/ops-lab1-8/pull/7) | [#8](https://github.com/aCannedFish/ops-lab1-8/pull/8) |
