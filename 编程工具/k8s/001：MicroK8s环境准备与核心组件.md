MicroK8s是一个轻量级、快速、功能齐全的Kubernetes发行版，适合开发、测试和边缘计算。它将所有K8s组件打包成一个简单的Snap包，极大地简化了安装和管理，是开发、测试和边缘计算的理想选择。

### 1.1 MicroK8s安装方式选择与建议

您可以选择直接在Mac上安装MicroK8s，在Multipass虚拟机中安装，或直接在物理Ubuntu服务器上安装。**推荐在虚拟机或物理服务器上安装**，它们更接近生产环境，且隔离性更好。

#### 方式一：直接在Mac电脑上安装MicroK8s (不推荐)

```bash
brew install ubuntu/microk8s/microk8s # 下载并安装 Multipass 的一个版本
microk8s install --channel=1.29       # 初始化环境
microk8s status --wait-ready          # 等待启动，可能要多等一会
```

#### 方式二：在Multipass虚拟机里安装MicroK8s (推荐用于开发/测试)

在公司局域网环境下，为了模拟生产环境的多节点K8s集群，我们通常会在虚拟机中运行K8s。Multipass是Ubuntu官方提供的一个轻量级虚拟机管理器，特别适合在本地快速创建和管理Ubuntu VM。

1. **在Mac上安装Multipass虚拟机管理器：**
    
    ```bash
    brew install multipass # multipass --version 可查看版本
    # multipass find      # 查看可用的Ubuntu镜像，默认安装最新的ubuntu
    ```
    
2. **创建MicroK8s主节点虚拟机：**
    
    ```bash
    # MicroK8s 的运行内存只有 540MB，建议系统至少具有 20G 磁盘空间和 4G 内存
    multipass launch -n microk8s-vm -c 4 -m 8GB -d 50GB
    ```
    
    > [!CAUTION] 注意事项  
    > - **资源分配：** 4核8G50G是一个比较均衡的配置，对于开发和测试环境足够支撑起多个应用和K8s自身的需求。如果资源太少，K8s组件可能无法正常启动或运行不稳定。对于初学，这个配置有助于您体验到K8s的完整功能。
    >     
    > - **未来扩展：** 预留足够的磁盘空间（50GB），以应对后续镜像、日志和应用数据的增长，避免因空间不足导致的问题。
    >			
    > - 如果您的宿主机资源有限，可以适当调低CPU和内存，但至少建议2核4GB，否则K8s运行可能会非常缓慢或频繁崩溃。
    
3. **管理Multipass虚拟机：**
    
    ```bash
    multipass list              # 查看所有虚拟机的状态
    multipass info microk8s-vm  # 查看指定虚拟机的详细信息
    multipass set microk8s-vm.memory=6G # 运行时无法修改，需先停止虚拟机
    multipass stop microk8s-vm  # 停止某个虚拟机
    multipass delete microk8s-vm # 删除某个虚拟机
    multipass purge             # 彻底清除已删除的虚拟机数据
    ```

4. **进入虚拟机Shell：**
    
    ```bash
    multipass shell microk8s-vm
    ```
    
#### 方式三：在物理Ubuntu服务器里安装MicroK8s (推荐用于生产/稳定测试)

在Ubuntu系统不使用虚拟机。这是一种更直接的部署方式，避免了虚拟机层的额外开销，更接近生产环境的裸金属部署。

