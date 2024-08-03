# 0、常见问题

## Git Bash终端文件名都只显示数字串

原因：在默认设置下，中文文件名在工作区状态输出，中文名不能正确显示，而是显示为八进制的字符编码。

解决：

```bash
git config --global core.quotepath false
```

## fatal：refusing to merge unrelated histories

过程：

1. GitHub上新建仓库A；
2. 本地初始化仓库B，并设置remote orgin为远程仓库A的地址；
3. 本地执行pull或者push操作；

解决：

```bash
git pull origin master --allow-unrelated-histories
```

## Enter passphrase for key '本地SSH私钥地址'

原因：应该是生成公钥和私钥的时候设置了密码，导致每次操作git都需要输入密码。

解决：

```bash
# 命令
ssh-keygen -p [-P old_passphrase] [-N new_passphrase] [-f keyfile]
# sample
ssh-keygen -p -P 123456 -N '' -f ~/.ssh/id_rsa
```

## ^和~的差别

https://blog.csdn.net/albertsh/article/details/106448035

不加数字的情况下或者数字是1的情况下，^和~效果相同，都是指向第一个父节点：

![image-20240803150752770](./assets/image-20240803150752770.png)

数字大于2的情况下，~永远指向的是第一个父节点，连续退后至第一个父节点；而对于合并后的节点来说，存在2个父节点，所以^2则指向的是第2个父节点：

![image-20240803151142668](./assets/image-20240803151142668.png)

# 1、初始化

全局配置，所有仓库生效。

```bash
git config --global user.name ryualvin
git config --global user.email lsben1014@gmail.com
```

系统配置，对所有用户生效。

```bash
git config --global user.name ryualvin
git config --global user.email lsben1014@gmail.com
```

用户名中如果存在空格，需要加双引号。

```bash
git config --global user.name “ryu alvin”
```

保存用户名和密码：

```bash
git config --global credential.helper store
```

查看全局配置和系统配置：

```bash
git config --global --list
git config --system --list
```

# 2、创建仓库

> 方式一：创建本地仓库

```bash
git init
# 可指定仓库名
git init learn-repo
```

> 方式二：克隆一个远程仓库

```bash
git clone https://github.com/xxx/learn-repo.git
```

# 3、工作区域和文件状态

> Git的四个区域

![image-20240427181326918](./assets/image-20240427181326918.png)

- 工作区（Working Directory）：就是你再电脑里能看到的目录；
- 暂存区（Stage/Index）：一般存放在.git目录下的index文件，所以我们把暂存区有时也叫做索引（index）；
- 本地仓库（Repository）：工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库；
- 远程仓库（Remote）：托管再远程服务器上的仓库。

> Git的三种状态

![image-20240427181504566](./assets/image-20240427181504566.png)

- 已修改（Modified）：修改了文件，但没保存到暂存区；
- 已暂存（Staged）：把修改后的文件放到暂存区；
- 已提交（Committed）：把暂存区的文件提交到本地仓库。

```bash
# 查看状态
ryualvin ~/Desktop/GitTest/repo-ignore (master)
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        file4.txt

nothing added to commit but untracked files present (use "git add" to track)

# 简洁显示状态
ryualvin ~/Desktop/GitTest/repo-ignore (master)
$ git status -s
?? file4.txt

ryualvin ~/Desktop/GitTest/repo-ignore (master)
$ git add file4.txt

ryualvin ~/Desktop/GitTest/repo-ignore (master)
$ git status -s
A  file4.txt

# 简化命令 git status => git st
git config --global alias.st status
```

# 4、文件操作

移动一个文件到新的位置：

```bash
git mv <file> <new-file>
```

恢复一个文件到之前的版本：

```bash
git checkout <file> <commit-id>
```

撤销工作区的修改（只能撤销工作区的修改，且不可恢复，不会撤销暂存区修改）：

```
git checkout <file>
```

将文件从工作区添加至暂存区：

```bash
git add .
git add file1.txt
# 可使用通配符
git add *.txt
```

提交至本地仓库：

- 只会提交暂存区里有的内容；
- 不指定提交消息的话，会自动进入vim编辑模式。

```bash
git commit -m "这是第一次提交"
```

添加至暂存和提交两个操作（关联远程仓库后才可使用）：

```bash
git commit -a -m "feat:1"
git commit -am "feat:1"
```

