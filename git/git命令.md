#### 常用命令

1、查看状态： git status

2、将工作区的“新建/修改”添加到暂存区：git add

3、将暂存区的内容提交到本地库：git commit -m"xxx" [filename]

4、查看历史记录： git log （多屏显示控制方式 ：空格向下翻页、b向上翻页、q退出）

git log --pretty=oneline(简洁一些)

git log --oneline(更简洁)

git reflog (比上一条命令多显示HEAD@{},表示移动到当前版本需要多少步)

5、前进后退

+ 基于索引值操作（推荐）：git reset --hard [局部索引值]
+ 使用^符号（只能后退,一个^表示后退一步）：git reset --hard HEAD^
+ 使用~符号（只能后退，n表示几步）：git reset --hard HEAD~n

6、reset命令的三个参数对比

+ --soft 仅仅在本地库移动HEAD指针
+ --mixed  在本地库移动HEAD指针，重置暂存区
+ --hard 在本地库移动HEAD指针，重置暂存区、重置工作区 

7、删除文件并找回

+ git reset --head [指针位置]

8、比较文件差异

- 将工作区中的文件和暂存区进行比较 ：git diff【文件名】 

- 将工作区中的文件和本地历史库记录比较：git diff 【本地库中历史版本】【文件名】
- 不带文件名比较多个文件

9、分支

+ 创建  git branch [分支名]
+ 查看 git branch -v
+ 切换 git checkout [分支名]
+ 合并 1.切换到接受修改的分支 2.git merge [分支名]
+ 解决冲突

10、远程库

+ git remote -v 查看当前所有远程地址别名
+ git remote add 【地址别名】【远程库地址】

11、推送

git push 【地址别名】【分支名】

12、克隆

自动完成这三个操作：完整把远程库下载本地 、创建远程库地址别名、初始化本地库

git clone 【远程库地址】

13、拉取

+ pull = fetch + merge
+ git fetch 【远程库地址别名】【远程分支名】
+ git merge 【远程库地址别名/远程分支名】