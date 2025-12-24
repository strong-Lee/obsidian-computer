#### **第一步：环境准备（在你的 MacBook Pro 上）**

1. **安装 DevSpace CLI：**  
    打开你的 Mac 终端，运行：
    
    ```bash
    brew install devspace
	```

2. **配置 kubectl 连接到你的 MicroK8s 集群（这是最关键的一步）：**  
	你需要在你的 Mac 上能够运行 kubectl get pods -n lida 并看到结果。

- 通过 SSH 登录到你的 Linux 开发服务器：ssh aila@192.168.1.19
    
- 在服务器上运行 microk8s config 命令。这会打印出 kubeconfig 文件的内容。
    
- **将这些内容完整地复制下来**。
    
- 回到你 Mac 的终端，编辑 ~/.kube/config 文件（如果文件或目录不存在，请创建它：mkdir -p ~/.kube && touch ~/.kube/config）。将刚才复制的内容粘贴进去。
    
- **【极其重要】** 在你粘贴到 Mac 上的 config 内容中，找到 server: 这一行。它的地址很可能是 https://127.0.0.1:16443。你必须**将 127.0.0.1 修改为你的 Linux 服务器的局域网 IP 地址 192.168.1.19**。修改后应如下所示：

	```bash
		clusters:
		- cluster:
		    certificate-authority-data: ...
		    server: https://192.168.1.19:16443 # 确认这里是服务器的IP
		  name: microk8s-cluste
	```
- 保存文件。然后在你的 Mac 终端上运行 kubectl get ns 进行测试。如果你能看到 lida 这个 namespace，**恭喜你，最关键的一步配置成功！**

#### **第二步：改造 business-middle-office 项目**

我们以 business-middle-office 为例。你需要进入**你本地 Mac 上的项目目录**：  
cd /Users/cay/www/aila/business-middle-office

