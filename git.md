



## ✅ 推荐方式：先 stash 一下，等更新完再恢复

如果你希望保险一点，推荐你先暂存一下本地的修改：

```
git stash
git pull origin main
git stash pop
```

- #### 如果暂时不想 commit，可以用 **stash**

  如果你还不想 commit，但是又要切换分支，可以用：

  ```
  git stash
  git checkout wklbranch
  ```

  等你切回 `add_face_scheduler` 分支之后，恢复：

  ```
  git stash pop
  ```
  
  stash 就像是一个临时存储空间。👍

### ✅**所有的stash操作:**

```
git stash             # 保存当前修改（默认 message）
git stash push -m "备注"    # 保存并写备注
git stash save "备注"       # 老写法，兼容
git stash list
git stash apply stash@{0}        # 应用 stash，保留记录
git stash apply                  # 应用最新 stash，保留记录
git stash pop stash@{0}          # 应用 stash，并删除记录
git stash pop                    # 应用最新 stash，并删除记录
git stash drop stash@{0}         # 删除指定 stash
git stash clear                  # 删除所有 stash
```





## ⚠️ 小提醒：push

如果你本地的分支是新建的，并且远程还没有对应分支，**首次 push** 可能需要加上：

```
git push -u origin 你的分支名
```

这个 `-u` 的作用是「建立追踪关系」，以后你就可以直接用：

```
git push
```

不用再带分支名了。

1. `git push origin 分支名`

👉 **意思**：
 把本地的某个分支，推送到远程的 `origin` 仓库。
 这个是**单纯的一次性推送**，下一次你还得手动指定分支名。

比如：

```
git push origin wklbranch
```

如果你后续还要推送，还是得写上：

```
git push origin wklbranch
```

------

2. `git push -u origin 分支名`

👉 **意思**：
 推送代码 **并且建立本地分支和远程分支的关联关系**。
 用 `-u` （或者全写是 `--set-upstream`），意思是：

> “我这个本地分支以后默认推送到这个远程分支，不用每次都写名字。”

比如：

```
git push -u origin wklbranch
```

这样之后你只需要写：

```
git push
```

Git 会自动知道：哦，你是要把本地的 `wklbranch` 推送到远程的 `wklbranch`。

3. 





## ⚠️ 小提醒：git pull = git fetch + git merge

`git pull = git fetch + git merge`）

## ✅小总结：branch

- `git branch`：查看本地分支。
- `git branch -r`：查看远程分支。
- `git branch -a`：查看所有分支（本地 + 远程）。



## ⚠️创建远程分支

| `git checkout -b add_modbus`                   | 创建本地分支 `add_modbus`              | 普通创建本地分支，和远程没有关系           |
| ---------------------------------------------- | -------------------------------------- | ------------------------------------------ |
| `git checkout -b add_modbus origin/add_modbus` | 创建并跟踪远程分支 `origin/add_modbus` | ✅ **推荐用这个！用于拉远程分支到本地开发** |



## ⚠️远程分支拉到本地

```
git fetch origin
git checkout -b biyi_processor_model origin/biyi_processor_model
```

**切换到 master 分支：**

```
git checkout master
git pull origin master
```



## ⚠️**先更新本地分支，再 push 到远程**

```
git add .
git commit -m "本地改动：描述一下"
git pull origin gaotie_docker
```

​     在本地解决冲突....

```
git push origin 你的分支名
```



## ⚠️重命名本地分支

```
git branch -m Cmodbus modbus
```

# -----------------------

## 🚀 **Git 技巧**

## stash

1.  `git stash list` 输出

**应用并删除**（弹出）

```bash
# 或弹出 stash@{1}（描述为 "pytest"）
git stash pop stash@{1}
```

**仅应用不删除**（保留储藏项）

```bash
git stash apply stash@{0}
```

**删除储藏项但不应用**：

```bash
git stash drop stash@{0}
```

**创建新分支并应用储藏**（适合长期保存）：

```bash
git stash branch new-feature stash@{0}
```

# -----------------------

## 🚀 **Git 开发流程总结**



### 1. 初始化项目

```
git remote add origin <仓库地址>
```

### 2. 克隆远程仓库

```
bash


复制编辑
git clone <仓库地址>
```

------

## 🌿 **分支管理**

### 查看所有分支（本地 + 远程）

```
git branch -a
```

### 创建本地分支并关联远程分支

```
git fetch origin
git checkout -b 本地分支名 origin/远程分支名
```

### 创建新分支（本地新分支）

```
git checkout -b 新分支名
```

### 切换分支

```
git checkout 分支名
```

### 删除本地分支

```
git branch -d 分支名
```

------

## 📥 **拉取更新**

### 拉取远程分支更新到本地

```
git pull origin 分支名
```

### 拉取所有远程更新，不合并

```
git fetch origin
```

------

## 📝 **提交操作**

### 查看当前状态

```
git status
```

### 添加到暂存区

```
git add .         # 添加所有修改
git add <文件名>  # 添加指定文件
```

### 撤销 add（移出暂存区）

```
git reset HEAD <文件名>
```

### 提交到本地仓库

```
git commit -m "提交说明"
```

### 修改最近一次 commit 说明

