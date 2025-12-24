在开始部署应用之前，我们先来快速理解Kubernetes的一些核心概念，它们是理解后续操作的基础。

### 2.1 Kubernetes核心概念速览

- **Pod (豆荚)：**
    
    > [!INFO] 概念与原理
    > 
    > - **概念：** 最小的、可部署的计算单元。一个Pod是一个或多个容器（Sidecar边车模式，让主应用容器保持纯粹，把一些辅助功能（如日志收集、监控、代理）剥离到另一个专门的容器里）的集合。它们共享网络命名空间（即共享IP地址和端口范围）和存储卷。
    >     
    > - **重点** 
    > 	- 通常一个Pod运行一个应用（比如PHP+Nginx容器）
    > 	- Pod是短暂的，随时可能被创建/销毁。**不要直接修改Pod！**
    > 	- 可以理解为 **临时运行的服务器**, 启动的时候设置好，运行的时候别管，挂了就重新启动一个。
    >     
    > - **生命周期：** Pod是短暂的，它们可以被创建、销毁、替换，但不应该被直接修改。
    >     
    
- **Deployment (部署)：**
    
    > [!INFO] 概念与原理
    > 
    > - **本质：** 一个控制器，用来管理 Pod 的副本数量、更新策略、版本等。
    >     
    > - **重点** 
    > 	- **定义你期望的应用状态** (例如：运行 3 个相同的 Pod，为了实现负载均衡和高可用);
    > 	- Deployment 自动创建、更新和替换 Pod，**实现滚动更新和回滚**。
    > 	- **核心作用：保证你的应用始终按期望运行!**
    >     
    
- **Service (服务)：**
    
    > [!INFO] 概念与原理
    > 
    > - **本质：** 提供一个稳定的IP地址和 DNS 名称，让其他应用或用户访问到你的 Pod。
    >     
    > - **重点** 
    > 	- Pod 的 IP 地址会变化，**Service 提供一个固定的访问入口**
    > 	- Service 自动将流量分发到后端 Pod
    > 	- **作用：让应用可以被稳定地访问!**
    >     
    > - **Service类型：**
    >     
    >     - **ClusterIP (默认)：** 只能在集群内部访问。
    >         
    >     - **NodePort：** 在每个K8s节点上开一个指定端口（nodePort），外部可以通过 NodeIP:NodePort 访问。不推荐，端口范围有限（30000-32767）。
    >         
    >     - **LoadBalancer：** 需要云平台支持（AWS，GCP，Azure）。云平台自动创建一个外部负载均衡器，并将流量转发到Service。推荐，简单方便。
    >         
    >     - **ExternalName：** 将Service映射到外部DNS名称，而不是集群内部的Pod。
    >         
    
- **Ingress (入口)：**
    
    > [!INFO] 概念与原理
    > 
    > - **概念：** 管理外部对集群内部服务的HTTP/HTTPS访问的API对象。它允许您定义路由规则，将不同域名或路径的流量转发到集群内部的不同Service。
    >     
    > - **重点** 
    > 	- **定义路由规则**，将不同域名或 URL 路径的流量转发到不同的 Service；
    > 	- **一个 Ingress 可以管理多个 Service 的外部访问**
    > 	- 通常需要一个 **Ingress Controller** (如 Nginx Ingress Controller) 来实现。
    >     
    
- **Namespace (命名空间)：**
    
    > [!INFO] 概念与原理
    > 
    > - **概念：** K8s中用于将集群资源划分为独立的、虚拟的分组。
    >     
    > - **为什么是Namespace？** 资源隔离和管理。在大型集群中，可以为不同的团队、项目或环境（开发、测试、生产）创建不同的命名空间，避免资源命名冲突，并限制访问权限。
    >     
    

### 2.2 部署Nginx应用

我们将通过一个Nginx示例来演示如何在MicroK8s中部署应用并从外部访问。

**操作步骤：**

1. **创建命名空间：**
    
    ```bash
    kubectl create namespace lida
    ```
    
    > [!INFO] 目的  
    > 将Nginx应用及其相关资源隔离在 lida 命名空间下，方便管理和清理，避免与默认命名空间或其他应用混淆。
    
2. **创建Nginx Deployment：**
    
    ```bash
    # 先生成YAML清单文件，便于理解和修改
    kubectl run nginx --image=nginx --port=80 -n lida --dry-run=client -oyaml > nginx-deployment.yaml
    
    # 编辑nginx-deployment.yaml，通常不需要额外修改，保持Pod和Deployment定义
    # 注意把配置文件里镜像后的hash值删了，太坑人了。
    # 例如：image: nginx@sha256:abcd...  应改为 image: nginx
    vim nginx-deployment.yaml
    
    # 应用Deployment
    kubectl apply -f nginx-deployment.yaml
    ```
    
    > [!INFO] 命令解释
    > 
    > - **kubectl run：** 这是一个快速创建Deployment或Pod的命令，-oyaml --dry-run=client 选项可以将命令执行的结果输出为YAML格式，而不是直接创建资源，这是学习K8s YAML的最佳实践。
    >     
    > - **--image=nginx：** 指定使用的容器镜像为官方Nginx镜像。
    >     
    > - **--port=80：** 告诉K8s这个Pod内部的应用监听80端口。
    >     
    > - **kubectl apply -f：** K8s中部署和更新资源的标准命令，通过提交YAML文件来声明资源的期望状态。
    >     
    
