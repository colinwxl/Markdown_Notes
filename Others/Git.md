## [git](https://github.com/colinwxl)
/how to and management/
[《Pro Git》](https://git-scm.com/book/zh/v2) <chinese version>
### How To [git commands]
1. Git Manual
`man git-log` or `git help log`

2. cogigure Git：
```
git config --global user.name "My Name"
git config --global user.email myEmail@example.com
```
3. create new empty repository
```
mkdir git_exercise
cd git_exercise
git init  ## init a root empty repository: git init --bare test_repo
```

4. detecting status
```
$ touch hello.txt
$ git status
Initial commit
Untracked files:

  (use "git add ..." to include in what will be committed)

	hello.txt
```
5. staging: `git add .`
This tells Git to take a snapshot of the contents of all files under the current directory(note the .) with *git add*. The snapshot is now stored in a temporary staging area which Git calls the "index".
<git 有个概念叫 暂存区，你可以把它看成一块空白帆布，包裹着所有你可能会提交的变动。>

6. You can permanently store the contents of the index in the repository with *git commit*: `git commit`
`git commit -m "commit message"`

7. connect to the remote repository: `git remote add`
`git remote add origin https://github.com/colinwxl/test`

8. update to remote repository
`git push origin[local branch] master[remote branch]`

9. creating branch:
checking branches: `git branch`, * mark out the activative branch
creating new branch: `git branch [new branch name]`

10. branched switching: `git checkout [branch name]`

11. branches merging:
```
git branch new_branch # create a new branch
git checkout new_branch
touch feature.txt # create a file under new_branch
git checkout master # ls no feature.txt under master
git merge new_branch
git branch -d new_branch # remove a branch which has already been used in merging
git branch -D new_branch # remove a branch which not participates in any merging
```

12. clone a repository: `git clone https://github.com/username/repository`
`git clone username@host:/path/to/repository`

13. update the local repo from the remote repo:
`git pull [repository name] [branch name]:[local branch]`

14. tracking commits
```
git log # get commit id
git show [id] # check out what happend (only a little part of the id)
``` 

15. roll back to previous version:
`git checkout [id] [file name]`

16. 回滚提交： `git checkout HEAD/[id]` # 最新一次提交的别名叫HEAD, 提交会被打回暂存区，重新提交

17. 解决合并冲突，冲突经常出现在合并分支或者去拉取别人代码。

18. 配置.gitignore, 在项目根目录创建, 列出不需要提交的文件名、文件夹名, 每个一行

19. 不指明的情况下，远程主机被Git命名为origin。 可通过`git remote -v` 查看。 

### Management
1. Installation
[git for windows](https://git-for-windows.github.io/)
[version 2.15.0](https://github.com/git-for-windows/git/releases/download/v2.15.0.windows.1/Git-2.15.0-64-bit.exe)
```
# double clicks and install with default settings
```

2. Setting and Example

```
# open git base
git config --global user.name "colinwxl"
git config --global user.email "colinabc@qq.com"
mkdir -p /d/git_project/python_function_core
cd /d/git_project/python_fucntion_core
git init
vim READMED.md # write sth. inside and save
git add README.md
git commit -m "create a README.md"

# create ssh public key
ssh-keygen -C "colinwuxl@outlook.com" -t rsa

# create an empty "python_function_core" repository in your git hub
# copy "/c/Users/colinwxl/.ssh/id_rsa.pub" 's content to your github ssh-key
ssh -vT git@github.com ## use "git" user instead of yours

git remote add origin git@github.com:colinwxl:python_function_core.git
git push origin master
```

3. Git GUI
http://www.cnblogs.com/iruxu/p/gitgui.html
