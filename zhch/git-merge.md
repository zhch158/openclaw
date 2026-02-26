你这个需求本质上就是：**解决合并冲突时，让 `pnpm-lock.yaml` 直接采用 main 分支版本（theirs）**。

在你这个 PR（https://github.com/zhch158/openclaw/pull/5）里：

- base（你的 `zhch158` 分支）上的 `pnpm-lock.yaml`：BlobSha `d89445e690f97f396337bcde40c7ad5f1096cd9e`  
  https://github.com/zhch158/openclaw/blob/5b25b01cc9807ae2c8d0238e4379dd7cb773bf36/pnpm-lock.yaml
- head（`main`）上的 `pnpm-lock.yaml`：BlobSha `0ddc70d9f97755cb322fc1f73a415d3cce912d00`  
  https://github.com/zhch158/openclaw/blob/e3b25f64c8f895479b0d0bf6d6b4ff3c616a225c/pnpm-lock.yaml

你要“接受 main 的所有修改”，最终合并结果里 `pnpm-lock.yaml` 应该等同于 **main 的版本**。

## 在本地用 git 最快的做法（推荐）

假设你现在在要合并的目标分支（`zhch158` 分支）上，并且正在把 `main` merge 进来且出现冲突：

```bash
git checkout zhch158
git merge origin/main   # 如果此时提示冲突

# 对 pnpm-lock.yaml 选用对方(main)版本
git checkout --theirs pnpm-lock.yaml
git add pnpm-lock.yaml

# 继续解决其它冲突（如有），然后完成 merge
git status
git commit
git push
```

说明：

- `--theirs` 在“merge main -> 当前分支”的语境下，一般指 **被合并进来的那个分支（main）**。
- 如果你不确定 theirs/ours，可以用下面的“显式指定来源”方式，永远不会搞错。

## 永远不会搞错的做法：直接从 main 取文件覆盖

不管当前 merge 到哪一步，你都可以强制把文件替换成 `main` 的版本：

```bash
git checkout origin/main -- pnpm-lock.yaml
git add pnpm-lock.yaml
```

然后再 commit / 继续 merge。

## GitHub 网页端（Resolve conflicts）怎么做

如果你在 PR 页面点 “Resolve conflicts”，在冲突编辑器里对 `pnpm-lock.yaml`：

- **直接删掉冲突标记**并把内容改成 main 的版本，或者
- 更简单：不要手改 lockfile，改用本地方法一次性选 theirs（因为 lockfile 很大，网页端容易误操作）。

## 可选但推荐：合并完成后重新生成锁文件校验

为了确保 lockfile 与 `package.json`/workspace 状态一致：

```bash
pnpm -v
pnpm install --lockfile-only
git status
# 若有变化再提交
```

---

如果你告诉我你现在是用哪种方式合并（本地 `git merge` / `rebase` / GitHub 网页端），以及冲突状态截图或 `git status` 输出，我可以按你当前状态给你精确到每一步的命令。
