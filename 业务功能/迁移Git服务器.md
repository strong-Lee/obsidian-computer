## 场景

公司有一个git服务器用于开发，目前该服务器到期了（已经不能写了），需要将git迁移到另外一台能访问的主机上，保留当前的分支和注释。

### 第一步：在新阿里云服务器上搭建 Git 服务

#### 1. 安装 Git
```bash
# 使用 root 权限执行
sudo yum update -y
sudo yum install git -y
```

#### 2. 创建一个专门的 Git 用户
```bash
# 添加用户
sudo adduser git

# （可选）为 git 用户设置密码，但我们主要用 SSH 密钥，所以密码不是必须的
# sudo passwd git
```

#### 3. 配置 SSH 免密登录
```bash
# a. 在你的本地开发电脑上（不是服务器！），获取你的 SSH 公钥。
cat ~/.ssh/id_rsa.pub
# 如果这个文件不存在，说明你还没有生成过 SSH 密钥对。用以下命令生成一个
ssh-keygen -t rsa -b 4096
# b. 回到你的阿里云服务器上，进行以下操作
# 切换到 git 用户
sudo su - git

# 在 git 用户的家目录下创建 .ssh 文件夹和 authorized_keys 文件
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# 编辑 authorized_keys 文件, 存入本地电脑刚复制的公钥
vim ~/.ssh/authorized_keys
```

#### 4. 创建一个裸仓库 (Bare Repository)
```bash
# 确保你当前是 git 用户 (如果不是，执行 sudo su - git)
# 我们在 git 用户的家目录下创建一个仓库
# 仓库名最好和你的项目名一致，并以 .git 结尾，这是一个惯例
git init --bare business-middle-office.git
```

### 第二步：在你的本地电脑上操作

#### 1. 添加新的远程仓库地址
```bash
# 测试连接是否成功，一般阿里云服务器需要用到，因为白名单的问题
ssh -T git@8.140.250.2

# 格式: git remote add <远程仓库名> <SSH地址>
# SSH地址格式: git@<服务器IP>:<裸仓库的绝对路径>

# 注意！路径是 /home/git/business-middle-office.git，这是上一步创建仓库时显示的路径
git remote add aliyun git@8.140.250.2:/home/git/business-middle-office.git
```

#### 2. 验证远程连接
```bash
❯ git remote -v
aliyun  git@8.140.250.2:/home/git/business-middle-office.git (fetch)
aliyun  git@8.140.250.2:/home/git/business-middle-office.git (push)
origin  https://e.coding.net/g-kzzx5985/aila/business-middle-office.git/ (fetch)
origin  https://e.coding.net/g-kzzx5985/aila/business-middle-office.git/ (push)
```

### 第三步：推送你的重要分支

#### 1. 推送 develop 分支
```bash
git push aliyun develop
git push aliyun feature/mall
git push aliyun release/1.0.2

# 如果你想把所有分支都迁移过去，可以使用下面的命令
git push --all aliyun
git push --tags aliyun
```

### 第四步：后续清理和设置（推荐）

#### 1. 验证迁移是否成功
```bash
# 在一个空目录下执行
git clone git@8.140.250.2:/home/git/business-middle-office.git
cd business-middle-office
git branch -a # 查看是否所有分支都过来了
```

#### 2. 将新仓库设置为主仓库 origin
```bash
# 回到你原来的项目目录
# 1. 删除旧的 origin
git remote rm origin

# 2. 将 aliyun 重命名为 origin
git remote rename aliyun origin

# 现在你再执行 git remote -v，就只会看到新的 origin 了
```

#### 3. 设置上游分支 (Upstream)
```bash
# 切换到你的开发分支，比如 feature/mall
git checkout feature/mall

# 设置上游分支
git push -u origin feature/mall
```

### 第五步：修改 远程正式node1 上的项目远程地址

#### 1. 进入 node1 上的项目目录
```bash
# 在 node1 上执行
# cd /path/to/your/project/business-middle-office
cd /root/business-middle-office  # 假设路径是这个，请根据实际情况修改
```

#### 2. 修改远程仓库 origin 的 URL
```bash
# 使用新的 SSH 地址替换旧的 HTTPS 地址
git remote set-url origin git@8.140.250.2:/home/git/business-middle-office.git
```

#### 3. 验证修改是否成功
```bash
git remote -v
```

#### 4. 拉取最新的代码和分支信息
```bash
# 首先，从新的 origin 获取所有分支信息
git fetch origin

# 切换到你需要的正式站分支 release/1.0.2
# 因为本地没有这个分支，Git 会自动创建并跟踪远程同名分支
git checkout release/1.0.2

# 确认一下代码是最新的
git pull origin release/1.0.2
```