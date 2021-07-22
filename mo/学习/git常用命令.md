### 拉取远程分支

git checkout -b xxx(本地分支名) origin/xxx(远程分支名)  创建本地分子并关联远程

git pull origin xxx(远程分支)

### 重新关联分支

git branch --set-upstream local_branch_name origin/remote_branch_name

git push --set-upstream origin mohai-online-1.15 

本地有原代码，但是没有与远程git做关联，或关联不正确，需要重新跟踪远程git:

**直接修改;**

git remote origin set-url URL

**删除在关联**

git remote rm origin  #删除

git remote add origin https://xxx.git 关联

git push -u origin yourbranch //推送