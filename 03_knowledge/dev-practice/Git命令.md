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


# merge的原理

## 一、先回到那个图

merge 前：

A -- B -- C -- F -- G   main  
         \  
          D -- E        feature

如果你站在 `feature` 分支上执行：

git merge main

结果会变成：

A -- B -- C -- F -- G -------- M   feature  
         \                  /  
          D ------ E ------/

这里：

- `E` 是你原来 `feature` 分支的头
- `G` 是 `main` 分支的头
- `M` 是 merge 后新生成的提交

---

## 二、merge 的原理到底是什么

Git merge 的核心不是“无脑拼接文件”，而是：

> **找两个分支的最近公共祖先，然后做三方合并（three-way merge）。**

这里有 3 个关键点：

1. **公共祖先**：`C`
2. **当前分支末端**：`E`
3. **要合并进来的分支末端**：`G`

也就是 Git 会比较：

- `C -> E`：你这边改了什么
- `C -> G`：对方那边改了什么

然后把两边改动尝试合并，生成新的结果，也就是 `M` 对应的快照。

---

## 三、所以 M 里到底有什么

答案是：

> **M 的文件内容，通常是“E 的改动 + G 的改动”合并后的最终结果。**

也就是说：

- `D E` 这条线上的改动，不会丢
- `F G` 这条线上的改动，也会进来
- 如果两边改的是不同地方，Git 通常能自动合并
- 如果两边改的是同一块内容，就会冲突，需要你手动选

所以可以粗略理解为：

M = 合并 E 和 G 后的结果

而不是：

M = 只包含 D、E

---

## 四、M 和普通 commit 有什么不同

普通提交通常只有 **一个父提交**：

E -> X

这里 `X` 的 parent 是 `E`。

但 merge commit `M` 有 **两个父提交**：

- 第一个父提交：`E`
- 第二个父提交：`G`

你可以理解成：

> `M` 同时承认：“我是从这两条线合并出来的。”

这就是为什么 merge 不会改写原来的 `D E F G`，而是新增一个 `M` 来把历史接起来。

---

## 五、为什么说 M“包含 D、E 的改动”也对，但不够严谨

因为从**最终文件结果**看，M 当然包含了 D、E 带来的效果。  
但从 **Git 历史结构** 看：

- `D`
- `E`

这些提交本身仍然独立存在于历史中，**不是被 M 替代掉了**

也就是说：

- `D E` 没消失
- `F G` 也没消失
- `M` 只是说：从这一刻起，这两条历史汇合了

所以更精确的表述是：

> **M 的内容结果包含了两边改动的合并结果；历史结构上，D/E 和 F/G 仍然是独立提交。**

---

## 六、一个更具体的例子

假设公共祖先 `C` 里有文件：

`a.txt`

内容是：

hello

---

## 你的分支 `feature` 上

`D E` 把它改成：

hello world

---

## `main` 分支上

`F G` 又加了一行：

hello  
bye

---

## merge 时 Git 会看什么

Git 发现：

- 从 `C` 到 `E`，你改了第一行
- 从 `C` 到 `G`，别人加了第二行

这两种改动不冲突，于是 `M` 里的结果可能是：

hello world  
bye

这就是 merge commit `M` 的文件快照。

所以你可以看到：

- `D E` 的效果保留了
- `F G` 的效果也保留了

---

## 七、那冲突是怎么来的

如果两边都改了同一行，比如：

`C` 里是：

timeout=10

你的分支改成：

timeout=20

main 改成：

timeout=30

这时 Git 比较 `C -> E` 和 `C -> G`，发现：

> 两边都改了同一个地方，而且改法不一致

它就没法自动决定 M 里到底该写 `20` 还是 `30`，于是产生冲突。

所以 merge 的本质不是简单拼接，而是：

> **基于共同祖先，尝试把两边相对祖先的改动融合。**

---

## 八、merge 为什么不改写历史

因为 merge 的思路是：

> “两边历史我都尊重，我不重放你的提交，我只额外造一个提交，把结果接起来。”

所以：

- `merge` 会新增一个 `M`
- `rebase` 会重写 `D E` 成 `D' E'`

这就是它们最大的原理区别。

---

## 九、fast-forward 是什么关系

顺带一提，不是所有 merge 都会生成 `M`。

比如如果历史是：

A -- B -- C   main  
         \  
          D -- E   feature

如果你在 `main` 上执行：

git merge feature

而 `main` 自己从 `C` 之后没有新提交，Git 发现：

> main 只是单纯落后于 feature，没有分叉

那它可以直接把 `main` 指针从 `C` 挪到 `E`，这叫：

> **fast-forward merge**

就不会生成 `M`。

只有真正“两边都前进过”时，通常才需要 merge commit。
