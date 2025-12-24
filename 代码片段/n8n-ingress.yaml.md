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
[n8nadmin@iz2ze4bh3v8s785bblkgsrz k8s-yaml]$ cat n8n-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: n8n-ingress
  namespace: n8n # 同样在 n8n 命名空间
spec:
  ingressClassName: nginx
  rules:
  - host: automation.hbaila.com # !!! 换成你自己的内部域名
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: n8n-service # 对应我们上面创建的 Service
            port:
              number: 80 # 对应 Service 的端口


```