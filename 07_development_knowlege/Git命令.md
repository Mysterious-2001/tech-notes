# git branch 

查看本地分支 
-a 查看所有分支
# git switch

切换到某个分支

# git merge

把别的分支合并进当前分支。

# git fetch

更新远程分支跟踪状态

# 回退
## 最安全的做法：`git revert`

如果这个远程分支**已经被别人看到、可能被别人拉过、或者已经发过 PR**，最稳的是：

git revert commit-id

它的意思不是“把历史删掉”，而是：新建一个反向提交，把某次提交的改动抵消掉。

这样做的特点：

- 不改写历史
- 远程历史是连续的
- 最适合公共分支

### 例子

先看历史：git log --oneline

假设是：

a7a7a7a feat: add api C  
b6b6b6b feat: add api B  
c5c5c5c feat: add api A

你想撤销 `a7a7a7a` 这个提交：

git revert a7a7a7a  
git push

这样会生成一个新提交，大意相当于：

d8d8d8d revert "feat: add api C"  
a7a7a7a feat: add api C  
b6b6b6b feat: add api B  
c5c5c5c feat: add api A

最终效果是代码回退了，但历史还在。

### 想撤销多个提交

比如想把最近 3 个提交都撤掉，可以一个个 revert：

git revert commit3
git revert commit2
git revert commit1

通常按**从新到旧**撤。

---

## 你自己独占分支时：`git reset` + 强推

如果这个分支**基本只有你自己用**，没人基于它继续开发，你想直接把分支指针退回去，可以用：

git reset --hard commit-id
git push --force

这叫：

> **改写历史**

它的意思是：

- 本地分支直接退回到某个旧提交
- 然后强制把远程也改成那个状态

---

## 例子

历史原来是：

a7a7a7a feat: add api C  
b6b6b6b feat: add api B  
c5c5c5c feat: add api A

你想回到 `c5c5c5c`：

git reset --hard c5c5c5c  
git push --force

结果会变成：

c5c5c5c feat: add api A

后面的 `b6b6b6b` 和 `a7a7a7a` 在这个分支历史上就“消失了”。

---

## 为什么危险

因为如果别人已经拉了这个分支，他们本地历史还是旧的。  
你一 force push，大家历史就分叉了，后面容易乱。

所以这招适合：

- 个人功能分支
- 还没被别人依赖
- 你明确知道自己在改写历史

---

## 比 `--force` 更推荐的写法

git push --force-with-lease

比单纯 `--force` 更安全一点。  
它会先检查远程是不是还是你预期的状态，避免把别人刚 push 的内容冲掉。

所以推荐写：

git reset --hard commit-id  
git push --force-with-lease

# 只是想“临时看看旧版本”，不是永久回退

那不要 reset，也不要 revert。

可以直接：

git checkout commit-id

# 如果已经 reset 过头了怎么办

别慌，先看：

git reflog

这是 Git 的“后悔药”。

它会记录你本地分支指针移动历史。  
就算你 `reset --hard` 了，很多时候也能从 reflog 里把原来的 commit 找回来。

比如：

git reflog

可能看到：

abc1234 HEAD@{0}: reset: moving to c5c5c5c  
a7a7a7a HEAD@{1}: commit: feat: add api C  
b6b6b6b HEAD@{2}: commit: feat: add api B

想恢复到 `a7a7a7a`：

git reset --hard a7a7a7a