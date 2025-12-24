	这是最激动人心的部分！我们将配置 Neovim 连接到本地运行的 AI 模型，实现代码生成、解释和编辑，完全免费且隐私安全。

### 步骤 1：安装并运行 Ollama (你的本地 AI 服务器)

1. **从官网下载并安装**:
    
    - 访问 [https://ollama.com/](https://www.google.com/url?sa=E&q=https%3A%2F%2Follama.com%2F)
        
    - 点击 "Download for macOS"。
        
    - 解压下载的 .zip 文件，将 Ollama.app 拖到你的“应用程序”(Applications)文件夹。
        
2. **运行 Ollama 应用**:
    
    - 从“应用程序”文件夹中，找到并双击 Ollama.app 来启动它。
        
    - **关键验证**: 启动后，你的 Mac 屏幕顶部的菜单栏应该会出现一个小小的 **Llama 头像图标**。这表示 Ollama 的后台服务正在运行。
        
3. **拉取 AI 模型**:
    
    - 现在，回到你的 WezTerm 终端。
        
    - 执行拉取命令。我们推荐一个轻量且为编码优化的模型，以保证速度。

```bash
# 首选推荐 (速度最快, 约 1.3B 参数)
ollama pull deepseek-coder:1.3b-instruct

# 备选方案 (能力更强, 稍慢, 约 3B 参数)
# ollama pull stable-code:3b-code-instruct-q4_1
```

### 步骤 2：配置 Neovim 连接到 Ollama

	我们将使用 CopilotChat.nvim 插件，并将其后端从 GitHub Copilot 切换到我们本地的 Ollama 服务。

1. 创建 AI 配置文件 ~/.config/nvim/lua/plugins/ai.lua。
    
2. 粘贴以下内容：

```lua
-- ~/.config/nvim/lua/plugins/ai.lua
return {
  {
    "CopilotC-Nvim/CopilotChat.nvim",
    branch = "canary", -- ★ 使用 canary 分支以获得对 Ollama 等后端的最新支持
    dependencies = {
      { "zbirenbaum/copilot.lua" }, -- 作为底层依赖
      { "nvim-lua/plenary.nvim" }, -- 提供常用 Lua 工具
    },
    opts = {
      -- ★★★ 核心配置：定义并使用 Ollama 作为 AI 后端 ★★★
      -- 我们不使用默认的 `copilot` 后端，而是自定义一个
      provider = "ollama", -- 指定 Ollama 为提供者
      ollama_opts = {
        -- Ollama 服务的本地地址
        url = "http://127.0.0.1:11434",
        -- 使用我们下载的轻量级模型，保证流畅体验
        model = "deepseek-coder:1.3b-instruct",
        -- 控制 AI 的创造性。0.1 表示更精确、确定性的回答，适合编码
        temperature = 0.1,
        -- 你可以根据需要添加其他 ollama 参数
      },
    },
    -- 插件加载时运行的配置函数
    config = function(_, opts)
      local chat = require("CopilotChat")
      -- 应用我们的 Ollama 配置
      chat.setup(opts)

      -- ★★★ 定义你的 AI 快捷键 (Leader = 空格) ★★★
      vim.api.nvim_set_keymap("n", "<leader>ac", "<cmd>CopilotChat<CR>", { desc = "AI - Chat" })
      vim.api.nvim_set_keymap("v", "<leader>ac", "<cmd>CopilotChat<CR>", { desc = "AI - Chat with Selection" })
      vim.api.nvim_set_keymap("n", "<leader>ae", "<cmd>CopilotChatExplain<CR>", { desc = "AI - Explain Code" })
      vim.api.nvim_set_keymap("n", "<leader>at", "<cmd>CopilotChatTests<CR>", { desc = "AI - Generate Tests" })
      
      -- ★★★ 最像 Cursor 的功能：内联编辑/修复 ★★★
      -- 在可视模式下选中代码，按 <leader>aE，然后输入你的修改指令
      vim.api.nvim_set_keymap("v", "<leader>aE", "<cmd>CopilotChatEdit<CR>", { desc = "AI - Edit/Fix Selection" })
    end,
  },
}
```

	重启 Neovim 后，你的 AI 功能就绪了！

> - **AI 聊天**: 在代码中，按 `<Space`> a c，一个聊天窗口会弹出。向它提问吧！例如："用 php 写一个 curl a post request 的例子"。
    
> - **代码解释**: 将光标移动到一段你不懂的代码上，按 `<Space`> a e，AI 会为你解释它。
    
> - **代码编辑**: 用 V (可视模式) 选中一段代码，按 `<Space`> a E，在弹出的输入框中输入修改指令，如 "add type hints to all parameters"，然后回车。

---

#### **步骤 1：获取 DeepSeek API Key**

1. **注册 DeepSeek 账户**：
    
    - 访问 [https://platform.deepseek.com/](https://www.google.com/url?sa=E&q=https%3A%2F%2Fplatform.deepseek.com%2F)
        
    - 使用邮箱或手机号注册账户。
        
2. **创建 API Key**：
    
    - 登录后，进入“API密钥”页面。
        
    - 点击“创建新的密钥”。
        
    - 给你的 Key 起一个名字（如 nvim-deepseek）。
        
3. **复制并保存你的 API Key**：
    
    - 系统会生成一长串以 sk- 开头的字符串。
        
    - **同样，这是唯一一次看到完整 Key 的机会！** 立即复制并妥善保管。
	
#### **步骤 2：将 API Key 设置为环境变量**

1. **打开你的 Fish Shell 配置文件**：
```bash
nvim ~/.config/fish/config.fish
```

2. **在文件末尾添加以下行**（如果你之前加了 Groq 的，可以先注释掉或替换掉）：
```bash
# ~/.config/fish/config.fish
# ...
# 将 DeepSeek API Key 设置为环境变量
set -x DEEPSEEK_API_KEY "sk_YourDeepSeekApiKeyHere"
```

3. **重启你的 WezTerm 终端**，让环境变量生效。

#### **步骤 3：配置 Neovim 连接到 DeepSeek**

1. **打开或创建 ai.lua 文件**：
```bash
nvim ~/.config/nvim/lua/plugins/ai.lua
```
2. **粘贴以下为 DeepSeek 量身定制的配置**：
```lua
-- ~/.config/nvim/lua/plugins/ai.lua
return {
  {
    "CopilotC-Nvim/CopilotChat.nvim",
    branch = "canary",
    dependencies = {
      { "zbirenbaum/copilot.lua" },
      { "nvim-lua/plenary.nvim" },
    },
    opts = {
      -- ★★★ 核心配置：使用 OpenAI Provider 来接入 DeepSeek ★★★
      -- DeepSeek 的 API 格式与 OpenAI 兼容，所以我们可以“伪装”成 OpenAI
      provider = "openai", 
      openai_opts = {
        -- 使用 DeepSeek 的模型
        model = "deepseek-coder",
        
        -- ★★★ 关键步骤：指定 DeepSeek 的 API 地址 ★★★
        host = "api.deepseek.com",

        -- API Key 会自动从我们设置的环境变量 `DEEPSEEK_API_KEY` 中读取
        -- CopilotChat 会优先寻找 `DEEPSEEK_API_KEY`，其次是 `OPENAI_API_KEY`
        
        temperature = 0.1,
      },
    },
    config = function(_, opts)
      require("CopilotChat").setup(opts)

      local map = vim.keymap.set
      map("n", "<leader>ac", "<cmd>CopilotChat<CR>", { desc = "AI - Chat" })
      map("v", "<leader>ac", ":CopilotChat<CR>", { desc = "AI - Chat with Selection" })
      map("n", "<leader>ae", "<cmd>CopilotChatExplain<CR>", { desc = "AI - Explain Code" })
      map("n", "<leader>at", "<cmd>CopilotChatTests<CR>", { desc = "AI - Generate Tests" })
      map("v", "<leader>aE", ":CopilotChatEdit<CR>", { desc = "AI - Edit/Fix Selection" })
    end,
  },
}
```

**注意**：我们这里巧妙地利用了 provider = "openai"，因为 DeepSeek 的 API 接口标准与 OpenAI 完全兼容。我们只需要把 host 指向 DeepSeek 的服务器即可。

#### **步骤 4：重启并体验**

重启 Neovim，插件会自动安装。再次重启后，你就可以通过相同的快捷键，体验到目前市面上最强的编程专项大模型了。

下一章：[[005：实战演练 —— 从零搭建 Hyperf 项目]]

#lazyvim 