给当前的提交打上标签，通常用于版本发布。

```bash
git tag "first release"
```

更改文件名：

```bash
git mv READ.md README
```

相当于：

```bash
mv README.md README
git rm README.md
git add README
```

# 5、RESET和REVERT

## 5.1、RESET的三种模式

![image-20240427184124816](./assets/image-20240427184124816.png)

### 5.1.1、HARD

回退到某一个版本，并且丢弃工作区和暂存区的所有修改内容：

```bash
ryualvin ~/Desktop/GitTest/repo-hard (master)
$ git log --oneline
88a7a69 (HEAD -> master) third commit
c6db37e second commit
6fb9bca first commit

ryualvin ~/Desktop/GitTest/repo-hard (master)
$ git reset --hard 6fb9bca
HEAD is now at 6fb9bca first commit

# 工作区和暂存区都只有第一次提交的file1
ryualvin ~/Desktop/GitTest/repo-hard (master)
$ git status
On branch master
nothing to commit, working tree clean

ryualvin ~/Deskt
$ ls
file1.txt
```

### 5.1.2、SOFT

回退到某一个版本，并且保留工作区和暂存区的所有修改内容：

```bash
ryualvin ~/Desktop/GitTest/repo-soft (master)
$ git log --oneline
88a7a69 (HEAD -> master) third commit
c6db37e second commit
6fb9bca first commit

ryualvin ~/Desktop/GitTest/repo-soft (master)
$ git reset --soft 6fb9bca

# 此时暂存区中有第二次和第三次提交的file2和file3
ryualvin ~/Desktop/GitTest/repo-soft (master)
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   file2.txt
        new file:   file3.txt

# git ls-files操作可查看暂存区中的所有文件
ryualvin ~/Desktop/GitTest/repo-soft (master)
$ git ls-files
file1.txt
file2.txt
file3.txt

# 此时工作区中有第二次和第三次提交的file2和file3
ryualvin ~/Desktop/GitTest/repo-soft (master)
$ ls
file1.txt  file2.txt  file3.txt
```

HEAD^：

```bash
ryualvin ~/Desktop/GitTest/repo-log (master)
$ git log --oneline
ef0ca57 (HEAD -> master) fifth commit
019d908 forth commit
6fb9bca first commit

# HEAD指向分支的最新提交节点
ryualvin ~/Desktop/GitTest/repo-log (master)
$ git reset --soft HEAD^

ryualvin ~/Desktop/GitTest/repo-log (master)
$ git log --oneline
019d908 (HEAD -> master) forth commit
6fb9bca first commit
```

HEAD~：

```bash
ryualvin ~/Desktop/GitTest/repo-log (master)
$ git log --oneline
bb9851d (HEAD -> master) sixth commit
019d908 forth commit
6fb9bca first commit

ryualvin ~/Desktop/GitTest/repo-log (master)
$ git reset --soft HEAD~2

ryualvin ~/Desktop/GitTest/repo-log (master)
$ git log --oneline
6fb9bca (HEAD -> master) first commit
```

指定具体文件回退：

```bash
$ git reset --soft HEAD^ file1.txt
$ git reset --soft HEAD~2 file1.txt
```

### 5.1.3、MIXED

回退到某一个版本，只保留工作区的修改内容，丢弃暂存区的修改内容：

```bash
ryualvin ~/Desktop/GitTest/repo-mixed (master)
$ git log --oneline
88a7a69 (HEAD -> master) third commit
c6db37e second commit
6fb9bca first commit

ryualvin ~/Desktop/GitTest/repo-mixed (master)
$ git reset --mixed 6fb9bca

# 此时工作区中有第二次和第三次提交的file2和file3
ryualvin ~/Desktop/GitTest/repo-mixed (master)
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        file2.txt
        file3.txt

nothing added to commit but untracked files present (use "git add" to track)

ryualvin ~/Desktop/GitTest/repo-mixed (master)
$ ls
file1.txt  file2.txt  file3.txt

# 但暂存区中有只有第一次提交的file1
ryualvin ~/Desktop/GitTest/repo-mixed (master)
$ git ls-files
file1.txt
```

## 5.2、REVERT

撤销某次操作，此次操作之前和之后的提交记录都会保留，并且把这次撤销操作作为一次最新的提交版本。

