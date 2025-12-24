```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: websocket-gateway
  namespace: lida
spec:
  replicas: 1 # 先用一个副本进行测试
  selector:
    matchLabels:
      app: websocket-gateway
  template:
    metadata:
      labels:
        app: websocket-gateway
    spec:
      containers:
        - name: websocket-gateway
          image: hyperf/hyperf:7.4-alpine-v3.11-swoole
          tty: true
          imagePullPolicy: IfNotPresent #Always
          workingDir: /app
          ports:
            - containerPort: 9501
              name: http
            - containerPort: 9502 # WebSocket端口
              name: ws
          volumeMounts:
            - mountPath: /app
              name: websocket-gateway-app
            - mountPath: /etc/php7/php.ini
              name: php-ini-config-volume
              subPath: php.ini
          command: ["/bin/sh", "-c", "--"]
          args:
            - "/usr/bin/php /app/bin/hyperf.php server:watch;"
            #- "sleep 3600;" # <-- 替换成这一行，让容器启动后睡1小时
          env:
            - name: COMPOSER_ALLOW_SUPERUSER
              value: "1"
            # 重要：在这里注入POD_IP环境变量
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
      volumes:
        - name: websocket-gateway-app
          hostPath:
            path: /var/www/lida/websocket-gateway
        - name: php-ini-config-volume
          configMap:
            name: php-ini-config

---
apiVersion: v1
kind: Service
metadata:
  name: websocket-gateway-service
  namespace: lida
spec:
  selector:
    app: websocket-gateway
  ports:
    - name: http
      port: 80
      targetPort: 9501
    - name: ws # WebSocket端口的映射
      port: 9502
      targetPort: 9502
```