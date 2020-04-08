# git常用指令

## 创建

创建用户，设置用户名、邮箱

   `git config --global user.name "zhangbowen"`

   `git config --global user.email  "zbwbiehua@163.com"`

## 初始化

初始化本地git仓库，并关联远程仓库，有两种方法：

1. 克隆远程仓库到本地，建立本地仓库，并自动建立关联关系。也用于备份仓库

   `git clone [仓库地址]`

2. 初始化本地git仓库

   `git init`

   关联远程仓库

   `git remote add origin [仓库地址]`

   推送master分支，建立远程master分支，-u是添加关联

   `git push -u origin master`

## 远程

1. 查看当前仓库关联的远程仓库

   `git remote -v`

## 拉取

1. 从远程分支获取最新的版本，并merge到本地当前分支

   `git pull origin [远程分支名称]`

2. 对于已经关联远程分支的本地当前分支，可以直接使用

   `git pull`

## 分支

1. 切换分支

   `git checkout [分支名称]`

2. 基于当前分支新建一个分支，并切换到新分支

   `git checkout -b [分支名称]`

3. 查看分支信息

   `git branch`

4. 删除分支

   `git branch -D [分支名称]`

5. 把本地的dev分支强制覆盖并推送到远程master

   `git push origin dev:master -f`

6. 使用本地dev分支强制覆盖本地当前分支后，再强制推送

   `git reset –hard dev`

   `git push origin master -f`

## 提交

1. 将当前目录所有文件添加到暂存区

   在WebStorm中文件由红色变为绿色，将文件添加到版本管理

   `git add .`

2. 提交暂存区到仓库区（当前分支稳定代码），-m为加注释

   在WebStorm中文件由绿色变为白色，更新当前分支代码

   `git commit -m [描述信息]`

3. 查看提交历史

   `git log`

4. 显示当前目录文件与暂存区状态变化

   比较当前分支暂存区和仓库区（commit后，当前分支稳定代码）区别

   `git status`

5. 对于新建的本地分支，push到远程仓库，-u是添加关联

   `git push -u origin [分支名称]`

6. 对于已关联远程分支的本地当前分支，可以直接使用

   `git push`

7. 修改最近一次commit的message

   `git commit --amend`

8. 修改旧的commit的message，要使用rebase

   `git rebase -i [目标commit的上一个commitId]`

   找到目标commit，选择r命令，只修改commit的message

9. 整理合并几个连续的commit

    `git rebase -i [需要合并的目标commit列表中的最上面的一个commit的上一个commitId]`

    使用s命令，将后面的commit合并到前面的commit

10. 整理合并不连续的commit

    `git rebase -i [需要合并的目标commit列表中的最上面的一个commit的上一个commitId]`

    手动编辑commit的顺序，其他同上

## 比较

1. 比较暂存区和HEAD差异

   `git diff cached`

2. 比较工作区和暂存区差异

   `git diff`

## 合并

1. 合并其他分支代码到当前分支

   `git merge [其他分支名称]`

## 还原、撤销

revert：撤销。一般用于删除某个commit，在log中会有记录

1. 撤销某个提交过的commit

   `git revert [commit-id]`

   执行这个操作时，git需要一条新的commit记录这次操作

reset：回滚。一般在"线上事故"通常需要回滚到想要的版本（分支），再提交到远程分支，解决问题

1. 撤回当前代码到历史某一版本号代码

   `git reset [commit-id]`

2. 回滚到某个版本，强推到远程分支，发布。一般用于"线上事故"

   `git reset [commit-id]`

   对于已关联的分支：`git push -f`

   对于未关联的分支：`git push origin HEAD:[branch-name] -f`

3. 撤销工作目录中所有未提交文件的修改内容

   `git reset --hard HEAD`

4. 将暂存区所有文件还原到HEAD版本

   `git reset HEAD`

5. 将暂存区部分文件还原到HEAD版本

   `git reset HEAD -- [目标文件]`

6. 将工作区部分文件还原到暂存区版本

   `git checkout -- [目标文件]`

## 打标签

标签是与分支是并行的两个概念，我们可以检出分支，得到指定版本代码，也可以通过检出tag，得到指定版本代码，tag方式一般用于线上发布做标记。

1. 查看tag

   `git tag`

2. 删除本地tag

   `git tag -d [tag-name]`