```bash
# 撤销最新的一次提交
git revert HEAD
# 撤销前一次提交
git revert HEAD^
# 撤销某一次提交
git revert <版本号>
```

## 5.3、RESET和REVERT的区别

Revert是用一次新的commit来回滚之前的commit，Reset是直接删除指定commit。

# 6、版本信息和操作记录

查看提交过的版本信息：

```bash
git log
# 一行显示
git log --oneline
```

查看所有操作记录：

```bash
git reflog --oneline
```

git log和git reflog的区别：

```bash
ryualvin ~/Desktop/GitTest/repo-log (master)
$ git log --oneline
88a7a69 (HEAD -> master) third commit
c6db37e second commit
6fb9bca first commit

ryualvin ~/Desktop/GitTest/repo-log (master)
$ git reflog --oneline
88a7a69 (HEAD -> master) HEAD@{0}: commit: third commit
c6db37e HEAD@{1}: commit: second commit
6fb9bca HEAD@{2}: commit (initial): first commit

# 做回退撤销操作
ryualvin ~/Desktop/GitTest/repo-log (master)
$ git reset --soft 6fb9bca

# 因为回退至第一次提交，第二次和第三次提交都被撤销了，所以git log只显示了第一次提交的版本信息
ryualvin ~/Desktop/GitTest/repo-log (master)
$ git log --oneline
6fb9bca (HEAD -> master) first commit

# 但git reflog会显示用户所有的操作记录，相当于后悔药，任何时候都可以恢复到想要的那个版本
ryualvin ~/Desktop/GitTest/repo-log (master)
$ git reflog --oneline
6fb9bca (HEAD -> master) HEAD@{0}: reset: moving to 6fb9bca
88a7a69 HEAD@{1}: commit: third commit
c6db37e HEAD@{2}: commit: second commit
6fb9bca (HEAD -> master) HEAD@{3}: commit (initial): first commit
```

查看图形化提交记录：

```bash
git log --oneline --graph --decorate --all
# 命令太长可定义别名
alias graph="git log --oneline --graph --decorate --all"
# 定义了别名之后就可以用别名查看
graph
```

# 7、查看差异

```bash
# 查看工作区修改（工作区 VS 本地仓库）
git diff

# 查看本地仓库修改（工作区/暂存区 VS 本地仓库）
git diff HEAD

# 查看暂存区修改（暂存区 VS 本地仓库）
git diff --cached
git diff --staged

# 比较版本之间的差异
git diff 88a7a69 6fb9bca

# 比较当前版本和前个版本之间的差异
git diff HEAD~ HEAD

# 比较分支之间的差异
git diff main dev

# 指定具体文件名
git diff 88a7a69 6fb9bca file2.txt
```

**一般使用GUI工具查看差异会更方便些。**

# 8、删除文件

## 8.1、先删除后提交

```bash
# 先通过LINUX命令删除文件
ryualvin ~/Desktop/GitTest/repo-rm (master)
$ rm file4.txt

# 此时工作区中file4已被删除，提示需要将该file4被删除的状态提交到暂存区
ryualvin ~/Desktop/GitTest/repo-rm (master)
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    file4.txt

no changes added to commit (use "git add" and/or "git commit -a")

# 将被删除状态提交至暂存区
ryualvin ~/Desktop/GitTest/repo-rm (master)
$ git add file4.txt

ryualvin ~/Desktop/GitTest/repo-rm (master)
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    file4.txt
        
# 将file4被删除的状态提交至本地仓库，即让本地仓库删除该文件
ryualvin ~/Desktop/GitTest/repo-rm (master)
$ git commit -m "delete file4"
[master 87f7c64] delete file4
 1 file changed, 1 deletion(-)
 delete mode 100644 file4.txt
```

## 8.2、Git删除

### 8.2.1、把文件从工作区和暂存区同时删除

```bash
git rm readme.md
```

相当于：

```bash
rm readme.md
git add .
```

### 8.2.2、把文件从暂存区删除但保留在工作区

```bash
git rm --cached readme.md
```

### 8.2.3、递归删除某个目录下的所有子目录和文件

