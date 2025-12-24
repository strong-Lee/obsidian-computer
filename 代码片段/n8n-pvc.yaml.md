```yaml
# n8n-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: n8n-data-pvc
  namespace: n8n
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: microk8s-hostpath # 使用 microk8s 的存储类
  resources:
    requests:
      storage: 5Gi # 和您 helm 命令里的size保持一致
```