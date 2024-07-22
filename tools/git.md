### 创建分支
```
git branch branch-name
```

### 切换分支
```
git checkout branch-name
```

### 创建并切换至分支
```
git checkout -b branch-name
```

### 本地分支建立与远程分支关联
```
git checkout -b branch-name origin/branch-name
git branch --track branch-name origin/branch-name
git branch -u origin/branch-name(当前分支与指定远程分支关联)
```
### 查看本地分支与远程分支的跟踪关系
```
git branch -vv
```
------
### 初始化远程仓库
```
git init
git add .
git commit -m ''
git branch -m master main
git push -u origin main
```

### 初始化git远程仓库
```
初始化git仓库
git init

添加代码到本地暂存库
git add .

提交代码
git commit -m ''

设置远程仓库地址
git remote add origin {远程仓库地址}

校验远程仓库
git remote -v

出现不允许拉取代码时使用
git pull origin main --allow-unrelated-histories

切换至main分支
git checkout -b main

将代码推至远程仓库
git push -u origin main
```

### 从某个提交创建分支
```
git checkout -b newBranch commitId
```