```bash
ryualvin ~/Desktop/GitTest/repo-rm (master)
$ ls
file1.txt  file2.txt  file3.txt  file5.txt  test-rm/

# 进入目标文件夹
ryualvin ~/Desktop/GitTest/repo-rm (master)
$ cd test-rm

# 递归删除该文件夹下的所有子目录和文件
ryualvin ~/Desktop/GitTest/repo-rm/test-rm (master)
$ git rm -r *
rm 'test-rm/test-rm-1/test-rm.txt'

ryualvin ~/Desktop/GitTest/repo-rm/test-rm (master)
$ cd ..

ryualvin ~/Desktop/GitTest/repo-rm (master)
$ ls
file1.txt  file2.txt  file3.txt  file5.txt  test-rm/

ryualvin ~/Desktop/GitTest/repo-rm (master)
$ cd test-rm

# 目标文件夹下的所有子目录和文件夹都已被删除
ryualvin ~/Desktop/GitTest/repo-rm/test-rm (master)
$ ls -altr
total 4
drwxr-xr-x 1 ryualvin 197121 0 May  4 16:21 ../
drwxr-xr-x 1 ryualvin 197121 0 May  4 16:21 ./
```

**注意：以上删除操作后不要忘记提交！！！**

# 9、忽略文件

## 9.1、应该忽略哪些文件

- 系统或者软件自动生成的文件；
- 编译产生的中间文件和结果文件；
- 运行时生成日志文件、缓存文件、临时文件；
- 涉及身份、密码、口令、密钥等敏感信息文件。

## 9.2、匹配规则

> 规则

- 从上到下逐行匹配，每一行表示一个忽略模式
- 空行或者以#开头的行会被Git忽略。一般空行用于可读性的分离，#一般用作注释；
- 使用标准的Blob模式匹配，例如：
  - 星号*通配任意个字符；
  - 问好?匹配单个字符；
  - 中括号[]表示匹配列表中的单个字符，比如：[abc]表示a/b/c；
- 两个星号**表示匹配任意的中间目录；
- 中括号可以使用短中线链接，比如：
  - [0-9]表示任意一位数字，[a-z]表示任意一位小写字母；
- 感叹号!表示取反。

> 举例

```bash
# 忽略所有的 .a 文件
*.a
# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a
# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO，/ 表示根目录
/TODO
# 忽略任何目录下名为 build 的文件夹
build/
# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt
# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

**在Github上提供了各种常用语言的忽略文件的模板，在新建仓库的时候可以直接使用：**

https://github.com/github/gitignore

## 9.3、注意点

- 即使在.gitignore中增加已经被纳入版本控制的文件，也起不到忽略作用。必须先将文件从版本库中移出，才会起到忽略作用：

  ```bash
  # 将temp文件忽略
  ryualvin ~/Desktop/GitTest/repo-ignore (master)
  $ cat .gitignore
  temp.txt
  
  # 将文件移出版本库
  ryualvin ~/Desktop/GitTest/repo-ignore (master)
  $ git rm --cached temp.txt
  rm 'temp.txt'
  
  # 对该文件做修改
  ryualvin ~/Desktop/GitTest/repo-ignore (master)
  $ echo "add line2" >> temp.txt
  
  # 修改被忽略
  ryualvin ~/Desktop/GitTest/repo-ignore (master)
  $ git status
  On branch master
  nothing to commit, working tree clean
  ```

- 空文件夹，不会被纳入版本控制：

  ```bash
  # 创建空文件夹
  ryualvin ~/Desktop/GitTest/repo-ignore (master)
  $ mkdir emptyfolder
  
  # 空文件夹不被跟踪
  ryualvin ~/Desktop/GitTest/repo-ignore (master)
  $ git status
  On branch master
  nothing to commit, working tree clean
  
  # 在文件夹中添加文件
  ryualvin ~/Desktop/GitTest/repo-ignore (master)
  $ echo "line1" >> emptyfolder/newfile1.txt
  
  # 状态显示有未被跟踪的文件
  ryualvin ~/Desktop/GitTest/repo-ignore (master)
  $ git status
  On branch master
  Untracked files:
    (use "git add <file>..." to include in what will be committed)
          emptyfolder/
  
  nothing added to commit but untracked files present (use "git add" to track)
  ```

# 10、克隆远程仓库

HTTPS方式在把本地代码push到远程仓库的时候，需要验证用户名和密码。

SSH方式在推送的时候不需要验证用户名和密码，但是需要在GitHub上添加SSH公钥（锁）的配置。

**注意：在2021年8月13日以后，HTTPS的这种方式已经被GitHub停止使用，所以推荐大家使用SSH的方式。**

## 10.1、生成公钥（锁）和密钥

```bash
ryualvin /d/develop/git/SSH-Key
# -t rsa：指定协议
# -f：指定文件名
# -C：备注
$ ssh-keygen -t rsa -f id_rsa_github_Ryuxxx -C "github ryualvin ssh"
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa_github_Ryuxxx
Your public key has been saved in id_rsa_github_Ryuxxx.pub
The key fingerprint is:
SHA256:5Qlk+NdULurwxxxxxIMCPA9Mj72J7QpZPrUHM github ryualvin ssh
The key's randomart image is:
+---[RSA 3072]----+
| xxxxxxxxxxxxxxx |
+----[SHA256]-----+
```

## 10.2、配置公钥（锁）

将公钥（锁）配置到GitHub：

![image-20240504213434384](./assets/image-20240504213434384.png)

## 10.3、指定私钥

如果是第一次配置，并且在创建密钥的时候没有修改过默认的文件名的话，SSH密钥的配置到这里就完成了。

但是如果在生成密钥的时候指定新的文件名，或者对于不同的网站需要配套不同的SSH密钥的时候，就需要进一步在config文件中配置：D:\develop\git\Git\etc\ssh\ssh_config

```bash
# 访问gitee的SSH密钥
# Host别名
Host gitee_Ryuxxx
# gitee地址
 	HostName gitee.com
