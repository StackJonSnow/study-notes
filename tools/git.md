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