1. **修改 Kubernetes YAML 文件**
    
    为了规范，建议在项目根目录下创建一个 k8s 文件夹，并把你的 YAML 文件放进去，命名为 deployment.yaml。  
    mkdir -p k8s && vim k8s/deployment.yaml
    
    **对 k8s/deployment.yaml 进行如下修改（请仔细对比，我已标出修改点）：**
    
    Generated yaml
    
    ```
    # 文件路径: /Users/cay/www/aila/business-middle-office/k8s/deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: business-middle-office-service
      namespace: lida
    spec:
      selector:
        matchLabels:
          app: micro-business-middle-office-service
      template:
        metadata:
          labels:
            app: micro-business-middle-office-service
        spec:
          containers:
            - name: micro-business-middle-office-service
              image: hyperf/hyperf:7.4-alpine-v3.11-swoole
              tty: true
              imagePullPolicy: IfNotPresent
              workingDir: /app
              ports:
                - containerPort: 9501
              # 【修改点 1】: 将启动命令修改为 "sleep"，DevSpace 会在开发时覆盖它
              # 这样能保证容器先"活"下来，等待DevSpace注入代码和开发命令
              command: ["sleep"]
              args: ["infinity"]
              # 【修改点 2】: 删除 volumeMounts 和 volumes，DevSpace会处理同步
              env:
                - name: COMPOSER_ALLOW_SUPERUSER
                  value: "1"
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: business-middle-office-service
      namespace: lida
    spec:
      selector:
        app: micro-business-middle-office-service
      ports:
        - port: 80
          targetPort: 9501
    ```
    
    content_copydownload
    
    Use code [with caution](https://support.google.com/legal/answer/13505487).Yaml
    
    **修改解释:**
    
    - **command: ["sleep"], args: ["infinity"]**: 我们让容器启动后什么都不做，只是“活着”。因为 DevSpace 会通过 kubectl exec 注入真正的开发启动命令。这使得同一个镜像可以用于开发（DevSpace 注入命令）和生产（使用镜像默认命令）。
        
    - **删除 volumeMounts 和 volumes**: 我们不再使用 hostPath。代码将由 DevSpace 动态同步进去，这让我们的部署配置与服务器物理路径解耦，更加灵活和规范。
        
2. **创建 devspace.yaml 文件**
    
    在项目根目录 (/Users/cay/www/aila/business-middle-office) 创建 devspace.yaml 文件：  
    vim devspace.yaml
    
    **写入以下内容：**
    
    Generated yaml
    
    ```
    # 文件路径: /Users/cay/www/aila/business-middle-office/devspace.yaml
    version: v2beta1
    name: middle-office
    
    # 定义如何部署应用
    deployments:
      - name: app
        # 指向我们修改过的 kubectl manifest
        kubectl:
          manifests:
            - k8s/deployment.yaml
    
    # 定义核心的开发模式
    dev:
      # 1. 端口转发、日志和命令注入
      - name: app
        namespace: lida
        labelSelector:
          app: micro-business-middle-office-service
        # 将容器的9501端口转发到你本地Mac的9091端口
        # 你可以通过 http://localhost:9091 访问
        ports:
          - port: "9501:9091"
        # 自动在容器内执行开发命令 (这会替换掉 "sleep infinity")
        # 并且将日志输出到你的终端
        command:
          command: php bin/hyperf.php server:watch
    
      # 2. 自动文件同步 (替换 rsync)
      - name: sync-code
        namespace: lida
        labelSelector:
          app: micro-business-middle-office-service
        # 将本地当前目录 (.) 同步到容器的 /app 目录
        sync:
          - path: .:/app
        # 从同步中排除的目录/文件 (来自你的rsync脚本)
        exclude:
          - .git/
          - runtime/
          - vendor/
          - .idea/
          - composer.json
          - composer.lock
          - "*.php~"
          - nohup.out
          - .DS_Store
    ```
    
    content_copydownload
    
    Use code [with caution](https://support.google.com/legal/answer/13505487).Yaml
    
3. **启动开发会话！**
    
    现在，你只需要在你的项目目录 /Users/cay/www/aila/business-middle-office 下，运行一个命令：
    
    Generated bash
    
    ```
    devspace dev
    ```
    
    content_copydownload
    
    Use code [with caution](https://support.google.com/legal/answer/13505487).Bash
    
    **DevSpace 会自动完成以下所有事情：**
    
    - 使用 k8s/deployment.yaml 在你的 MicroK8s 集群中部署或更新应用。
        
    - 等待 Pod 启动并进入 Running 状态。
        
    - **启动高效的文件同步**，将你的本地代码首次同步到容器的 /app 目录。
        
    - **启动端口转发**，将 localhost:9091 映射到容器的 9501 端口。
        
    - **在容器内执行 php bin/hyperf.php server:watch**，并将其日志实时输出到你的 Mac 终端。
        
    
    现在，你可以在本地的编辑器里修改任何 PHP 代码，文件会**毫秒级**同步到容器中，然后 Hyperf 的 watch 机制会检测到变化并自动重载 Worker，你在 Postman 中刷新接口（访问 http://localhost:9091/你的路由）就能立刻看到效果。
    
4. **结束开发**  
    在终端里按 Ctrl + C，DevSpace 会自动清理端口转发等所有它建立的连接。
    

---

#### **第三步：为 front-api 和其他项目配置**

这个过程是完全可以复制的。对于 front-api 项目：

1. 进入本地目录 cd /Users/cay/www/aila/front-api。
    
2. 同样，创建一个 k8s/deployment.yaml，并根据上面的例子修改它（主要是 command 和删除 hostPath volume）。**注意，front-api 的 YAML 有一个额外的 ConfigMap volume，你需要保留它，这完全没问题。**
    
3. 在 front-api 根目录创建一个 devspace.yaml，内容和 business-middle-office 的几乎一样，只需要修改 name，labelSelector，以及本地转发端口（避免冲突，比如用 9092）。
    
4. 在你需要同时开发多个服务时，只需为每个服务打开一个新的终端 Tab，进入其项目目录，然后运行 devspace dev。
    

---

### 总结你的新工作流

1. **一次性设置**：为每个项目配置好 k8s/deployment.yaml 和 devspace.yaml。
    
2. **日常开发**：
    
    - 打开一个终端，cd 到 business-middle-office 目录，运行 devspace dev。
        
    - 打开另一个终端，cd 到 front-api 目录，运行 devspace dev。
        
    - ...
        
    - 在你的 Mac 上愉快地编码，所有同步和重载全部自动完成。
        
    - 彻底告别手动 rsync 脚本和 kubectl exec / delete pod。
        

这套方案将让你的开发体验提升一个档次，完全拥抱云原生的开发模式。祝你编码愉快！