# git@gitee.com
 	User git
# 通过.pub文件认证
 	PreferredAuthentications publickey
# 密钥文件地址
 	IdentityFile D:\develop\git\SSH-Key\id_rsa_gitee_Ryuxxx

# 访问github的SSH密钥
# Host别名
Host github_Ryuxxx
# github地址
 	HostName github.com
# git@github.com
 	User git
# 通过.pub文件认证
 	PreferredAuthentications publickey
# 密钥文件地址
 	IdentityFile D:\develop\git\SSH-Key\id_rsa_github_Ryuxxx
```

## 10.4、克隆

```bash
ryualvin ~/Desktop/GitTest
# git@github.com:Ryuxxx/remote-repo.git ->
# github_Ryuxxx:Ryuxxx/remote-repo.git
$ git clone github_Ryuxxx:Ryuxxx/remote-repo.git
Cloning into 'remote-repo'...
# 输入创建SSH密钥时候的密码
Enter passphrase for key 'D:\develop\git\SSH-Key\id_rsa_github_Ryuxxx':
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
# 克隆成功
Receiving objects: 100% (3/3), done.
```

# 11、关联本地和远程仓库

```bash
# 本地仓库local-repo，远程仓库remote-repo
ryualvin ~/Desktop/GitTest/local-repo (master)
$ git remote add origin github_Ryuxxx:Ryuxxx/remote-repo.git

ryualvin ~/Desktop/GitTest/local-repo (master)
# 拉取远程仓库
# 本地和远程仓库中都存在一个file1文件，且文件内容不一致
# --allow-unrelated-histories：该选项可以合并两个独立启动仓库的历史（实际上本地和远程是两个独立的仓库），不加该选项会出现错误：fatal: refusing to merge unrelated histories
$ git pull origin master --allow-unrelated-histories
Enter passphrase for key 'D:\develop\git\SSH-Key\id_rsa_github_Ryuxxx':
remote: Enumerating objects: 9, done.
# ...
From github_Ryuxxx:Ryuxxx/remote-repo
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
Auto-merging file1.txt
CONFLICT (add/add): Merge conflict in file1.txt
Automatic merge failed; fix conflicts and then commit the result.

ryualvin ~/Desktop/GitTest/local-repo (master|MERGING)
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)
# 来自远程仓库的新文件file2
Changes to be committed:
        new file:   file2.txt
# 来自远程仓库的冲突文件file1，需要手动解决冲突内容
Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both added:      file1.txt

ryualvin ~/Desktop/GitTest/local-repo (master|MERGING)
$ cat file1.txt
<<<<<<< HEAD
local file1
=======
file11111
2222222222222222
>>>>>>> 85b263ebc71c8380f6ae3cf7403a1979178e0f29

# 解决冲突后推送到远程
ryualvin ~/Desktop/GitTest/local-repo (master)
$ git push origin master
Enter passphrase for key 'D:\develop\git\SSH-Key\id_rsa_github_Ryuxxx':
# ...
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
To github_Ryuxxx:Ryuxxx/remote-repo.git
   85b263e..b5fd819  master -> master
