---
title: "#Git# 速查手册：常用 Git 命令"
date: 2020-5-14 07:00:21
categories: Quick Ref Guide
tags:
- Git
thumbnail: /images/git.png
---





覆盖在学习和工作中的**常用 Git 命令**：配置 / 仓库 / 查看 / 暂存 / 提交 / 回退 / 克隆 / 拉取 / 推送 / 合并 / 冲突 / 分支 / 标签 / 工具

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---

## **配置**

### 账号

```bash
# 配置多个本地用户的账号

# 全局级
git config --global user.name "${user_name}"
git config --global user.email "${user_email}"

# 仓库级
# git config --global --unset user.name "${user_name}"
# git config --global --unset user.name "${user_email}"
git config --local user.name "${user_name}"
git config --local user.email "${user_email}"
```

### 密钥

```bash
# 配置多个远程仓库的密钥

# 生成密钥
ssh-keygen -t rsa_github
ssh-keygen -t rsa_gitlab

# 配置密钥
vim ~/.ssh/config

Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/rsa_github
 
Host gitlab.com
HostName gitlab.com
User git
IdentityFile ~/.ssh/rsa_gitlab

:wq
```

## **仓库**

### 查看

```bash
git remote -v
```

### 新建

```bash
# Example: git remote add origin git@github.com:LotteWong/lottewong.github.io.git
git remote add ${origin_name} git@host:/path/to/registry.git
```

### 删除

```bash
git remote rm ${origin_name}
```

## **查询**

### 当前状态

```bash
git status
```

### 差异分析

#### 工作区 vs 暂存区

```bash
git diff (${branch_name}) (${file}) # 默认 branch = current; file 若不填作用于全部文件
```

#### 暂存区 vs 版本库

```bash
git diff --cached (${commit_id}) (${file}) # 默认 commit = HEAD; file 若不填作用于全部文件
```

#### 工作区 vs 版本库

```bash
git diff ${commit_id} (${file}) # file 若不填作用于全部文件
```

#### 版本库 vs 版本库

```bash
git diff ${commit_id} ${commit_id} (${file}) # file 若不填作用于全部文件
```

### 提交记录

```bash
git log # 详细
git log --oneline # 简略
git log --graph # 图示
```

### 操作记录

```bash
git reflog
```

## **暂存**

### 查看

```bash
git stash list
```

### 保存

```bash
git stash
```

### 恢复

```bash
git stash pop # 默认还原栈顶数据
git stash pop stash@{${id}} # 指定还原特定数据
```

### 删除

```bash
git stash clear # 删除全部暂存数据
git stash drop stash@{${id}} # 删除指定暂存数据
```

## **提交**

### 暂存区

```bash
git add -A or git add --all # 提交整个仓库全部文件(New/Modified/Deleted)
git add /path/to/file # 提交指定目录全部文件(New/Modified/Deleted)
git add -u # 提交整个仓库已有文件(Modified/Deleted)
```

### 版本库

```bash
git commit -m "${commit_message}" # 指定提交信息标题
git commit # 打开默认的编辑器
git commit --amend # 追加提交
```

## **回退**

### 工作区

```bash
git reset --hard (${commit_id}) (${file}) # 默认 commit = HEAD; file 若不填作用于全部文件
# git checkout ${commit_id} ${file}
```

### 暂存区

```bash
git reset (--mixed) (${commit_id}) (${file}) # 会清除暂存区; 默认 commit = HEAD; file 若不填作用于全部文件
# git checkout ${commit_id} ${file}
```

### 版本库

```bash
# 向前移动指针
git reset --soft (${commit_id}) (${file}) # 不清除暂存区; 默认 commit = HEAD; file 若不填作用于全部文件
# git checkout ${commit_id} ${file}

# 向后移动指针
git revert (${commit_id}) # 默认 commit = HEAD; 撤销单次提交并生成单次提交
git revert -n ${old_commit}^..${new_commit} # 撤销连续提交并生成单次提交
```

## **克隆**

```bash
# Example: git clone git@github.com:LotteWong/lottewong.github.io.git
git clone git@host:/path/to/registry.git # 克隆默认分支

# Example: git clone -b backup git@github.com:LotteWong/lottewong.github.io.git
git clone -b ${branch_name} git@host:/path/to/registry.git # 克隆指定分支
```

## **拉取**

```bash
git pull origin ${remote_branch} # 拉取到已有本地分支
git checkout -b ${local_branch}:${remote_branch} # 拉取并新建本地分支

git fetch origin ${remote_branch}
git pull origin ${remote_branch} # git fetch + git merge
git pull origin --rebase ${remote_branch} # git fetch + git rebase
```

## **推送**

```bash
git push origin ${remote_branch} # 推送到已有远程分支
git push origin ${local_branch}:${remote_branch} # 推送并新建远程分支 
```

## **合并**

### Merge

```bash
git merge ${branch_name} # 带有单独合并提交信息
git merge ${branch_name} --no-commit # 不带单独合并提交信息
git merge --continue # 解决冲突后使用
git merge --abort # 不打算解决冲突
```

### Squash

```bash
git merge --squash ${branch_name} # A1 → B1 → M3
git rebase -i ${startpoint} (${endpoint}) # A1 → B1 → C2 → D2; 默认 endpoint = HEAD
# pick(p)：保留本次提交
# squash(s)：合并前后提交
```

### Rebase

```bash
git rebase ${branch_name}
git rebase --continue # 解决冲突后使用
git rebase --abort # 不打算解决冲突
```

### Cherry-Pick

```bash
git cherry-pick ${commit_id}
git cherry-pick --continue # 解决冲突后使用
git cherry-pick --abort # 不打算解决冲突
```

## **冲突**

```bash
git status # 查看冲突文件
vim /path/to/file # 解决冲突之处
git add /path/to/file # 将文件加入暂存区
git commit -m "${commit_message}" # 将文件加入版本库
```

## **分支**

### 查看

``` bash
git branch -a # 全部分支
git branch -v # 本地分支
git branch -r # 远程分支
```

### 更新

```bash
git remote update
```

### 新建

```bash
git checkout -b ${local_branch}:${remote_branch} # 创建本地分支：远程分支 → 本地分支
git push origin ${local_branch}:${remote_branch} # 创建远程分支：本地分支 → 远程分支
```

### 切换

```bash
git checkout ${branch_name}
```

### 删除

```bash
git branch -d ${local_branch} # 非强制删除本地分支
git branch -D ${local_branch} # 强制性删除本地分支
git push origin --delete ${remote_branch} # 删除远程分支
```

## **标签**

### 查看

```bash
git tag # 全部标签
git show ${tag_name} # 指定标签
```

### 新建

```bash
git tag ${tag_name} (${commit_id}) # 创建本地标签; 默认 commit = HEAD
git push origin --tags # 创建全部远程标签
git push origin ${tag_name} # 创建指定远程标签
```

### 删除

```bash
git tag -d ${tag_name} # 删除本地标签
git push origin --delete ${tag_name}
```

## **工具**

### 提交规范

```bash
# 安装
npm install -g commitizen cz-conventional-changelog

# 使用
git cz
```

### 变更日志

```bash
# 安装
npm install -g conventional-changelog-cli

# 使用
conventional-changelog -p angular -i CHANGELOG.md -s -r 0
```

### 代码评审

- *To be continued...*

### 子级模块

- *To be continued...*

---