3. 给当前版本打一个tag

   `git tag v2019-04-01`

   `git tag -a [标签名称] -m [标签描述]`

4. 将标签推送到远程仓库

   `git push origin [标签名称]`

5. 查看标签详情

   `git show [标签名]`

6. 删除远程tag

   先删除本地tag，再推到远程仓库

   `git tag -d [tag-name]`

   `git push origin :refs/tags/[标签名]`

7. 新建一个本地tag

   `git tag v2019-09-19`

## 储藏

1. 储藏当前状态

   `git stash`

2. 查看所有储藏状态列表

   `git stash list`

3. 导出指定储藏状态

   `git stash apply stash@{index}`

4. 删除指定储藏状态

   `git stash drop stash@{index}`

5. 重新应用，并移除储藏

   `git stash pop`

## 修改

1. 修改最后一次改动

   `git commit --amend`

## 变基

rebase常用两个功能：

1. 合并多次描述冗余的commit为一次精简的commit

   推荐：[【Git】rebase 用法小结](https://www.jianshu.com/p/4a8f4af4e803)

   git rebase -i HEAD~4  // 从当前commit起，包括当前，选择处理前4个commit，s指令表示与之前的commit进行合并

2. 保证提交分支图优雅（直线型）展示

   改变当前分支commit基础，在新的基础上追加当前分支自己的commit，当前分支rebase完成后，再合并到其他分支，就不会出现像下方的无意义提交记录了

   `Merge branch '$test$' into bugfix`

   例：多人协作时，主分支为bugfix，成员A在bugfix_A上完成自己的bug修复，提交commit，切到本地bugfix，pull下了B的commit

   方式一：

   如果直接将bugfix_A分支merge到bugfix，会出现提交记录中多出一条

   `Merge branch 'bugfix_A' into bugfix`

   这并不是我们希望的

   方式二：

   成员A切回到自己的bugfix_A分支，使用 `git rebase bugfix` ,将自己的commit在此时的bugfix分支代码基础上（此分支已更新有成员B的commit）提交

   完成rebase后，在将bugfix_A分支merge到bugfix分支，此时发现并没有任何无意义的提交记录，push到远程分支

   rebase 过程中可能会遇到冲突，因为相当于把成员B的代码和成员A的代码做了一次整合

   解决完冲突之后，使用 `git add .`，标记为已解决

   再 `git rebase --continue` 完成rebase

## Q&A

1. **执行 git merge 时分支处于merging状态，提示MERGE_MSG.swp文件报错**

   执行 git pull 或者 push 或者 merge 命令操作的时候，当前分支一直处于merging状态

   当打开 git/MERGE_MSG 文件的时候，发现有 git/MERGE_MSG.swp 文件的存在，并且从时间上来看， MERGE_MSG 比 MERGE_MSG.swp 要更新

   **关于 MERGE_MSG.swp 文件的说明：**

   .swp 文件和 git 无关，在使用 VIM 开始编辑某文件时，都会产生该文件对应的 .swp 文件。正常的退出，VIM 会自动删除此类型文件，非正常退出情况下， VIM 不会删除 ，.swp 文件会作为文件编辑状态的内容备份

   其实多次打开多次不正常关闭，会一直产生 .sw* 文件

   **问题解决：**

   第一步：回到合并前状态

   `git merge -abort` 或者 `git reset --hard HEAD`

   第二步：删除 vim 非正常关闭产生的文件

   rm .git/.MERGE_MSG.sw*

   第三步：重新合并

   `git merge [分支名称]`

   使用`:wq`正常安全退出

2. 指定不需要git管理的文件

   使用.gitignore文件，/结尾指文件夹，后缀名结尾指具体文件

3. 设置ssh，配置公私钥

   创建私钥：`ssh-keygen -t rsa -b 4096 -C "[邮箱地址]"`

   一路回车

   cd ~/.ssh    寻找ssh文件夹

   cat id_rsa.pub   复制粘贴key

   粘贴到github的账号  setting——ssh and GPG key

## Commit命名规范

``` js
feat：新功能（feature）
fix：修补bug
docs：文档（documentation）
style： 格式（不影响代码运行的变动）
refactor：重构（即不是新增功能，也不是修改bug的代码变动）
test：增加测试
chore：构建过程或辅助工具的变动
```

摘自[《阮一峰 Commit message 和 Change log 编写指南》](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

## 资料

- [廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600)
