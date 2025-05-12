# 一些git的用法 

让git 默认使用vim作为commit message编辑：

```bash
git config --global core.editor "vim"
```

Git 默认对非 ASCII 字符（如中文）进行转义，以避免终端兼容性问题。以下是解决方法：运行以下命令，关闭 Git 的路径转义功能：

```
git config --global core.quotepath false
```



## 打patch  

要将修改后的代码以patch文件的形式发送给其他人，可以按照以下步骤操作：

### 生成patch文件

首先，确保你在包含修改的代码库目录中。然后，你可以使用`git diff`命令生成一个patch文件。

```bash
git diff > changes.patch
cat changes.patch #生成patch文件后，最好检查一下文件内容，确保没有遗漏或者错误。你可以使用以下命令查看patch文件内容：
```

### 如何应用patch

可以使用以下命令将patch文件中的更改应用到代码库：

```bash
git apply changes.patch
```

通过以上步骤，你就可以成功地生成和发送一个包含代码修改的patch文件。

### 更正式一点的做法

正式的做法应该先将改好的代码入库然后有一份commit的备注，具体操作包括将修改后的代码提交到版本库，并附上详细的提交备注，然后生成patch文件。以下是详细步骤：

首先，确保你的代码库是干净的，即没有未提交的更改。如果有未提交的更改，首先将它们提交。

```bash
# 查看当前状态
git status
# 添加所有更改到暂存区
git add .
# 提交更改并附上提交备注
git commit -s 
```

生成patch文件的方法之一是使用`git format-patch`命令，它会生成一个或多个包含提交的patch文件。

```bash
# 假设你刚才的提交是HEAD，生成包含该提交的patch文件
git format-patch -1 HEAD
```

这个命令会在当前目录生成一个以提交编号开头的patch文件，例如`0001-简短而具体的提交备注.patch`。



## 提Merge Request的时候应该注意的事情  

anyway, 先看看code review指南。

改完代码记得使用checkpatch查查错误

```bash
./scripts/checkpatch.pl/ -f ****.c
```

如果搞抽象了，也可以回退一下

```bash
git reset (--hard --soft) commit hash
# 修改完之后，使用下面命令可以覆盖掉
git push -f origin 分支名  
```



## 查看配置的账户邮箱和用户名

```bash
git config --list
```



## 看git日志的技巧

正常来说git log就可以、、、不过也有一些有用的操作

```bash
git log -p -- ./path/to/directory #查询指定目录下的修改日志，-p意思是可以看具体的代码
```

```bash
git log --author=" " 查询某个作者的提交
```

git blame

## 关于revert时合并时遇到的冲突解决

### diff3样式

据说更清晰，idk...

```bash
git config merge.conflict style diff3
```

```markdown
<<<<<<< HEAD
这是当前分支的代码 (A)
||||||| common ancestor
这是两个分支的共同代码 (B)
=======
这是合并进来的分支的代码 (C)
>>>>>>> branch
```

核心操作是只复现变化：

1. A，B当中有重复的要删掉（两块全部）。
2. B，C 中一样的不要管。
3. C 中多出来的要加上。



### 查询某个特定文件修改者和时间

```bash
git blame +filename
```

强制清除所有未被跟踪的文件

```bash
git clean -fd
```



### git stash用法

存储多个缓存

当你执行 `git stash` 时，使用 `git stash save "描述信息"` 命令添加描述信息：

```bash
git stash save "修复Bug的缓存"
git stash save "新增功能的缓存"
```

使用以下命令可以查看当前所有的缓存：

```bash
git stash list
```

使用以下命令可以弹出特定的缓存：

```bash
git stash apply stash@{n}
```

如果希望应用并删除该缓存，可以用 `pop` 替代 `apply`：

```bash
git stash pop stash@{1}
```

删除特定缓存

###### 如果你不需要某个缓存了，可以单独删除它：

```bash
git stash drop stash@{n}
```

清空所有缓存，运行命令：

```bash
git stash clear
```



## GitHub 秘钥配置

### 1. 创建秘钥

```bash
ssh-keygen -t rsa -C "your_email@example.com"
```
- -f 设置密钥文件存储文件名字，按提示选择密钥保存路径（默认按回车即可，生成在 `~/.ssh/id_rsa`）。
- 可选：设置密钥密码（增强安全性，若无需密码直接回车跳过）。
- -t 指定秘钥类型，默认rsa，ed25519也可以
- -C 设置注释文字，邮箱主机名，随便写写

### 2. 将公钥添加到GitHub

- **macOS/Linux**：  
  
  ```bash
  cat ~/.ssh/id_rsa.pub
  ```
  全选输出内容并复制。
  