```

#  12、分支操作

```bash
# 查看分支列表
git branch

# 创建分支（只创建，未切换）
git branch dev

# 切换分支
git checkout main
# checkout还能恢复一个文件或目录到之前的某一个状态。比如我们可能意外地修改了某一个文件，这个时候就可以使用git checkout命令来恢复我们到我们修改之前的状态，而这个时候如果分支名称和文件名称相同的话就会出现歧义，因为git checkout命令会默认切换分支而不是恢复文件。为了避免这种歧义，Git官方在2.23版本开始为我们提供了一个新的命令（git switch）专门用来切换分支。
git switch main

# 创建分支并切换
git checkout -b dev
# 创建分支并指定回退到某个时间点的版本号
git checkout -b dev 244d35

# 合并分支（在main分支上合并dev分支的内容）
git merge dev

# 删除已合并分支
git branch -d dev
# 若删除未合并分支会报以下错误
error: The branch 'newDev' is not fully merged.
If you are sure you want to delete it, run 'git branch -D newDev'.

# 删除未合并分支（强制删除）
git branch -D dev

# 修改分支名
git branch -m dev newDev
```

合并分支a到分支b，--no-ff参数表示禁用Fast forward模式，合并后的历史有分支，能看出曾经做过合并。

```bash
git merge --no-ff -m "message" <branch-name>
```

![image-20240505215035739](./assets/image-20240505215035739.png)

--ff参数表示使用Fast forward模式，合并后的历史会变成一条直线。

```bash
git merge --ff -m "message" <branch-name>
```

![image-20240505215231459](./assets/image-20240505215231459.png)

合并&压缩所有提交到一个提交。

解决的是什么问题？由于在dev分支上执行的是开发工作，有一些很小的提交，或者是纠正前面的错误的提交，对于这类提交对整个工程来说不需要单独显示出来一次提交，不然导致项目的提交历史过去复杂；所以基于这种原因，我们可以把dev上的所有提交都合并成一个提交，然后提交到主干。

**注意：squash merge并不会替你产生提交，它只是把所有的改动合并，然后放在本地文件，需要你再次手动执行提交操作。**

```bash
ryualvin ~/Desktop/GitTest/local-repo-squash (master)
$ git log --oneline --graph --decorate --all
* d28ac8b (HEAD -> master) master:5
* a868a83 master:4
| * 5429c38 (dev) dev:2
| * b6163a2 dev:1
|/
* 71df0b0 master:3
* 5994758 master:2
* f03bbfd master:1

ryualvin ~/Desktop/GitTest/local-repo-squash (master)
$ git branch
  dev
* master

ryualvin ~/Desktop/GitTest/local-repo-squash (master)
$ git merge --squash dev
Automatic merge went well; stopped before committing as requested
Squash commit -- not updating HEAD

ryualvin ~/Desktop/GitTest/local-repo-squash (master)
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   dev1.txt
        new file:   dev2.txt

ryualvin ~/Desktop/GitTest/local-repo-squash (master)
$ git commit -m "squash commit"
[master 8d5ced0] squash commit
 2 files changed, 2 insertions(+)
 create mode 100644 dev1.txt
 create mode 100644 dev2.txt

