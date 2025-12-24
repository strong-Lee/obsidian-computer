```
# 1. 申请持久化存储 (PersistentVolumeClaim)
# ------------------------------------------
# 它会向你的MicroK8s存储系统申请一块空间。MicroK8s默认开启了hostpath-storage。
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pvc
  namespace: lida
spec:
  accessModes:
    - ReadWriteOnce # 这个卷只能被一个Pod挂载
  resources:
    requests:
      storage: 1Gi # 申请1GB的存储空间，用于测试足够了

---
# 2. Redis部署 (Deployment)
# -------------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: lida
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:6-alpine # 使用一个轻量的官方Redis镜像
          imagePullPolicy: IfNotPresent
          command:
            - redis-server
            - "--appendonly yes" # 开启AOF持久化
            # 如果需要密码，请取消下面一行的注释，并替换为你自己的密码
            # - "--requirepass YourStrongPassword"
          ports:
            - containerPort: 6379
              name: redis
          volumeMounts:
            - name: redis-data
              mountPath: /data # Redis的数据目录
      volumes:
        - name: redis-data
          persistentVolumeClaim:
            claimName: redis-pvc # 挂载我们上面申请的PVC

---
# 3. Redis服务 (Service)
# ---------------------
# 为Redis Pod创建一个集群内部可以访问的稳定域名
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: lida
spec:
  selector:
    app: redis # 选择所有标签为 app: redis 的Pod
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
```