# 一、文件与目录操作

## 1. `pwd`

查看当前所在目录。

pwd

## 2. `ls`

查看目录内容。

ls  
ls -l      # 详细信息  
ls -a      # 包含隐藏文件  
ls -la     # 常用组合

## 3. `cd`

切换目录。

cd /Users/yourname/Desktop  
cd ~       # 回到用户目录  
cd ..      # 回到上一级  
cd -       # 回到上一次所在目录

## 4. `mkdir`

创建目录。

mkdir demo  
mkdir -p a/b/c   # 递归创建多层目录

## 5. `touch`

创建文件。

touch test.txt

## 6. `cp`

复制文件/目录。

cp a.txt b.txt  
cp -r dir1 dir2

## 7. `mv`

移动或重命名文件。

mv old.txt new.txt  
mv a.txt ./backup/

## 8. `rm`

删除文件或目录。

rm a.txt  
rm -r dir  
rm -rf dir   # 危险，强制递归删除

## 9. `find`

查找文件。

find . -name "*.go"  
find . -type f  
find . -type d

## 10. `open`

用 Finder 或默认应用打开文件/目录。

open .  
open test.txt  
open -a "Visual Studio Code" .

这个在 Mac 上非常实用。

---

# 二、查看文件内容

## 11. `cat`

直接输出文件内容。

cat test.txt

## 12. `less`

分页查看大文件。

less app.log

常用操作：

- `q`：退出
- `/关键字`：搜索
- `n`：下一个匹配

## 13. `head` / `tail`

看文件开头或结尾。

head -n 20 app.log  
tail -n 50 app.log  
tail -f app.log   # 实时追踪日志

工作中看日志非常高频。

---

# 三、文本搜索与处理

## 14. `grep`

搜索文本内容。

grep "error" app.log  
grep -i "error" app.log      # 忽略大小写  
grep -rn "TODO" .           # 递归搜索

## 15. `wc`

统计行数、单词数、字节数。

wc -l app.log

## 16. `sort`

排序。

sort names.txt

## 17. `uniq`

去重，通常和 `sort` 配合。

sort names.txt | uniq

## 18. `cut`

提取列。

cut -d ',' -f 1 data.csv

## 19. `awk`

按列处理文本，很强。

awk '{print $1}' file.txt  
awk -F ',' '{print $2}' data.csv

## 20. `sed`

替换文本。

sed 's/old/new/g' file.txt

Mac 自带 `sed` 和 Linux 有些细节差异，尤其 `-i` 原地修改时要注意：

sed -i '' 's/old/new/g' file.txt

---

# 四、开发中最常用的命令

## 21. `git`

版本控制核心工具。

git status  
git add .  
git commit -m "fix: update logic"  
git pull  
git push  
git log --oneline --graph  
git diff  
git checkout branch-name  
git switch branch-name

高频中的高频。

## 22. `which`

查看命令路径。

which python3  
which git

## 23. `command -v`

判断命令是否存在，也常用。

command -v node

## 24. `history`

查看历史命令。

history

配合搜索更好用：

history | grep git

## 25. `clear`

清屏。

clear

快捷键也很常用：

Ctrl + L

---

# 五、进程与系统排查

## 26. `ps`

查看进程。

ps aux | grep python  
ps aux | grep java

## 27. `top`

实时查看系统进程资源占用。

top

Mac 上也常用：

top -o cpu

## 28. `kill`

结束进程。

kill 12345  
kill -9 12345

## 29. `lsof`

查看端口占用、文件占用。

lsof -i :8080  
lsof -i tcp:3000

排查“端口被谁占了”时极其常用。

## 30. `netstat`

查看网络连接。

netstat -an

## 31. `df`

查看磁盘空间。

df -h

## 32. `du`

查看目录大小。

du -sh .  
du -sh *

---

# 六、网络相关

## 33. `curl`

请求接口、下载内容、调试 HTTP。

curl https://example.com  
curl -X POST https://api.example.com \  
  -H "Content-Type: application/json" \  
  -d '{"name":"test"}'

后端开发几乎每天都会用到。

## 34. `ping`

测试网络连通性。

ping google.com

## 35. `ssh`

远程登录服务器。

ssh user@host

## 36. `scp`

传文件到远程机器。

scp file.txt user@host:/tmp/  
scp -r mydir user@host:/tmp/

---

# 七、权限相关

## 37. `chmod`

修改权限。

chmod +x script.sh  
chmod 755 script.sh

## 38. `chown`

修改文件属主。

sudo chown yourname file.txt

## 39. `sudo`

以管理员权限执行。

sudo lsof -i :80

---

# 八、环境变量与 Shell 高频操作

## 40. `echo`

输出内容或变量。

echo hello  
echo $PATH  
echo $HOME

## 41. `export`

设置环境变量。

export GOPATH=$HOME/go  
export PATH=$PATH:/usr/local/bin

## 42. `env`

查看环境变量。

env

## 43. `source`

让配置文件立即生效。

source ~/.zshrc

Mac 默认通常是 `zsh`，所以你以后最常改的是：

~/.zshrc

---

# 九、压缩解压

## 44. `tar`

打包解压。

tar -czvf archive.tar.gz mydir  
tar -xzvf archive.tar.gz

## 45. `zip` / `unzip`

zip -r demo.zip demo/  
unzip demo.zip

---

# 十、Mac 特别常用的几个命令

## 46. `pbcopy`

把内容复制到剪贴板。

echo "hello" | pbcopy  
cat file.txt | pbcopy

## 47. `pbpaste`

读取剪贴板内容。

pbpaste

这两个是 Mac 终端神器。

## 48. `defaults`

读写一些 macOS 配置，进阶会用到。

defaults read

平时不算最高频，但偶尔很好用。

## 49. `say`

让 Mac 朗读文本。

say "hello world"

偏娱乐，但很有 Mac 味。

---

# 十一、最建议你先掌握的 15 个

如果你现在还不是 CLI 熟手，先把这 15 个练熟，工作里就够用了：

pwd  
ls -la  
cd  
mkdir -p  
touch  
cp  
mv  
rm -rf  
cat  
less  
tail -f  
grep -rn  
find  
git status  
lsof -i :8080

再加上：

curl  
ps aux | grep  
kill  
source ~/.zshrc  
open .  
pbcopy

---

# 十二、几个非常实用的组合拳

## 看日志

tail -f app.log  
grep "ERROR" app.log  
tail -f app.log | grep "ERROR"

## 查端口被谁占用

lsof -i :8080  
kill -9 PID

## 查某类文件

find . -name "*.md"  
find . -name "*.go"

## 搜代码

grep -rn "context.WithTimeout" .  
grep -rn "TODO" .

## 打开当前项目目录

open .

## 修改 shell 配置后立即生效

source ~/.zshrc

---

# 十三、你要特别小心的命令

下面这几个最容易误伤：

rm -rf  
kill -9  
sudo  
chmod -R  
chown -R

尤其是这种：

rm -rf /  
rm -rf ./*  
sudo rm -rf 某目录

一定要确认路径再执行。

---

# 十四、给你的建议

如果你是做开发，最值得优先练的是三类：

1. **文件操作**：`ls`、`cd`、`cp`、`mv`、`rm`
2. **搜索排查**：`grep`、`find`、`tail -f`、`lsof`
3. **工程开发**：`git`、`curl`、`ssh`