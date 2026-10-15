# 实验 1 记录：第 8 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 8 组 |
| 仓库地址 | <https://github.com/aCannedFish/ops-lab1-8> |
| 代码载体 | 统一示例 `enroll-conflict`（Go） |
| runner 与语言版本 | `ubuntu-24.04`，Go `1.26.9`；[ci 第 3 次运行](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38353330057)成功，已产出 artifact `enroll-conflict-d24ae3fadbb46bceed0a8d8556e2f8a598109187-3` |
| 合并方式 | squash（仓库仅允许此方式；[PR #1](https://github.com/aCannedFish/ops-lab1-8/pull/1)合并后对应提交 `d24ae3fadbb46bceed0a8d8556e2f8a598109187`） |

## 2. 任务 0：本地基线

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | 在 Linux 主机 `ecs-249f` 的 `/home/ubuntu/lab1-local` 中克隆[课程模板仓库](https://github.com/zilinglius/operations-2026-lab1-sample)，执行 `make package VERSION=1.0.0 BUILD_NO=1`。两次构建均设置 `GOMAXPROCS=1`、`GOFLAGS=-p=1`、`GOMEMLIMIT=256MiB`。实际输出：<br>`name=enroll-conflict`<br>`version=1.0.0`<br>`build_no=1`<br>`run_id=local`<br>`commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`go=go1.26.9` |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | 将 `conflict.go` 第 1 行改为 `package main // task0 baseline observation`，不提交，以相同参数再次打包。`BUILD_INFO` 没有变化，二进制 sha256 发生变化。两次 `sha256sum` 的实际输出：<br>第一次：`f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e  dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict`<br>第二次：`fd9e4cdfb5dba94309d1eb422110fa4e4217bce6cd13a5c8e22048f5ca1fa098  dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict` |
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | 第一次：<br>`build vcs=git`<br>`build vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`build vcs.time=2026-10-12T16:26:07Z`<br>`build vcs.modified=false`<br>第二次：<br>`build vcs=git`<br>`build vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`build vcs.time=2026-10-12T16:26:07Z`<br>`build vcs.modified=true`<br>仅 `vcs.modified` 从 `false` 变为 `true`，表示第二次构建包含未提交的工作区改动。虽然只修改了注释，这项内嵌构建信息仍使二进制内容及哈希发生变化。 |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | 不能完整回答。`name`、`version` 能标识包及版本，`go` 记录工具链版本；构建主机和构建时间没有记录，`go version -m` 中的 `vcs.time` 也是提交时间。`commit` 取本地 HEAD，第二次构建时仍为 `7b63b41c13f061439997b0e6e4eaa2af922a7aa1`，没有反映未提交的注释改动。因此，若仅凭该字段认定制品完全来自此提交，会把实际构建源码认错；`BUILD_INFO` 本身没有标记工作区是否干净。 |

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | [main-merge-gate](https://github.com/aCannedFish/ops-lab1-8/rules/24975603)：状态为 Active，作用于默认分支 `main`，Bypass list 为空，管理员也不能绕过。启用 Restrict deletions、Block force pushes、Require a pull request before merging（Required approvals：1）、Require status checks to pass（必需检查：`ci`，来源为 GitHub Actions；未要求分支在合并前更新到最新主干）。 |
| 组长直接 `git push origin main` 被拒的终端输出 | 使用组长 `aCannedFish` 账号（仓库管理员），创建一个空提交后执行 `git push origin main`，退出码为 1。实际输出：<br>`remote: error: GH013: Repository rule violations found for refs/heads/main.`<br>`remote: Review all repository rules at https://github.com/aCannedFish/ops-lab1-8/rules?ref=refs%2Fheads%2Fmain`<br>`remote:`<br>`remote: - Changes must be made through a pull request.`<br>`remote:`<br>`remote: - Required status check "ci" is expected.`<br>`remote:`<br>`To github.com:aCannedFish/ops-lab1-8.git`<br>` ! [remote rejected] main -> main (push declined due to repository rule violations)`<br>`error: failed to push some refs to 'github.com:aCannedFish/ops-lab1-8.git'` |
| 第一轮：失败的 PR 运行 | [PR #4](https://github.com/aCannedFish/ops-lab1-8/pull/4) 的 [ci #8](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38568957560)：格式、vet 通过，TestConflicts/单周对双周不冲突 与 TestConflictEndpoint 失败；当时必需检查 ci 为 FAILURE，PR 的 mergeStateStatus 为 BLOCKED，同时尚缺成员审批。 |
| 第一轮：修复后通过的 PR 运行 | [ci #9](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38569113229)：追加修复提交 3cfa07fc4239769d5a96ba9c1dde77578aec6623，恢复 weeksOverlap 的周次判断；格式、vet、测试全部通过，保留新增的双周重叠测试。 |
| 第一轮：合入 main 后的运行（含制品） | [PR #4](https://github.com/aCannedFish/ops-lab1-8/pull/4) 经 VigilLover 审批后 squash 合入，主干提交为 162fef9f00ef878e1f31409c8d78c81b040d748d。[ci #10](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38570502600) 全部通过，产出 [artifact enroll-conflict-162fef9f00ef878e1f31409c8d78c81b040d748d-10](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38570502600/artifacts/11897147070)，包含 tar.gz 与对应 .sha256。 |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | `Section.Valid` 的范围判断被改成 `s.End <= s.Start`，把 start 与 end 相同的单节次时段判为非法；现有 `TestValid` 的合法用例只有 `{Weekday: 3, Start: 1, End: 2}`，非法用例覆盖的是 start > end，两者都不含 start == end 这条边界，因此该改动不会让 `make test` 失败。 |
| 第二轮：仅含缺陷时仍然通过的运行 | [PR #6](https://github.com/aCannedFish/ops-lab1-8/pull/6) 的 [ci #13](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38600811332)：分支上只有缺陷提交 340d8bd，格式、vet、测试全部通过，gate 没有拦住它。 |
| 第二轮：补测试后失败的运行 | [ci #14](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38600929615)：追加用例提交 7e55b98，新增的单节次合法用例在 TestValid 中失败；格式与 vet 通过。 |
| 第二轮：撤销缺陷后通过的运行 | [ci #15](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38601111763)：提交 279f221 恢复 `s.End < s.Start`，保留新增用例，全部通过。 |
| 第二轮：合入 main 后的运行（含制品） | [PR #6](https://github.com/aCannedFish/ops-lab1-8/pull/6) 经 1JI0O 审批后 squash 合入，主干提交为 9364840e2faec689ef86d47b489809f37724e90e，main 上只新增了这一条测试用例。[ci #16](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38688536767) 全部通过，产出 [artifact enroll-conflict-9364840e2faec689ef86d47b489809f37724e90e-16](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38688536767/artifacts/11951360180)，包含 tar.gz 与对应 .sha256。 |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | [release #2](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709690916)：在临时分支 `scratch/gate-test` 的未合入提交 `301b6eb26b55dd1a90dfb5244fe6432c621f9bf1` 上创建测试 tag `v0.0.1`。[Release gate 步骤](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709690916/job/116337716093)输出 `Tag commit 301b6eb26b55dd1a90dfb5244fe6432c621f9bf1 is not in origin/main history.`，以退出码 1 失败；工具链安装、检查、打包和创建 Release 均跳过，未生成测试 Release。测试 tag 和临时分支已从远程及本地删除。 |
| Release v1.0.0 | [v1.0.0](https://github.com/aCannedFish/ops-lab1-8/releases/tag/v1.0.0)：在 [PR #8](https://github.com/aCannedFish/ops-lab1-8/pull/8) 合入 main 的提交 `3fdf50a724b1206ca62c8d602b7ceaa534f257a7` 上创建 tag，[release #1](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38709389752) 全部通过。已下载 `enroll-conflict_1.0.0_linux_amd64.tar.gz` 和对应 `.sha256`，执行 `sha256sum -c` 得到 `enroll-conflict_1.0.0_linux_amd64.tar.gz: OK`；包内 `BUILD_INFO` 的版本、提交和 `run_id=38709389752` 与本次发布一致。 |
| Release v1.0.1 | <链接> |

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | <填写> | <填写> |
| 构建号 | <填写> | <填写> |
| 提交号（SHA） | <填写> | <填写> |
| 引入该提交的 PR | <编号与链接> | <编号与链接> |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | <链接> | <链接> |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | `fc05bdfecbfe374e4569eaa9df4618c0355ebb69a3497fbfe21795f2b3f2683e` | <填写> |

仅凭构建号能否唯一确定是哪次运行？v1.0.0 的构建号在 `ci` 和 `release` 的运行列表中各对应哪次运行？<填写>

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：<填写>。与上表中发布当时的值是否一致？<填写>

`trace.sh` 输出（v1.0.0）：

```text
<粘贴>
```

`trace.sh` 输出（v1.0.1）：

```text
<粘贴>
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
<运行 v1.0.1 时>
<换回 v1.0.0 后>
```

## 5. 数据

时间均从 GitHub 页面读取，并注明时区。

第一轮 gate 演练的时间线（北京时间，UTC+8；CI 结束时间取该工作流唯一 ci job 的 completed_at，修复推送时间取 GitHub 仓库 Activity 的 push 事件）：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | 2026-10-14 21:05:38 | [PR #4](https://github.com/aCannedFish/ops-lab1-8/pull/4) |
| 第一次 `ci` 失败（运行结束） | 2026-10-14 21:06:01 | [ci #8](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38568957560/job/115868783325) |
| 推送修复提交 | 2026-10-14 21:06:46 | [修复提交](https://github.com/aCannedFish/ops-lab1-8/commit/3cfa07fc4239769d5a96ba9c1dde77578aec6623)、[推送记录](https://github.com/aCannedFish/ops-lab1-8/activity?ref=lab1%2Fgate-round1) |
| `ci` 通过（运行结束） | 2026-10-14 21:07:09 | [ci #9](https://github.com/aCannedFish/ops-lab1-8/actions/runs/38569113229/job/115869315003) |
| 合入 main | 2026-10-14 21:17:02 | [PR #4](https://github.com/aCannedFish/ops-lab1-8/pull/4)、[squash 提交](https://github.com/aCannedFish/ops-lab1-8/commit/162fef9f00ef878e1f31409c8d78c81b040d748d) |

提交到发布的间隔，至少记录三个合入的 PR：

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| #<编号> | <填写> | <填写> | <填写> | <填写> |
| #<编号> | <填写> | <填写> | <填写> | <填写> |
| #<编号> | <填写> | <填写> | <填写> | <填写> |

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| #<编号> | <填写> | <填写> | <填写> |

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

<填写>

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

<填写>

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，`VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 均指定固定值 | <填写> | <填写> |
| ② | 与 ① 完全相同，再构建一次 | <填写> | <填写> |
| ③ | 仅将 `BUILD_NO` 换成另一个值 | <填写> | <填写> |
| ④ | 去掉 `--mtime=@0`，其余同 ① | <填写> | <填写> |
| ⑤ | 同 ④，间隔几秒后再构建一次 | <填写> | <填写> |
| Actions | 对 main 上的一次 push 运行执行 Re-run all jobs，比较两次尝试中 package 步骤打印的值 | <两次的值> | <两次的值> |

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

<填写>

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

<填写>

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | 已实现，见 [PR #11](https://github.com/aCannedFish/ops-lab1-8/pull/11)；本地验证及发布后的取证步骤见下文。正式 GHCR digest 与匿名拉取结果待合入后发布验证。 |
| O2 缓存对反馈时间的影响 | 已完成两组各 5 次运行，全部成功；[PR #11](https://github.com/aCannedFish/ops-lab1-8/pull/11) 默认缓存，[PR #12](https://github.com/aCannedFish/ops-lab1-8/pull/12) 为不合并的无缓存对照。预期、逐次数据、波动与局限见下文。 |
| O3 迁移到小组项目 | 尚未迁移：未提供目标项目仓库，不能以本示例的流水线或迁移计划作为实际迁移证据。所需差异见下文。 |

### O1：Release 附件构建镜像

[PR #11](https://github.com/aCannedFish/ops-lab1-8/pull/11) 新增同一 `release.yml` 内的 `image` job，`needs: release` 保证先生成附件。镜像 job 下载该 tag 的 `enroll-conflict_<版本>_linux_amd64.tar.gz` 和 `.sha256`，校验后解压，将解压目录作为 Docker 构建上下文。仓库的 `Dockerfile` 仅复制 `enroll-conflict` 和 `BUILD_INFO`，没有重新编译。镜像名称通过 `${GITHUB_REPOSITORY,,}` 转小写，发布至 `ghcr.io/acannedfish/ops-lab1-8:<tag>`，声明平台 `linux/amd64`；只有镜像 job 获得 `packages: write`。

本地已下载 v1.0.0 附件，`sha256sum -c` 返回 `OK`，Docker 构建并运行 `-version` 成功。运行容器的本地主机 `uname -m` 为 `x86_64`，Docker 服务端为 `linux/x86_64`，镜像平台为 `linux/amd64`，输出如下：

```text
enroll-conflict version=1.0.0 commit=3fdf50a724b1206ca62c8d602b7ceaa534f257a7 build_no=1 go=go1.26.9
```

本地镜像摘要为 `sha256:460bffc7330ad7885a538b07e07d0c50c20f552c47a8d73534f76c899bf5adaf`，已用 `ops-lab1-o1@sha256:460bffc7330ad7885a538b07e07d0c50c20f552c47a8d73534f76c899bf5adaf` 而非 tag 运行容器，`/healthz` 返回 `ok`，`/version` 与上述版本一致。另从容器复制二进制，与解压后的附件执行 `cmp`，结果相同，二进制 SHA-256 为 `abe945029c49c5a9012ed371016d3dd21679892b8294b47d01140210ede49b0f`。

以上为本地验证，不是 GHCR 发布证明。`main` 的规则要求 PR 审批及 `ci` 通过；PR 合入后，需在 main 历史上的提交创建新的版本 tag。镜像 job 将在 Job Summary 中记录实际 GHCR digest、主机内核架构、Docker 服务端架构和镜像平台，并执行下列形式的命令，以摘要固定运行对象：

```bash
docker run --rm --platform linux/amd64 \
  ghcr.io/acannedfish/ops-lab1-8@sha256:<本次推送的摘要> -version
```

镜像中的二进制是 `linux/amd64`，不提供多架构构建。在 Apple 芯片等 arm64 主机上仍需 `--platform linux/amd64`，依赖 Docker 的模拟支持，运行速度可能降低。

**为什么不拆成另一个 Release 事件工作流？** 发布步骤使用仓库内置 `GITHUB_TOKEN` 创建 Release。为防止自动化递归触发，使用该 token 产生的事件通常不会启动新工作流（`workflow_dispatch` 和 `repository_dispatch` 是例外），因此不能期待 `release: published` 工作流因此运行。将镜像 job 放在同一工作流并显式依赖发布 job，可以沿用当前运行的上下文及已生成的 Release 附件。参见 [Triggering a workflow](https://docs.github.com/en/actions/how-tos/writing-workflows/choosing-when-your-workflow-runs/triggering-a-workflow#triggering-a-workflow-from-a-workflow)。

**匿名拉取检查。** GHCR 新包默认私有，`org.opencontainers.image.source` 将包关联到仓库，但仓库公开不能代替匿名拉取验证。工作流使用临时空 Docker 配置执行 `docker pull --platform linux/amd64 <image>@<digest>`，将成功或失败写入 Summary，失败时保留日志并提示检查包可见性。需结合 [包设置](https://github.com/users/aCannedFish/packages/container/ops-lab1-8/settings)确认实际权限；发布前不能声称已可匿名拉取。参见 [Configuring package access](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)。

### O2：缓存实验

**实验前预期（在创建无缓存 PR 和重跑之前记录）。** 本项目的 `go.mod` 没有第三方依赖，所以预计节省的依赖下载时间接近 0 秒；但 `setup-go` 同时缓存 Go 编译产物，标准库及项目代码在 `vet`、`test` 中仍会使用编译缓存。因此预计热缓存会缩短检查耗时，可能达到数十秒，净收益应扣除缓存下载、解压和保存的开销，不能预设“没有依赖就完全没有收益”。实验前查询仓库 Actions Cache，main 已有约 45 MB 缓存，键中 Go 版本为 `1.26.9`。将用相同的 `ubuntu-24.04`、Go `1.26.9` 和 Go 源码，以两个 PR 的 `ci` 作为比较，避免 main 的 push 额外打包与上传影响对照。

**实验方法。** 预期先在文档提交 `2602845` 中记录，然后创建 [PR #12](https://github.com/aCannedFish/ops-lab1-8/pull/12)，其相对 main 的唯一改动是增加 `cache: false`，该 PR 不合并。缓存开启组使用 [PR #11](https://github.com/aCannedFish/ops-lab1-8/pull/11) 的 `ci #26`，关闭组使用 `ci #27`；两者 Go 源码、测试、Makefile、Go 版本和 ci 检查步骤完全相同，参与实验的 #11 提交仅修改 tag 触发的 release 工作流。各保留首次运行，再通过 Re-run all jobs（`gh run rerun <run_id>`，未指定 `--failed` 或 job）重跑 4 次，尝试间没有修改提交。两组首轮依次启动，此后重跑时间大致交错并有并行，未控制分配到的物理主机。

**全部数据。** 日期均为 2026-10-15，时间为 UTC，来源为 GitHub Actions 的对应 attempt/job 链接及 API `run_started_at`、`updated_at`、job/step 的 `started_at` 和 `completed_at`。job 时长用于主要比较，不含排队；反馈时长为该次 `run_started_at` 到 `updated_at`，包含等待调度及工作流收尾。API 时间精度为秒，步骤显示 0 秒表示不足约 1 秒，不表示未执行。

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

**观察与结论。** 默认缓存组的 5 次日志均出现 `Cache restored successfully`，恢复同一个约 43 MiB 的缓存键；无缓存组均为 `cache: false`，没有恢复该编译缓存。默认缓存组 job 平均少 12.8 秒，约为关闭组的 43.5%，反馈平均少 11.4 秒；这些是本次样本的描述统计。`vet` 与 `test` 的耗时缩短支持“标准库及项目编译缓存仍有收益”的预期；`make test` 使用 `go test -count=1`，没有跳过测试执行。`Found in cache @ /opt/hostedtoolcache/...` 指 runner 预装的 Go 工具链，即使 `cache: false` 也可能出现，与 Actions 恢复的 `GOCACHE` / `GOMODCACHE` 不同。所分配的 runner 有两个镜像版本，工具链下载使 setup-go 耗时明显增加；缓存下载、网络、共享主机 CPU/磁盘负载以及排队也会波动，甚至有缓存 #26/2 与无缓存 #27/2 的反馈时长都为 30 秒。因此本次数据支持保留默认缓存，但不能把平均差值视为每次运行的固定收益。

**局限。** 每组仅 5 次，集中于同一时间段，未随机化执行顺序，也没有锁定同一 runner 镜像、硬件或负载。默认缓存组测量的是现有热缓存，没有测首次未命中及上传成本，且跨源码版本可能只复用部分编译结果。无缓存组禁用的是 setup-go 的模块及编译缓存，不是清空 runner 的所有系统缓存或工具链。本项目没有第三方依赖、PR 不执行 package/upload，结论不能直接推广到大型依赖项目、main 的完整制品流水线或冷缓存；如需准确量化，应增加样本、跨时段重复并控制 runner 镜像等因素。

### O3：迁移所需的项目差异

当前代码载体仍是统一示例 `enroll-conflict`，本记录没有目标小组项目的仓库链接，也没有迁移后的 PR 或运行证据，故 O3 尚未完成。实际迁移需要先检查目标项目的构建入口、测试、依赖锁文件、运行时和部署形式，再修改对应仓库；不能仅复制本实验的文档就认定完成。

| 项目差异 | 迁移时需要核对与调整 |
|---|---|
| 语言和依赖 | 以目标项目工具链及锁文件配置安装、缓存、格式检查、静态检查和测试，不直接沿用 Go 版本与 Makefile 目标。 |
| 测试环境 | 若使用数据库、外部服务或前端浏览器，需要可重复的测试环境和必要配置；必需检查名称须与实际 job 一致。 |
| 分支保护 | 在目标仓库配置 PR、审批和必需状态检查；用实际失败及修复的 PR 验证 gate。 |
| 发布制品 | 明确可部署制品、版本及提交追溯信息；保留 main 历史检查和附件校验，按目标项目更改包名及目录。 |
| 容器运行时 | 本示例是静态 Go 二进制，可用 `scratch`；动态链接程序、JVM、Node 或 Python 项目需要相应运行时、依赖及启动方式。 |
| 发布及回退 | 在目标仓库完成 tag 发布、下载和运行制品、回退旧版本，记录实际链接及迁移中遇到的差异。 |

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

以下记录截至任务 4 第二轮 PR #6 合入，依据各 PR 的作者与实际审批记录填写。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| aCannedFish | [#2](https://github.com/aCannedFish/ops-lab1-8/pull/2) | [#3](https://github.com/aCannedFish/ops-lab1-8/pull/3) |
| CharonRyui | [#1](https://github.com/aCannedFish/ops-lab1-8/pull/1) | [#2](https://github.com/aCannedFish/ops-lab1-8/pull/2) |
| 1JI0O | [#3](https://github.com/aCannedFish/ops-lab1-8/pull/3)、[#4](https://github.com/aCannedFish/ops-lab1-8/pull/4) | [#6](https://github.com/aCannedFish/ops-lab1-8/pull/6) |
| VigilLover | 暂无 | [#4](https://github.com/aCannedFish/ops-lab1-8/pull/4) |
| Mikujoker | [#6](https://github.com/aCannedFish/ops-lab1-8/pull/6) | 暂无 |
