# MyDuck 项目原则

1. **永远跟上游保持最新**：`third_party/microduck_rl` 是 `pollen-robotics/microduck_rl` 的 fork（remote: upstream）。每次开工前先同步：
   ```bash
   cd third_party/microduck_rl
   git fetch upstream
   git log --oneline develop..upstream/develop   # 有新提交就 merge/rebase 进我们的 develop
   ```
   我们的改动只在 develop 分支的增量 commit 上（Dance 任务相关），与上游文件尽量保持纯新增，降低合并冲突。
   `third_party/microduck-playground` 同理：`upstream` = `Vottivott/microduck-playground`，`origin` = 我们的 fork；开工前 `git fetch upstream`，看 `tug-of-war..upstream/main` 有无新提交。我们的工作分支是 `tug-of-war`（本地、未推送；历史包含已废弃的 `swing360` 分支全部提交；本地 `main` 只跟踪 `upstream/main`，不在上面改）。注意上游 2026-09 有过一次事故后重写历史（force-push），同步一律用 rebase，不要 merge 旧历史。
2. **修改必重训，同方才续训**：任何代码/奖励/编舞修改 → 从零重训；只有「同配方、只是加步数」（如 1000 步验证不错 → 继续到 2000/4000）才从 checkpoint 续训（`--agent.load-checkpoint model_XXX.pt --agent.resume True`）。
3. **迭代节奏与 envs 分档**（我们自己的约定，归纳自上游用法：主训示例 4096 envs、desk-climb 续训 1024、评估 64）：**配方探索期 1024 envs × 2000 迭代**快速试（trick 类任务先 500-1000 迭代看曲线再加）；**配方定稿出正式策略才用官方默认 4096 envs × 4000 迭代**；评估 64-512 envs。不一律 4096（那是官方为步态/爬梯等硬任务调的主训默认值）。
4. **成本纪律（用户对费用敏感）**：只用 **4090D**（¥1.88/h）；缺货时等待重试或**先问用户**，绝不擅自换更贵规格（如 vGPU/5090）。**余额低于 ¥15 时开新训练前必须先报余额并征得用户同意**。训练完成后**不自动关机，等用户指示**；用户明确说不用了才 `off`；长期不用经用户确认后 `release`（关机仍收磁盘费）。训练先冒烟（64 envs × 5 iters）再正式。评估/渲染/ONNX 导出等 CPU 能干的活优先挪 CPU 实例（¥0.1-0.3/h）或本地，不占 GPU 时长；同一轮实验尽量一次开机干完（冒烟→训练→评估→导出），省开机重建。
5. **验证纪律**：改训练侧代码必须本地 `uv run --with pytest pytest tests/ -q` 全绿；策略效果以 `scripts/dance_to_timeline.py` + `check_beat_align.py` 的部署侧数据为准，不看训练指标下结论。若测试收集阶段全部报 `ModuleNotFoundError: No module named 'mjlab_microduck'`：是 `.venv` 带了 macOS `hidden` 标记导致 .pth 被跳过，`chflags -R nohidden .venv` 即可。
6. **暂无实体机器人**：部署侧仓库 `pollen-robotics/microduck`（daemon/runtime）只作**契约参考**（61D 观测布局、命令槽约定、schema-2 策略清单，见该仓库 `docs/policy-manifest.md`），不追它的 release、不做机器人升级类操作；策略验证以 sim 彩排（`infer_policy.py` 等）为准。拿到硬件后再恢复追 daemon release。