ryualvin ~/Desktop/GitTest/local-repo-squash (master)
$ git log --oneline --graph --decorate --all
* 8d5ced0 (HEAD -> master) squash commit
* d28ac8b master:5
* a868a83 master:4
| * 5429c38 (dev) dev:2
| * b6163a2 dev:1
|/
* 71df0b0 master:3
* 5994758 master:2
* f03bbfd master:1
```

# 13、解决合并冲突

1. 两个分支未修改同一个文件的同一处位置：Git自动合并；

2. 两个分支修改了同一个文件的同一处位置：产生冲突；

   - 解决方法：

     ```bash
     ryualvin ~/Desktop/GitTest/local-repo (master)
     $ git merge dev
     Auto-merging file1.txt
     CONFLICT (content): Merge conflict in file1.txt
     Automatic merge failed; fix conflicts and then commit the result.
     
     # 可用git status查看产生冲突的文件
     $ git status
     On branch master
     You have unmerged paths.
       (fix conflicts and run "git commit")
       (use "git merge --abort" to abort the merge)
     
     Unmerged paths:
       (use "git add <file>..." to mark resolution)
             both modified:   file1.txt
     
     no changes added to commit (use "git add" and/or "git commit -a")
     
     # 也可用git diff 查看产生冲突的文件
     $ git diff
     diff --cc file1.txt
     index ae6642c,a4abf37..0000000
     --- a/file1.txt
     +++ b/file1.txt
     @@@ -1,4 -1,4 +1,8 @@@
       local file1
       file11111
       2222222222222222
     ++<<<<<<< HEAD
      +master:1
     ++=======
     + dev:1
     ++>>>>>>> dev
     
     # 手动修改冲突文件，合并冲突内容后提交
     $ git commit -am "merge conflict"
     ```

   - 中止合并：

     ```bash
     git merge --abort
     ```

# 14、Rebase

## 14.1、什么是Rebase

Git会先找到当前分支和目标分支的共同祖先，这里也就是main:3这个节点。再把当前分支上从共同祖先到最新提交节点的所有提交都嫁接移植到目标分支的最新提交节点后面。

> 在dev分支上做变基操作

![image-20240505210443018](./assets/image-20240505210443018.png)

> 在main分支上做变基操作

![image-20240505210533284](./assets/image-20240505210533284.png)

## 14.2、Rebase和Merge比较

### 14.2.1、区别

Rebase不会产生新的提交，而是把当前分支的每一个提交都复制到目标分支上，然后再把当前分支指向目标分支。

Merge会产生一个新的提交，这个提交有两个分支的所有修改。

> Merge的优缺点

优点：不会破坏原分支的提交历史，所有提交记录和合并历史都会保留下来，方便后续查看和回滚。

缺点：会产生额外的提交记录和两条分支线的合并，会使得提交记录变得复杂。

> Rebase优缺点

优点：不需要新增额外的提交记录到目标分支，可以形成一个线性的提交历史记录，非常直观和干净。

缺点：会改变提交历史，改变当前分支branch out的节点。因此避免在公共分支上进行变基操作，会对和你一起在这个分支上开发的同事造成一些困扰（多人协作时，不要对已经推送到远程的分支执行Rebase操作）。

### 14.2.2、如何选择

仁者见仁智者见智的问题。

一般来说，如果你只是想把两个分支合并起来而不关心提交历史的话，那么就可以使用git merge命令。

如果你确定只有你自己再一个分支上开发，并且希望提交历史更加的清晰明了，那么就建议使用git rebase命令。

# 15、Stash

Stash操作可以把当前工作现场储藏起来，等以后恢复现场后继续工作。

-u参数表示把所有未跟踪的文件也一起存储，-a参数表示把所有未跟踪的文件和忽略的文件也一并存储，save参数表示存储的信息，可以不写。

```bash
git stash save "message"
```

查看所有stash：

```bash
git stash list
```

恢复最近一次stash（**弹栈，恢复后旧从stash list中删除**）：

```bash
git stash pop
```

恢复指定的stash，stash@{2}表示第三个stash，stash@{0}表示最近的stash。

```bash
git stash pop stash@{2}
```

重新接收最近一次stash（**只应用，不弹栈，恢复后不从stash list中删除**）：

```bash
git stash apply
```

接收指定的stash，stash@{2}表示第三个stash，stash@{0}表示最近的stash。

```
git stash apply stash@{2}
```

pop和apply的区别是，pop会把stash内容删除，而apply不会。可以用git stash drop来删除stash。

```bash
git stash drop stash@{2}
```

删除所有stash：

```bash
git stash clear
```

# 16、Git Flow

GitFlow是一种流程模型，用于在Git上管理软件开发项目。

- 主分支（master）：代表了项目的稳定版本，每个提交到主分支的代码都应该是经过测试和审核的；
- 开发分支（develop）：用于日常开发。所有功能分支、发布分支和修补分支都应该从develop分支派生；
- 功能分支（feature）：用于开发单独的功能或特性。每个功能分支应该从develop分支派生，并在开发完成后合并回develop分支；
- 发布分支（release）：用于准备项目发布。发布分支应该从develop分支派生，并在准备好发布版本后合并回master和develop分支；
- 修补分支（hotfix）：用于修复主分支上的紧急问题。修补分支应该从master分支派生，并在修复完成后合并回master和develop分支。