```
git commit --amend -m "新的说明"
```

------

## 🚀 **推送代码**

### 推送分支到远程

```
git push origin 分支名
```

### 设置默认推送分支（第一次推送建议加）

```
git push -u origin 分支名
```

------

## 🔀 **合并与拉取请求（PR）**

### 合并分支到当前分支

```
git merge 其他分支名
```

### 解决冲突后继续

```
git add .
git commit -m "解决冲突"
```

```
# 获取最新主分支
git checkout main
git pull origin main

# 切换到你的 feature 分支
git checkout your-feature-branch

# 与 main 合并，解决冲突
git merge main
# 解决冲突文件（编辑后，删除 <<<<, >>>>, ====）
# 然后继续：
git add .
git commit

# 推送解决完冲突的分支
git push origin your-feature-branch
```



### 拉取请求（PR）流程：

1. 本地创建分支，提交代码。
2. 推送到远程：`git push origin 新分支`
3. 在 GitHub / GitLab 上发起 Pull Request，选择目标分支。
4. 代码评审、合并。



# -----------------------

# Rebase

### 更整洁：

1. 切换回工作分支：

   ```
   git checkout clean_queue
   ```

2. git log --oneline

3. git rebase -i HEAD~4

   把你想合并的提交改为 `squash` 或 `s`，第一个必须是`pick`

   ```
   pick 76477f8 put 4 in queue if empty
   squash c8a3588 change_processor
   squash 86cf032 change_conf
   squash 5322675 update something
   ```

4. 修改commit

5. 解决冲突git status  -->  vim  --> git add  -->  git commit  --> git rebase --continue

6. 

   ```
   git push -f origin xxx
   ```

   > 注意：使用 `-f` 是因为 rebase 更改了历史，会强推。



### git push 先 git pull master

```
git pull --rebase origin master
git add <冲突文件>
git rebase --continue
git rebase --abort

git add .
git commit -m "说明修改内容"

git push origin HEAD:new_branch
HEAD:当前分支
git push origin feature_branch:feature_branch
```



### rebase 最近基础commit

```
git rabase -i HEAD~n
```

$ git rebase --abort

$ git rebase --continue

# -----------------------

# commit

常见的 subject 种类以及含义如下：

feat: 新功能（feature）

用于提交新功能。
例如：feat: 增加用户注册功能
fix: 修复 bug

用于提交 bug 修复。
例如：fix: 修复登录页面崩溃的问题
docs: 文档变更

用于提交仅文档相关的修改。
例如：docs: 更新README文件
style: 代码风格变动（不影响代码逻辑）

用于提交仅格式化、标点符号、空白等不影响代码运行的变更。
例如：style: 删除多余的空行
refactor: 代码重构（既不是新增功能也不是修复bug的代码更改）

用于提交代码重构。
例如：refactor: 重构用户验证逻辑
perf: 性能优化

用于提交提升性能的代码修改。
例如：perf: 优化图片加载速度
test: 添加或修改测试

用于提交测试相关的内容。
例如：test: 增加用户模块的单元测试
chore: 杂项（构建过程或辅助工具的变动）

用于提交构建过程、辅助工具等相关的内容修改。
例如：chore: 更新依赖库
build: 构建系统或外部依赖项的变更

用于提交影响构建系统的更改。
例如：build: 升级webpack到版本5
ci: 持续集成配置的变更

用于提交CI配置文件和脚本的修改。
例如：ci: 修改GitHub Actions配置文件
revert: 回滚

用于提交回滚之前的提交。
例如：revert: 回滚feat: 增加用户注册功能

# -----------------------



# 失误

👉 **Git 检测到 `.git/index.lock` 文件存在**，以为另一个 Git 进程还在运行。

但实际上：

- 可能是之前的 Git 命令被中断/崩溃
- 或者确实已经没别的 Git 进程

> | 类型 `[!...]`  | 图标 / 含义 | 用途说明               |
> | -------------- | ----------- | ---------------------- |
> | `[!note]`      | 📝 备注      | 一般信息提示           |
> | `[!tip]`       | 💡 提示      | 建议或技巧             |
> | `[!info]`      | ℹ️ 信息      | 重要信息提醒           |
> | `[!warning]`   | ⚠️ 警告      | 有风险的操作提示       |
> | `[!danger]`    | 🚨 危险      | 高风险、不可逆操作提示 |
> | `[!important]` | ❗ 重要      | 强调需要特别注意的内容 |
> | `[!quote]`     | 🗨️ 引用      | 强调引用语句           |
> | `[!example]`   | 📌 示例      | 显示示例代码或案例     |

> [!note] 
> 这是一个备注信息。
>
> 

> [!tip] 
> 💡 小贴士：可以使用快捷键 `Ctrl + P` 快速搜索。
>
> 

> [!info] 
> ℹ️ 你可以在设置中启用高级模式。

> [!warning] 
> ⚠️ 修改配置文件前请备份原始文件！
>
> 

> [!danger] 
>
> 🚨 此操作将清除所有用户数据，**无法恢复**！

> [!important] 
> ❗ 部署前请确保配置环境变量！
>
> 

> [!example] 
> 示例：这是一个示例块，可以放代码、场景说明等。

