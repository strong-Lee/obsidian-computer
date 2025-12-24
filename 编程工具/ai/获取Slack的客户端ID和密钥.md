这个过程分为三步：**从n8n复制信息 -> 在Slack创建App -> 把信息粘贴回n8n**。请严格按照步骤来。

#### 阶段一：在 n8n 中获取“重定向URL”

这是第一步，我们需要先从 n8n 这里拿到一个关键信息。

1. 回到 n8n 的 Slack 凭证创建窗口 (就是你看到要填客户端ID的那个窗口)。
    
2. 找到一个叫做 **OAuth Redirect URL** (或类似的) 的字段。
    
3. **复制 (Copy)** 这个 URL。它通常是 https://<你的n8n域名>/rest/oauth2-credential/callback。这个 URL 非常重要，是 Slack 完成授权后“跳回” n8n 的地址。
    

#### 阶段二：在 Slack API 网站上创建应用

现在，打开你的电脑浏览器，我们去 Slack 的“安保中心”。

1. **访问 Slack API 网站：** 打开 [https://api.slack.com/apps](https://www.google.com/url?sa=E&q=https%3A%2F%2Fapi.slack.com%2Fapps)
    
2. **创建新应用：** 点击绿色的 **"Create New App"** 按钮。
    
3. **选择创建方式：** 在弹出的窗口中，选择 **"From scratch"** (从头开始)。
    
    - **App Name:** 给你的应用起个名字，比如 n8n-Aila-Bot，这样以后你在 Slack 里就知道是哪个应用在操作。
        
    - **Workspace:** 选择你想要 n8n 连接的那个 Slack 工作区。
        
    - 点击 **"Create App"**。
        
4. **找到密钥，但先别急着复制：** 创建成功后，你会进入应用的“Basic Information”页面。向下滚动，就能看到 **App Credentials** 部分，这里就有我们要的 **Client ID** 和 **Client Secret**。**先让这个页面开着，我们还有关键步骤没做。**
    
5. **配置 OAuth 和权限 (最关键的一步！):**
    
    - 在左侧菜单栏，找到并点击 **"OAuth & Permissions"**。
        
    - 向下滚动到 **"Redirect URLs"** 部分。
        
    - 点击 **"Add New Redirect URL"**。
        
    - **粘贴** 我们在【阶段一】从 n8n 复制的那个 URL。
        
    - 点击 **"Add"**，然后点击 **"Save URLs"**。
        
6. **授予权限 (Scopes):**
    
    - 继续在 "OAuth & Permissions" 页面向下滚动，找到 **"Scopes"** 部分。
        
    - Scopes 就是你具体要给 n8n 这个机器人哪些权限。我们需要给它“发消息”的权限。
        
    - 在 **"Bot Token Scopes"** 下，点击 **"Add an OAuth Scope"**。
        
    - 我们需要添加以下几个常用权限 (一个一个添加)：
        
        - chat:write: 允许它在频道里发消息。
            
        - channels:read: 允许它读取频道的列表（这样你在n8n里才能看到频道下拉菜单）。
            
        - groups:read: 允许它读取私有频道的列表。
            
        - users:read: 允许它读取用户列表。
            
        - users:read.email: 允许它读取用户的邮箱地址。
            
7. **安装应用到工作区：**
    
    - 配置好权限后，滚动到页面顶部，点击 **"Install to Workspace"** 按钮。
        
    - Slack 会让你确认授权，点击 **"Allow"**。
        

#### 阶段三：将密钥复制回 n8n

1. 安装成功后，回到我们第4步打开的那个 **"Basic Information"** 页面。
    
2. 现在，正式地**复制 Client ID**，然后切换回 n8n 窗口，把它**粘贴**到“客户端ID”的输入框里。
    
3. 同样，回到 Slack 页面，点击 **"Show"** 按钮显示 **Client Secret**，**复制**它，然后**粘贴**到 n8n 的“客户端密钥”输入框里。
    
4. 在 n8n 窗口中，点击 **"Save"** 或 **"Connect my account"**。
    

这时，n8n 就会使用你提供的ID和密钥，跳转到 Slack 进行最终的授权确认。你点击允许后，凭证就创建成功了！