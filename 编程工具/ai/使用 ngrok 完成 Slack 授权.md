#### 阶段一：在你的 n8n 服务器上准备 ngrok

1. **注册 ngrok 账户并获取 AuthToken:**
    
    - 在你的电脑上，打开浏览器访问 [https://dashboard.ngrok.com/signup](https://www.google.com/url?sa=E&q=https%3A%2F%2Fdashboard.ngrok.com%2Fsignup) 注册一个免费账户。
        
    - 登录后，在左侧菜单点击 "Your Authtoken"，你会看到你的认证令牌。复制它。
        
2. **在你的 n8n 服务器 (aila 服务器) 上安装 ngrok:**
```bash
# 下载 ngrok
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null && echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list && sudo apt update && sudo apt install ngrok
```

3. **配置 ngrok AuthToken:**
```bash
# 把 <YOUR_AUTHTOKEN> 替换成你刚才复制的令牌
ngrok config add-authtoken <YOUR_AUTHTOKEN>
```

#### 阶段二：启动 ngrok 隧道并更新 Slack 配置

1. **启动隧道：** 在你的局域网服务器上，运行以下命令。
    
    - **重要:** 我们需要告诉 ngrok，把流量转发到你 n8n 所在的 **K8s Ingress Controller 的端口**，而不是 n8n Pod 的 5678 端口。因为你的 Ingress Controller (Nginx) 才是处理 n8n.aila.com 这个域名的入口。通常这个端口是 80。

```bash
# 启动一个 HTTP 隧道，指向你的 Ingress Controller 的 80 端口
ngrok http 80 --host-header=n8n.aila.com
```

2. **获取临时的 HTTPS URL:**

```bash
Forwarding                    https://random-string-123.ngrok-free.app -> http://localhost:80
```

- **复制这个 https://...ngrok-free.app 的地址。** 这就是你 n8n 实例的临时公网身份证。

3. **更新 Slack App 的重定向 URL:**
    
    - 回到你之前在 [https://api.slack.com/apps](https://www.google.com/url?sa=E&q=https%3A%2F%2Fapi.slack.com%2Fapps) 创建的应用页面。
        
    - 进入左侧的 **"OAuth & Permissions"**。
        
    - 在 **"Redirect URLs"** 部分，点击 **"Add New Redirect URL"**。
        
    - 粘贴刚刚复制的 ngrok URL，并在结尾加上 /rest/oauth2-credential/callback。
        
    - 最终的地址看起来像这样：https://random-string-123.ngrok-free.app/rest/oauth2-credential/callback
        
    - **点击 "Add"，然后点击 "Save URLs"。**

#### 阶段三：使用 ngrok URL 更新 n8n 并完成授权

1. **临时修改 n8n 的 WEBHOOK_URL:**
    
    - 我们需要临时告诉 n8n，它现在的“公网身份”是 ngrok 的地址。
        
    - 编辑你的 n8n.yaml 文件：
	
```bash
nano n8n.yaml

# 找到 env 部分，**暂时**把 WEBHOOK_URL 的值改成你的 ngrok 地址
  - name: WEBHOOK_URL
    value: "https://random-string-123.ngrok-free.app/" 

kubectl apply -f n8n.yaml
```

2.  **重新创建 Slack 凭证：**
    
    - 现在，回到 n8n 界面，**删除之前创建失败的那个 Slack 凭证**，然后重新创建一个新的。
        
    - 在创建窗口，把从 Slack API 页面复制的 **Client ID** 和 **Client Secret** 粘贴进去。
        
    - 点击 **"Connect my account"**。
        
    - **这一次，因为 n8n 告诉 Slack 的重定向地址 (https://...ngrok...) 和 Slack 配置里的地址完全匹配，授权流程将会顺利完成！**
	
#### 阶段四：收尾工作 (重要！)

1. **关闭 ngrok:**
    
    - 在你的 aila 服务器上，回到运行 ngrok 的那个终端窗口，按 Ctrl + C 停止它。你的临时公网地址就失效了。
        
2. **把 n8n 的 WEBHOOK_URL 改回来：**
    
    - 再次编辑 n8n.yaml 文件。
        
    - 把 WEBHOOK_URL **改回你原来的局域网地址** http://n8n.aila.com/。
        
    - 再次 kubectl apply -f n8n.yaml 让配置生效。
        
3. **删除 Slack App 里的 ngrok 重定向 URL:**
    
    - 回到 Slack API 页面，把那个临时的 ngrok 重定向 URL 从配置中删除，保持干净。