```bash
# 步骤 1: 创建一个新的 Namespace (可选但推荐)
# 为了保持整洁，我们给 n8n 创建一个独立的命名空间。
kubectl create namespace n8n
# 步骤 2: 创建 n8n.yaml 配置文件
# 在你的home目录或合适的地方创建一个文件
touch n8n.yaml
# 这个文件包含了 Deployment 和 Service。我们一次性搞定。

# 在应用这个 yaml 前
# 请执行 sudo mkdir -p /data/n8n && sudo chmod 777 /data/n8n**

# 步骤 3: 应用配置
# 使用 kubectl 应用这个配置文件, 默认是英文版本没有中文
kubectl apply -f n8n.yaml
# 如果打算使用中文需要自己编译
# 3.1. 克隆官方仓库
git clone https://github.com/n8n-io/n8n.git
cd n8n
# 3.2. 切换到你想要的版本，因为上面安装的是1.101.1
git checkout n8n@1.101.1
# 3.3. 安装所有依赖，需要一些时间
pnpm install
# 3.4.1. 获取 zh-CN.json文件
# 方案A 从社区贡献中获取 [n8n 社区中文翻译文件 (zh-CN).json]
https://github.com/other-blowsnow/n8n-i18n-chinese/blob/main/languages/zh-CN.json
# 方案B 自己从头翻译
# 3.4.2. 放置语言包到正确位置, 在 n8n 源码根目录下执行
mkdir -p packages/frontend/@n8n/i18n/src/locales/
mv zh-CN.json packages/frontend/@n8n/i18n/src/locales/zh-CN.json
# 3.5.1. 运行构建脚本, 在 n8n 源码根目录下执行
# 这个过程会非常耗时，并占用较多 CPU 和内存
pnpm build:docker
# 成功后，这个镜像会被自动命名为 n8nio/n8n:local
# 3.5.2. 验证、标记并推送镜像
docker images | grep n8n
# 你应该能看到一个 n8n-local:dev，标签为 dev 的镜像
docker tag n8/local:dev my-dockerhub-user/n8n-zh:1.101.1
docker tag n8nio/n8n:local localhost:32000/my-n8n:latest
# 3.5.3. 使用镜像
# 方案A：标记并推送镜像，暂时先不用
docker login    # 登录到你的 Docker Hub
docker push my-dockerhub-user/n8n-zh:1.101.1
# 方案B：下载镜像到本地，同步到远程服务器
docker save n8n-local:dev -o n8n.local.dev.tar
scp n8n.local.dev.tar aila@192.168.1.19:/home/aila/k8s-images
# 远程服务器导入镜像
microk8s.ctr images import ./n8n.local.dev.tar
# 添加标签
microk8s ctr image tag docker.io/library/n8n-local:dev localhost:32000/n8n-zh:1.101.1
# 推送到本地仓库
microk8s ctr image push localhost:32000/n8n-zh:1.101.1 --plain-http

# 检查一下 Pod 是否成功运行：
kubectl get pods -n n8n
# 等待看到 n8n-deployment-xxxxx 的状态变成 Running
# 步骤 4: 创建 Ingress 规则
touch n8n-ingress.yaml
kubectl apply -f n8n-ingress.yaml
```

详细代码参考[[n8n.yaml]]
详细代码参考[[n8n-ingress.yaml]]
详细代码参考[[n8n-pvc.yaml]]


```bash
# firewalld防火墙屏蔽的问题
sudo systemctl stop firewalld  # 停止服务从浏览器访问看看是否能正常访问

# 第一步：启动 firewalld 服务
sudo systemctl start firewalld
# 第二步：现在 firewalld 正在运行，可以添加规则了
# 将所有以 'cali' 开头的接口加入 trusted zone
sudo firewall-cmd --permanent --zone=trusted --add-interface=cali+
# 同时，把 Calico 使用的 VXLAN 接口也加入，这样更稳妥
sudo firewall-cmd --permanent --zone=trusted --add-interface=vxlan.calico
# 第三步：重新加载防火墙规则，使刚才的永久性规则生效
sudo firewall-cmd --reload
# 第四步（验证）：查看当前的 active zones，确认接口是否已分配正确
sudo firewall-cmd --get-active-zones
```


### 总结与操作指南

**你的问题：“我应该怎么做？”**

**答案：继续按照我之前给你的“Helm + 本地镜像仓库”的详细步骤操作即可。所有命令都是正确的。**

1. **服务器准备**: 创建非 root 用户 n8nadmin 并给予 sudo 权限。(已完成)
    
2. **安装 MicroK8s**: (已完成)
    
3. **启用 MicroK8s 插件**: microk8s enable dns hostpath-storage registry helm3 ... (大部分已完成)
    
4. **导入你的中文镜像到 MicroK8s**:
    
    - scp 上传 n8n.local.dev.tar 文件到服务器。
        
    - microk8s ctr image import ... 导入镜像。
        
    - microk8s ctr image tag ... 给镜像打上 localhost:32000 的标签。
        
    - microk8s ctr image push ... 将镜像推送到 MicroK8s 自带的本地仓库。
        
5. **使用 Helm 安装 n8n (并指定你的中文镜像)**:
    
    - microk8s helm3 repo add ... 添加 n8n 仓库。
        
    - microk8s helm3 install n8n n8n/n8n --set image.repository='localhost:32000/n8n-zh' ... 执行安装。
        
6. **配置域名访问**:
    
    - 创建 n8n-ingress.yaml 文件。
        
    - microk8s kubectl apply -f n8n-ingress.yaml 应用 Ingress 配置。
        
7. **配置 DNS**: 在你的域名服务商那里添加 A 记录。
    

**你完全在正确的轨道上。** MicroK8s 就是 Kubernetes，请放心地使用所有标准的 microk8s kubectl 和 microk8s helm3 命令来完成部署。