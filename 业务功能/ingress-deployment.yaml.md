```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: iot-api-ingress
  namespace: lida
  annotations:
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "DNT,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type"
    nginx.ingress.kubernetes.io/cors-allow-headers: "*"
    nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "8m"
spec:
  #  tls:
  #    - hosts:
  #        - server.yes-love.develop.meiliguiji.com
  #      secretName: server-develop-meiliguiji-tls-secret
  rules:
    - host: iot-api.lida.debug.hbaila.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: iot-api-service
                port:
                  number: 80
    - host: backend-api.lida.debug.hbaila.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: backend-api-service
                port:
                  number: 80
    - host: front-api.lida.debug.hbaila.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: front-api-service
                port:
                  number: 80
    - host: ws-gateway.lida.debug.hbaila.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: websocket-gateway-service
                port:
                  number: 80

```