1. **服务器硬件配置分析与建议：**  
    根据您提供的 lscpu、free -h、df -h、ip a 输出：
    
    - **CPU:** Intel(R) Core(TM) i5-4460 CPU @ 3.20GHz (4核4线程)。
        
        > [!TIP] 建议  
        > 这是家用级或入门级工作站CPU。对于开发和测试环境足够，可以同时运行多个您的Hyperf应用、MySQL和Redis。但在高并发生产场景下，可能需要更多的核心或更强的单核性能。
        
    - **内存:** 15GiB总内存，当前已用4.3GiB，可用10GiB。
        
        > [!TIP] 建议  
        > 15GiB内存对于MicroK8s和您的几个应用来说是**相当充足的**。K8s核心组件通常占用1-2GB。剩余的约13-14GB可以高效地分配给您的应用Pods。这意味着您可以在部署时给Pod分配相对宽松的资源请求与限制，以保证性能。
        
    - **磁盘:** /dev/mapper/ubuntu--vg-ubuntu--lv 根分区54GB，已用36GB (71%)，剩余15GB。
        
        > [!CAUTION] 注意  
        > 您的主分区 (/) 已经使用了71%，只剩下15GB可用空间。
        > 
        > - **持久化存储：** 您在MySQL的[[PersistentVolume|PV]]中指定了 /var/lib/microk8s/data/mysql 路径，这将占用 / 分区空间。如果您有大量日志、Docker镜像、或者MySQL数据量增长，这15GB很快会用完。
        >     
        > - **临时存储：** Pod的[[#7.1 常见问题排查与解决|ephemeral-storage]] 也占用此分区。
        >     
        > - **建议：**
        > - 1. **清理空间**：sudo apt autoremove, sudo journalctl --vacuum-time=3d, 清理不再使用的镜像和文件。
		> - 2. **监控**：使用df -h和du -sh *持续监控。
        > 		
		> - 3. **最佳实践**：**强烈建议**将/var/snap/microk8s/common或/var/lib/containerd挂载到一块独立的大容量硬盘上，以隔离K8s数据并提供更好的I/O性能。>     
        > >         
        
    - **网络:** enp2s0 网卡IP为 192.168.1.19/24。docker0 和 br-adb575512c62 是Docker和桥接网络，vxlan.calico 是K8s Calico网络插件的接口。
        
        > [!TIP] 建议  
        > 192.168.1.19 是您的服务器局域网IP，这是一个静态IP地址，非常适合作为K8s节点的IP地址。确保这个IP地址在您的局域网中是固定分配的，不会变动。
        
2. **配置SSH登录服务器 (推荐)：**  
    为了方便管理，您应该配置从您的Mac Pro笔记本到Ubuntu服务器的SSH连接。
    
    - **方法一：允许Root密码登录 (不推荐用于生产)**

        ```bash
        # 在Ubuntu服务器内执行：
        sudo passwd                 # 修改root密码
        su root                     # 以root身份登录
        vim /etc/ssh/sshd_config    # 修改SSH配置文件
        # 找到并修改或添加以下两行：
        PermitRootLogin yes       # 允许root用户SSH登录
        PasswordAuthentication yes # 允许密码认证
        # PubkeyAuthentication yes   # 保持开启，支持密钥认证
        sudo service sshd restart   # 重启sshd服务
        # 或者 sudo systemctl restart ssh
        ```
        
    - **方法二：使用SSH密钥对登录 (强烈推荐)**  
        
		```bash
		# 1. 在本地电脑（宿主机，即您的Mac Pro）生成SSH密钥对
		ssh-keygen -t rsa -b 2048 # 按照提示操作，可选择不设置密码
		# 密钥默认生成在 ~/.ssh/id_rsa (私钥) 和 ~/.ssh/id_rsa.pub (公钥)。
		
		# 2. 将公钥复制到Ubuntu服务器
		ssh-copy-id -i ~/.ssh/id_rsa.pub aila@192.168.1.19 
		# 如果 ssh-copy-id 失败，可以手动复制：
		# 在Mac Pro上执行
		cat ~/.ssh/id_rsa.pub | ssh aila@192.168.1.19 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
		
		# 3. 尝试SSH登录
		ssh aila@192.168.1.19
	    ```

### 1.2 MicroK8s安装与核心配置 (在Ubuntu服务器或虚拟机内执行)

接下来的步骤无论您是在Multipass虚拟机内还是物理Ubuntu服务器内安装MicroK8s，都是相同的。

1. **更新系统和安装常用工具：**
    
    ```bash
    # 可选，更换为国内镜像源以加速
    # sudo sed -i -e "s/cn.archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/" /etc/apt/sources.list
	
    # sudo sed -i -e "s/security.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/" /etc/apt/sources.list
	
    sudo apt update && sudo apt -y upgrade
    sudo apt install net-tools
    sudo reboot	# 重启系统，让更新生效
	# 登录你的 root 账号
	# 创建一个新用户，例如叫 'n8nadmin'
	useradd n8nadmin
	
	# 为新用户设置密码，系统会提示你输入两次
	passwd n8nadmin
	
	# 给予新用户 sudo 权限，这样它可以执行需要 root 权限的命令
	usermod -aG wheel n8nadmin 
	# 在 CentOS/Alibaba Linux 中，'wheel' 组的用户默认有 sudo 权限
	
	# 切换到新用户
	su - n8nadmin
    ```
    
2. **安装MicroK8s：**
    
    ```bash
	sudo yum install -y snapd   # 安装snapd
	sudo systemctl enable --now snapd.socket
	sudo ln -s /var/lib/snapd/snap /snap  # 创建符号链接
	snap version  # 命令正常后再进行安装
    # 官网：https://microk8s.io/docs/release-notes  
    # snap info microk8s  // 查看snap提供了哪些版本
    # sudo snap refresh microk8s --channel=1.29/stable --classic 
    sudo snap install microk8s --classic # 默认是最新的版本，目前只能控制大版本
    # microk8s.version 	// 查看目前安装的版本
    # sudo snap remove microk8s // 卸载microk8s，不推荐频繁使用，可以使用reset
    ```
    
    > [!FAILURE] 缺点  
    > Snap的自动更新机制可能导致意外的版本升级，从而引入兼容性问题。这将在后面禁用。
    
3. **配置环境**
    
    ```bash
	# 1. 为非root用户授权, 以后就不用加sudo了（aila是您的用户名）
    sudo usermod -a -G microk8s aila 
    sudo mkdir -p ~/.kube
    sudo chown -R aila ~/.kube       # 确保当前用户对.kube目录有读写权限
	# 重新登录或执行 newgrp 使权限生效
    newgrp microk8s                  
    sudo passwd aila                 # 设置新密码为aila (如果需要)

	# 2. 创建别名，方便使用标准kubectl命令
    sudo snap alias microk8s.kubectl kubectl
    sudo snap alias microk8s.helm helm
    sudo snap alias microk8s.ctr ctr
    sudo snap alias microk8s.helm3 helm3
    # 也可以在用户home目录下的 .bashrc 或 .zshrc 中添加 alias kubectl="microk8s kubectl"
	
	# 3. 检查状态，确保所有组件正常启动
	# 阿里云服务器需要特殊处理
	microk8s inspect  # 查看报错原因
	sudo hostnamectl set-hostname iz2ze4bh3v8s785bblkgsrz 
	# 不能包含大写字母， 退出重新登录
	sudo /snap/bin/microk8s reset # 重置Microk8s
	sudo /snap/bin/microk8s start # 启动服务

	# 下面的镜像如果服务器不能自动下载可以手动上传并安装
	calico.cni.v3.28.1.tar
	calico.kube-controllers.v3.28.1.tar
	calico.node.v3.28.1.tar
	registry.k8s.io.pause.3.10.tar
	# 导入镜像
	microk8s.ctr images import  ./calico.kube-controllers.v3.28.1.tar 
	
	microk8s status --wait-ready  # 等待集群就绪
	kubectl get nodes # 节点状态必须是Ready
    kubectl get pod --all-namespaces  # 查看所有命名空间下的Pod运行状态
	# 查看pod节点未启动的原因
    kubectl describe -n kube-system pod calico-node-pd5lx 
    kubectl describe node aila   # 查看节点错误原因 (您的服务器名称是 'aila')
    ```
    
    > [!INFO] 目的与原理
    > 
    > - **安全性：** 我们通常不建议使用root用户直接操作K8s。将普通用户（如 aila）添加到 microk8s 用户组后，该用户将拥有执行 kubectl 等MicroK8s命令的权限，而无需每次都使用 sudo，这符合最小权限原则。
    >     
    > - **权限管理：** MicroK8s通过Linux用户组来管理对K8s API的访问权限。~/.kube 目录通常存放Kubeconfig文件，chown 确保当前用户对该目录有读写权限。
    > - **别名**：让您使用kubectl而非microk8s.kubectl，与社区标准保持一致，提高效率。

### 1.3 启用核心插件 (Addons)与配置

MicroK8s通过插件（Addons）的形式提供额外的功能，这些插件是K8s生态系统中常用的工具和组件。

1. **禁用Snap自动更新（非常重要！）：**
    
    ```bash
    sudo snap refresh --hold microk8s
    ```
    
2. **设置防火墙规则：**
    
    ```bash
    sudo iptables -P FORWARD ACCEPT
    ```
    
    > [!INFO] 目的与原理
    > 
    > - **网络转发：** Kubernetes Pod之间的通信以及Pod与外部网络的通信，很多情况下需要Linux内核进行IP数据包转发。iptables -P FORWARD ACCEPT 将转发链的默认策略设置为接受，确保流量可以正常转发。
    >     
    > - **原理：** iptables 是Linux的防火墙工具，用于配置网络包过滤规则。K8s的网络插件（如Calico、Flannel）依赖正确的网络转发设置来建立Pod网络。
    >     
    
    > [!CAUTION] 注意事项  
    > 在生产环境中，通常会配置更细粒度的 iptables 规则或使用专门的防火墙管理工具（如 ufw），避免过于开放的策略。但对于MicroK8s的开发测试环境，这是常见且有效的设置。
    
3. **启用核心插件：**
    
    ```bash
    microk8s enable dns hostpath-storage dashboard ingress metrics-server rbac registry
    ```
    
    - **dns：** [[CoreDNS]]
        
        > - **目的/优点：** 部署Kubernetes DNS服务（CoreDNS）。允许Pod之间通过服务名互相发现和通信，而不是通过不稳定的IP地址。例如，一个Web应用可以通过 mysql-service 来访问数据库，而不是 10.1.2.3。
        >     
        > - **原理：** CoreDNS会监听K8s API，当有新的Service被创建时，CoreDNS会自动为其创建DNS记录。Pod内部的DNS解析请求会被劫持并发送到CoreDNS。
        >     
        
    - **hostpath-storage：** [[HostPath Storage]]
        
        > - **目的/优点：** 创建一个默认的 StorageClass，使用 hostpath-provisioner。为需要持久化存储的Pod提供存储卷。在单节点MicroK8s中，它允许将Pod的数据存储在宿主机的文件系统上，实现简单的数据持久化。
        >     
        > - **缺点：** **不适用于生产环境！** HostPath 存储意味着数据与特定节点绑定。如果该节点故障，数据将丢失且无法被其他节点上的Pod访问。不具备高可用性。
        >     
        > - **替代方案（高级）：** 在生产环境中，应使用支持网络存储的 StorageClass，如NFS、Ceph (Rook)、GlusterFS、云服务商的PV (EBS, GPD等)。
        >     
        
    - **dashboard：** [[Kubernetes Dashboard]]
        
        > - **目的/优点：** 部署Kubernetes Web UI 仪表板。提供一个可视化的界面来管理和监控集群中的应用、资源和事件，方便初学者快速了解集群状态。
        >     
        > - **注意事项：** 访问Dashboard通常需要通过 kubectl proxy 或 ingress 配置。
        >     
        
    - **ingress：** [[Nginx Ingress Controller]]
        
        > - **目的/优点：** 部署Nginx Ingress Controller。提供HTTP/HTTPS路由功能，将外部流量根据域名或路径转发到集群内部的不同服务。相比于 [[#2.2 暴露Nginx服务（NodePort方式）|NodePort]]，Ingress 提供了更灵活、更高级的外部访问方式，例如SSL终止、虚拟主机、基于路径的路由等。
        >     
        > - **原理：** Ingress Controller 实际上是一个运行在集群内部的Pod，它会监听K8s Ingress资源的创建和更新，并根据这些规则动态配置其代理（如Nginx或Traefik）来转发流量。
        >     
        
    - **metrics-server：** [[Metrics Server]]
        
        > - **目的/优点：** 部署Metrics Server。收集集群中Pod和节点的资源使用数据（CPU、内存）。这些数据是 kubectl top 命令和 [[#水平 Pod 自动伸缩 (Horizontal Pod Autoscaler, HPA)|Horizontal Pod Autoscaler (HPA)]] 功能的基础。
        >     
        > - **原理：** Metrics Server 从 Kubelet（运行在每个节点上的代理）获取资源数据，并将其通过API暴露给其他组件。
        >     
        
    - **rbac：** [[RBAC (基于角色的访问控制)]]
        
        > - **目的/优点：** 启用基于角色的访问控制。允许您为不同的用户或服务账户定义精细的权限，控制他们能对K8s资源执行什么操作。这对于多用户、多团队环境下的安全至关重要。
        >     
        > - **原理：** RBAC通过定义 Role (或 ClusterRole) 和 RoleBinding (或 ClusterRoleBinding) 来将权限绑定到用户或服务账户上。
        >     
        
    - **registry：** [[Docker Private Registry]] (可选，高级)
        
        > - **目的/优点：** 部署一个私有的Docker镜像仓库在 localhost:32000。允许您在集群内部存储和管理自己的Docker镜像，无需依赖外部公共镜像仓库。在没有外部网络或需要严格控制镜像来源的场景下非常有用。
        >     
        > - **原理：** 它会在集群内部启动一个Docker Registry容器，并暴露一个NodePort服务，使得集群内外的客户端都可以访问。hostpath-storage 插件将作为此插件的一部分启用，用于持久化registry的数据。
        >     
        
    - **gpu / istio：**
        
        > - 您原文档中提及的这两个插件，在此处暂不启用，因为它们属于更高级的用例。
        > 
        > - gpu: 暴露GPU资源给K8s Pod，用于AI/ML工作负载。需要NVIDIA驱动。
        >     
        
        > - istio: 服务网格，提供流量管理、安全、可观测性等高级功能。学习曲线较陡峭。
            
    > [!CAUTION] 注意事项  
    > 启用插件后，可能需要等待一段时间，相关Pod才能完全启动并就绪。可以使用 microk8s status 或 kubectl get pods -n kube-system 查看。
    
4. **配置Kubeconfig文件（可选，但推荐）：**
    
    ```bash
	# 生成配置文件，以便外部工具（如IDE插件、Lens）连接集群
    microk8s config > ~/.kube/config
    chmod 600 ~/.kube/config # 确保只有所有者有读写权限，增强安全性
    ```
    
5. **查看MicroK8s状态和插件列表：**
    
    
    ```bash
    microk8s status
    # 这时你会看到提供了helm3、metrics-server，其实没有毛变化
    helm list # 查看Helm chart列表
    ```
    
    
6. **配置Containerd调试日志 (可选)：**
    
    ```bash
    echo '-l=debug' | sudo tee -a /var/snap/microk8s/current/args/containerd # debug级日志添加到containerd
    microk8s stop && microk8s start
    ```
    

### 1.4 镜像管理 (Containerd)与内网/离线部署

MicroK8s默认使用Containerd作为其容器运行时。在内网或无法访问gcr.io的环境中，镜像管理是成功的关键。K8s节点无法直接拉取镜像时，Pod会处于ImagePullBackOff状态。解决方案是：在有外网的机器上下载镜像，打包，然后传输到K8s节点上手动导入。

 **方式一：使用Docker下载并导出/导入缺失镜像 (推荐在有外网的跳板机或Mac Pro上操作)**
        
1. **安装Docker (在有外网的机器上)：** 如果没有安装Docker，请先安装。
            
2. **配置Docker镜像加速器 (可选，但推荐)：**  
		编辑 /etc/docker/daemon.json (Linux) 或 Docker Desktop 设置：

	```json
	{
		"exec-opts": ["native.cgroupdriver=systemd"],
		"registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com"]
	}
	 ```
            
	重启Docker服务。
            
 3. **下载缺失的K8s核心镜像：**  
	**完整的镜像列表可以参考这里：** https://github.com/canonical/microk8s/blob/1.22/build-scripts/images.txt (请注意版本号可能不同)
            
	```bash
    docker pull registry.k8s.io/pause:3.7
	# 根据实际需求调整版本
    docker pull registry.k8s.io/metrics-server/metrics-server:v0.6.3 
    docker pull registry.k8s.io/ingress-nginx/controller:v1.8.0
    # ... 其他缺失的镜像，如calico相关
	```
            
4. **导出镜像为tar包：**
            
	```bash
	docker save -o k8s_core_images.tar \
		registry.k8s.io/pause:3.7 \
		registry.k8s.io/metrics-server/metrics-server:v0.6.3 \
		registry.k8s.io/ingress-nginx/controller:v1.8.0
	```
            
5. **将tar包复制到MicroK8s服务器或虚拟机：**
            
	```bash
    # 从Mac Pro将文件复制到Ubuntu服务器/虚拟机
    scp /path/to/k8s_core_images.tar aila@192.168.1.19:/home/aila/
    # 如果是Multipass虚拟机，也可以使用
    # multipass transfer k8s_core_images.tar microk8s-vm:~/
    ```
            
6. **在MicroK8s服务器/虚拟机中导入镜像：**
            
	```bash
	# 导入镜像tar包到microk8s的Containerd
	microk8s.ctr images import ./k8s_core_images.tar 
	# 如果是其他docker load 的tar包，也可以用
	# docker load < /root/images.tar  # 导入镜像tar包到docker
	```
            
            
 **方式二：直接使用MicroK8s拉取/标记缺失镜像 (适用于可访问外部镜像源但Google源被墙)**
        
```bash
# 具体的版本号根据错误的提示进行拉取，不要直接复制，例如 `microk8s status` 或 `kubectl describe pod` 报错中提示的镜像版本。
# 示例：拉取pause镜像, 使用第三方镜像源
microk8s.ctr images pull docker.io/ghostwritten/registry.k8s.io.pause:3.7 # 标记为K8s需要的官方名称
microk8s.ctr images tag docker.io/ghostwritten/registry.k8s.io.pause:3.7 registry.k8s.io/pause:3.7
        
# 示例：拉取Calico相关镜像
microk8s.ctr images pull docker.io/calico/cni:v3.25.1 # 示例版本
microk8s.ctr images pull docker.io/calico/kube-controllers:v3.25.1
microk8s.ctr images pull docker.io/calico/node:v3.25.1
microk8s.ctr images pull docker.io/calico/pod2daemon-flexvol:v3.25.1
        
# 示例：拉取Ingress-Nginx Controller
microk8s.ctr images pull docker.io/giantswarm/ingress-nginx-controller:v1.8.0
microk8s.ctr image tag docker.io/giantswarm/ingress-nginx-controller:v1.8.0 registry.k8s.io/ingress-nginx/controller:v1.8.0
        
# 示例：拉取Metrics Server
microk8s.ctr images pull docker.io/bitnami/metrics-server:0.6.3 
microk8s.ctr image tag docker.io/bitnami/metrics-server:0.6.3 registry.k8s.io/metrics-server/metrics-server:v0.6.3
        
# 示例：拉取默认的hostpath-provisioner
microk8s.ctr images pull docker.io/cdkbot/hostpath-provisioner:1.1.0
        
# 示例：拉取CoreDNS
microk8s.ctr images pull docker.io/coredns/coredns:1.8.0 
        
# 示例：如果还需要其他K8s核心组件，例如更老的pause版本
microk8s.ctr images pull docker.io/rancher/pause:3.1
microk8s.ctr image tag docker.io/rancher/pause:3.1 k8s.gcr.io/pause:3.1
```
        
        
7. **MicroK8s内部镜像导出和导入 (集群内节点间传输)**
    
```bash
# 在已下载好镜像的MicroK8s节点上导出
microk8s.images export-local microk8s_images.tar # 导出当前MicroK8s的所有镜像
    
# 将tar文件从已安装好的MicroK8s节点复制到新的MicroK8s节点
scp /path/to/microk8s_images.tar aila@<new_node_ip>:/home/aila/
    
# 在新的MicroK8s节点上导入
microk8s.images import microk8s_images.tar
```
    
   > [!INFO] 目的与原理
   > 
   > - **目的：** 这种方式最适合在已经有一个 MicroK8s 节点成功拉取了所有必要镜像后，将这些镜像快速分发到集群中的其他 MicroK8s 节点，避免重复下载和网络问题。
   >     
   > - **原理：** microk8s.images export-local 会将当前 MicroK8s 的 Containerd 运行时中的所有镜像打包成一个 .tar 文件。microk8s.images import 则会将这些镜像加载回另一个 MicroK8s 实例的 Containerd 中。