3. **检查Nginx Deployment和Pod状态：**
    
    
    ```bash
    kubectl get deployment -n lida
    kubectl get pods -n lida -o wide # 可以看到它在哪部分运行以及它的IP
    ```
    
4. **暴露Nginx服务（NodePort方式）：**  
    默认情况下，Nginx Pod只能在集群内部通过其Pod IP访问。为了从外部访问，我们需要创建Service。
    
    ```bash
    # 先生成Service的YAML清单文件
    kubectl expose deployment/nginx --type=NodePort --port=80 --target-port=80 -n lida --dry-run=client -oyaml > nginx-service.yaml
    
    # 编辑nginx-service.yaml，确保type为NodePort，并可以指定nodePort
    vim nginx-service.yaml
    ```
    
    **nginx-service.yaml 示例：**
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
      namespace: lida
    spec:
      type: NodePort # 确保Service类型为NodePort
      ports:
      - port: 80
        targetPort: 80
        nodePort: 30082 # 可选，如果不指定，K8s会自动分配一个30000-32767之间的端口
      selector:
        app: nginx
    ```
    
    ```bash
    # 应用Service
    kubectl apply -f nginx-service.yaml
    ```
    
    > [!INFO] 命令解释与原理
    > 
    > - **kubectl expose：** 快速创建Service的命令。
    >     
    > - **--type=NodePort：** 指定Service类型为NodePort，表示将在每个节点上暴露一个端口。
    >     
    > - **--port=80：** Service在集群内部监听的端口。
    >     
    > - **--target-port=80：** Pod内部容器监听的端口。
    >     
    > - **原理：** NodePort Service会在每个K8s节点上打开一个随机或指定的端口（在30000-32767范围内），所有发送到这个NodePort的流量都会被路由到Service的ClusterIP，然后由Service的负载均衡器将流量分发到后端Pod。
    >     
    
    > [!CAUTION] 您之前的错误提示分析
    > 
    > - **错误提示：** "The Service "nginx" is invalid: spec.ports[0].nodePort: Forbidden: may not be used when type is ClusterIP"
    >     
    > - **原因：** 您在尝试为 ClusterIP 类型的Service指定 nodePort。nodePort 只能用于 NodePort 或 LoadBalancer 类型的Service，因为 ClusterIP 服务只在集群内部可见，无需外部端口。
    >     
    > - **解决方法：** 正如您所做的，在Service YAML中明确指定 type: NodePort。
    >     
    
5. **验证Nginx服务：**
    
    ```bash
    kubectl get svc -n lida
    kubectl get ep -n lida # 检查Service的Endpoint，确保它指向正确的Pod
    kubectl get pods --show-labels -o wide # 查看Pod的label，Service的selector需要匹配这些label
    ifconfig # 获取虚拟机的IP地址，例如 192.168.64.12
    ```
    
    - 从宿主机浏览器访问：http://<VM_IP_Address>:<NodePort>/ (例如 http://192.168.64.12:30082/)，您应该能看到Nginx的欢迎页面。
        

### 2.3 使用Ingress暴露Nginx服务 (更推荐)

[[NodePort]] 简单方便，但在生产环境中不实用：端口范围有限、需要记住多个端口、无法实现域名路由。[[Ingress]] 是更好的选择。

**操作步骤：**

1. **确认Ingress Controller已安装：**
    
	```bash
    kubectl get pods -n ingress -l app.kubernetes.io/component=controller # 检查Ingress Controller Pod状态
    ```
    > [!NOTE] 在启用 microk8s enable ingress 时，Nginx Ingress Controller 就会被部署在 ingress 命名空间下。
    
2. **安装NGINX Ingress Controller (如果 microk8s enable ingress 未自动安装或需要更新)**
    
    ```bash
    microk8s.kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
    # 注意把配置文件里镜像后的hash值删了，太坑人了。（如果通过直接apply官方url的方式，有时需要手动编辑本地副本再apply）
    ```
    
3. **创建Nginx Ingress资源：**  
    创建一个 nginx-ingress.yaml 文件：
    
    ```yaml
    # nginx-ingress.yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: nginx-ingress
      namespace: lida
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: / # 如果路径需要重写，例如访问 /foo 实际访问 /
    spec:
      rules:
      - host: nginx.example.com # 示例域名，你需要解析这个域名到VM的IP
        http:
          paths:
          - path: / # 匹配所有路径
            pathType: Prefix # 或 Exact, ImplementationSpecific
            backend:
              service:
                name: nginx-service # 你的Nginx Service名称
                port:
                  number: 80        # Nginx Service的端口
    ```
    
    > [!INFO] 命令解释与原理
    > 
    > - apiVersion: networking.k8s.io/v1: Ingress资源的API版本。
    >     
    > - kind: Ingress: 声明这是一个Ingress资源。
    >     
    > - metadata.name: Ingress资源的名称。
    >     
    > - spec.rules: 定义路由规则。
    >     
    > - host: nginx.example.com: 外部请求的域名。
    >     
    > - path: /: 外部请求的路径。
    >     
    > - backend.service.name: 后端Service的名称。
    >     
    > - backend.service.port.number: 后端Service的端口。
    >     
    
    > [!SUCCESS] 优点
    > 
    > - **域名路由：** 可以根据不同的域名将请求转发到不同的Service。
    >     
    > - **路径路由：** 可以根据URL路径将请求转发到不同的Service（如 /api 到API服务，/web 到Web服务）。
    >     
    > - **SSL终止：** Ingress Controller可以处理HTTPS流量并将其解密为HTTP发送给后端Service。
    >     
    > - **统一入口：** 所有外部流量通过一个统一的入口点（Ingress Controller的IP和80/443端口）进入集群。
    >     
    
    > [!FAILURE] 缺点  
    > 增加了额外的组件（Ingress Controller），配置可能稍微复杂。
    
4. **应用Ingress：**
    
    ```bash
    kubectl apply -f nginx-ingress.yaml
    ```
    
5. **配置Hosts文件：**  
    因为 nginx.example.com 只是一个示例域名，您需要在宿主机的 /etc/hosts 文件中添加一条记录，将 nginx.example.com 解析到您的MicroK8s VM的IP地址：
    
    ```bash
    # 在宿主机上编辑 /etc/hosts (macOS/Linux) 或 C:\Windows\System32\drivers\etc\hosts (Windows)
    192.168.64.12 nginx.example.com # 替换为你的VM实际IP
    ```
    
6. **验证Nginx Ingress：**
    
    ```bash
    kubectl get ingress -n lida
    kubectl get svc -n ingress-nginx # 检查ingress-nginx-controller的服务状态，通常是NodePort或LoadBalancer
    ```
    从宿主机浏览器访问 http://nginx.example.com/，您应该能看到Nginx的欢迎页面。
    

### 2.4 ConfigMap 用于配置管理

[[ConfigMap]] 用于存储非敏感的配置数据，例如应用程序的配置、命令行参数或环境变量。

**操作步骤：**

1. **创建 php.ini 文件（示例）：**
    
    ```
    ; php.ini (示例文件)
    memory_limit = 256M
    upload_max_filesize = 100M
    post_max_size = 100M
    ```
    
    在VM中创建此文件，例如在 /home/ubuntu/php.ini。
    
2. **创建ConfigMap：**
    
    ```bash
    kubectl create configmap php-ini-config --from-file=php.ini -n lida
    ```
    
    > [!INFO] 目的与原理
    > 
    > - **配置与代码分离：** 将应用的配置从容器镜像中分离出来，使得配置的修改不需要重新构建镜像和部署。
    >     
    > - **集中管理：** K8s管理所有配置，方便统一更新和回滚。
    >     
    > - **多环境配置：** 可以为不同的环境（开发、测试、生产）使用不同的ConfigMap，实现配置的灵活切换。
    >     
    
3. **在Pod中使用ConfigMap：**  
    在您的Deployment YAML中，可以通过 volumeMounts 和 volumes 将ConfigMap作为文件挂载到Pod内部：
    
    ```yaml
    # 部分Deployment YAML示例
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: your-php-app
      namespace: lida
    spec:
      template:
        spec:
          containers:
          - name: php-fpm
            image: your-php-fpm-image
            volumeMounts:
            - name: php-config-volume
              mountPath: /etc/php/8.2/fpm/conf.d/ # 根据PHP-FPM配置文件的实际路径调整
              readOnly: true
          volumes:
          - name: php-config-volume
            configMap:
              name: php-ini-config # 引用你创建的ConfigMap名称
    ```
    
    > [!INFO] 原理  
    > ConfigMap挂载为卷时，K8s会将ConfigMap中的每个键值对作为文件和其内容创建在挂载路径下。
    
4. **查看和删除ConfigMap：**
    
    ```bash
    kubectl get configmap php-ini-config -n lida -o yaml # 查看ConfigMap内容
    kubectl delete configmap php-ini-config -n lida     # 删除ConfigMap
    ```