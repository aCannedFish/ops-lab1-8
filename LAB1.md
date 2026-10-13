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
| 第一轮：失败的 PR 运行 | <链接> |
| 第一轮：修复后通过的 PR 运行 | <链接> |
| 第一轮：合入 main 后的运行（含制品） | <链接> |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | <一两句话> |
| 第二轮：仅含缺陷时仍然通过的运行 | <链接> |
| 第二轮：补测试后失败的运行 | <链接> |
| 第二轮：撤销缺陷后通过的运行 | <链接> |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | <链接> |
| Release v1.0.0 | <链接> |
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
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | <填写> | <填写> |

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

第一轮 gate 演练的时间线：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | <填写> | <PR 链接> |
| 第一次 `ci` 失败（运行结束） | <填写> | <链接> |
| 推送修复提交 | <填写> | <链接> |
| `ci` 通过（运行结束） | <填写> | <链接> |
| 合入 main | <填写> | <链接> |

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
| O1 容器镜像 | <未做 / 链接、镜像摘要（digest）、运行容器的主机架构> |
| O2 缓存对反馈时间的影响 | <未做 / 预期、数据与结论> |
| O3 迁移到小组项目 | <未做 / 仓库链接> |

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| <用户名> | <编号> | <编号> |
