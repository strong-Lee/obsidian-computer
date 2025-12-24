```yaml
# ------------------- Deployment ------------------- #
apiVersion: apps/v1
kind: Deployment
metadata:
  name: n8n-deployment
  namespace: n8n # 部署在 n8n 命名空间
  labels:
    app: n8n
spec:
  replicas: 1 # 先跑一个实例就够了
  selector:
    matchLabels:
      app: n8n
  template:
    metadata:
      labels:
        app: n8n
    spec:
      containers:
      - name: n8n
        image: localhost:32000/n8n-zh:1.101.1 # 必须使用这个在本地仓库中的完整地址
        imagePullPolicy: IfNotPresent # <--- 添加这一行！
        ports:
        - containerPort: 5678 # n8n 默认监听的端口
        env:
        # 设置时区，对定时任务至关重要
        - name: TZ
          value: "Asia/Shanghai"
        - name: N8N_DEFAULT_LOCALE
          value: "zh-CN"
        # 告诉 n8n 它将通过哪个域名被访问，这会影响 Webhook URL 的生成
        - name: WEBHOOK_URL
          value: "http://automation.hbaila.com/" # 稍后在 Ingress 中我们会用这个域名
          #value: "https://fdf5f11800ed.ngrok-free.app/" # !!! 临时换成你的 ngrok 地址 !!!
        - name: N8N_SECURE_COOKIE # 禁用安全Cookie
          value: "false"
        volumeMounts:
        - name: n8n-data
          mountPath: /home/n8nadmin/.n8n # 将数据目录挂载出来
      volumes:
      - name: n8n-data
        persistentVolumeClaim:
          claimName: n8n-data-pvc # 指向我们上面创建的 PV
        # 使用 hostPath 将数据持久化到宿主机的一个明确路径
        #hostPath:
        #  path: /data/n8n # !!! 重要：请确保你的服务器上有 /data/n8n 这个目录
        #  type: DirectoryOrCreate

---

# ------------------- Service ------------------- #
apiVersion: v1
kind: Service
metadata:
  name: n8n-service
  namespace: n8n # 同样在 n8n 命名空间
spec:
  type: ClusterIP # 只在集群内部暴露，给 Ingress 用
  selector:
    app: n8n # 选择所有打了 app=n8n 标签的 Pod
  ports:
  - protocol: TCP
    port: 80 # Service 的端口
    targetPort: 5678 # 转发到 Pod 的 5678 端口
```