- **Windows**：  
  
  ```powershell
  type $env:USERPROFILE\.ssh\id_rsa.pub
  ```
  或手动用记事本打开 `C:\Users\你的用户名\.ssh\id_ed25519.pub` 文件并复制内容。
  
  然后执行以下步骤

1. 登录GitHub，点击右上角头像 → **Settings** → **SSH and GPG Keys**。
2. 点击 **New SSH Key**。
3. 填写标题（如“My Laptop”），将复制的公钥内容粘贴到 **Key** 框中。
4. 点击 **Add SSH Key** 保存。

---

### **3. 验证SSH连接**
在终端执行：
```bash
ssh -T git@github.com
```
- **成功提示**：  
  `Hi username! You've successfully authenticated...`  
  表示配置成功！
  
- **失败处理**：  
  - 检查公钥是否完整粘贴到GitHub。
  - 确保私钥路径正确，执行 `ssh-add ~/.ssh/id_rsa`（如设置了密码需输入）。
  - 修复文件权限（仅macOS/Linux）：
    ```bash
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/*
    ```

### 4. git 配置本地用户名与邮箱

```bash
git config --global user.email "llennwww@gmail.com"
git config --global user.name "Ran Sakura"
git config --list
```

开发者需要配置本地用户信息，为方便用户识别，遵守相同的配置规范。如上。

### 5. 其他事项

你可以更改 SSH 密钥的关联邮箱，也可以将同一 SSH 密钥用于多个平台（如 GitLab、Gitee），但需要根据需求权衡**便利性**和**安全性**。以下是具体操作指南：

#### **方法一：直接复用同一密钥**

- **优点**：方便，无需管理多密钥。
- **缺点**：若任一平台泄露私钥，所有关联平台均受影响。

**操作步骤**：
1. **复制现有公钥内容**：
   
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
2. **添加到其他平台**：
   - **GitLab**：`Settings` → `SSH Keys` → 粘贴公钥。
   - **Gitee（码云）**：`设置` → `SSH 公钥` → 粘贴公钥。

3. **验证连接**：
   - GitLab：
     ```bash
     ssh -T git@gitlab.com
     ```
     成功提示：`Welcome to GitLab, @username!`
   - Gitee：
     ```bash
     ssh -T git@gitee.com
     ```
     成功提示：`Hello username! You've successfully authenticated...`

#### 方法二：为不同平台使用不同密钥

**操作步骤**：

1. **生成新密钥**（例如专用于 GitLab）：
   ```bash
   ssh-keygen -t ed25519 -C "gitlab_email@example.com" -f ~/.ssh/id_ed25519_gitlab
   ```
2. **配置 SSH 多密钥管理**：
   创建或编辑 `~/.ssh/config` 文件，指定不同平台的密钥：
   ```config
   # GitHub
   Host github.com
     HostName github.com
     User git
     IdentityFile ~/.ssh/id_ed25519_github
   
   # GitLab
   Host gitlab.com
     HostName gitlab.com
     User git
     IdentityFile ~/.ssh/id_ed25519_gitlab
   
   # Gitee
   Host gitee.com
     HostName gitee.com
     User git
     IdentityFile ~/.ssh/id_ed25519_gitee
   ```
3. **添加公钥到对应平台**，并验证连接。



## 本地仓库初始化并推送到尚未创建的远程仓库

### 1：在本地初始化Git仓库

1. 进入你的项目目录：
   ```bash
   cd /path/to/your/project
   git init
   git add .
   git commit -m "Initial commit"
   ```

### 2：在代码平台（如GitHub）创建远程仓库

  GitHub 示例:

1. 登录GitHub，点击右上角 **+** → **New repository**。
2. 填写仓库名称（如 `my-project`），选择公开或私有。
3. **不要勾选** “Initialize this repository with a README”（本地已有代码）。
4. 点击 **Create repository**。

### 3. 关联本地仓库与远程仓库

1. 复制远程仓库的SSH或HTTPS地址（推荐SSH）：
   - GitHub：点击仓库页面的 **SSH** → 复制类似 `git@github.com:username/repo.git` 的地址。

2. 将远程仓库地址添加到本地仓库：
   ```bash
   git remote add origin git@github.com:username/repo.git
   ```
   - `origin` 是远程仓库的默认别名

### 4. 推送本地代码到远程仓库

1. 推送代码并设置上游分支（以 `main` 分支为例）：
   ```bash
   git push -u origin main       # 如果本地分支是 main
   # 或
   git push -u origin master     # 如果本地分支是 master
   ```
   - `-u` 参数将本地分支与远程分支关联，后续可直接用 `git push`。
