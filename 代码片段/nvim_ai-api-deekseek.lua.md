```lua
-- ~/.config/nvim/lua/plugins/ai-api-deepseek.lua
-- ★★★ 最终方案：使用 rest.nvim 作为通用 HTTP 后端，稳定可靠 ★★★

return {
  {
    "CopilotC-Nvim/CopilotChat.nvim",
    branch = "canary",
    dependencies = {
      -- ★ 关键变更：依赖 rest.nvim
      "rest-nvim/rest.nvim",
      "nvim-lua/plenary.nvim",
    },
    opts = {
      -- ★ 关键变更：设置 provider 为 "rest"
      provider = "rest",
      -- REST 后端的配置
      rest_opts = {
        -- 指定 API 的 URL
        url = "https://api.deepseek.com/v1/chat/completions",

        -- 定义请求头 (Headers)
        headers = {
          -- API Key 从环境变量读取，并拼接成 "Bearer <key>" 的格式
          Authorization = "Bearer " .. vim.env.DEEPSEEK_API_KEY,
          ["Content-Type"] = "application/json",
        },

        -- 定义如何从返回的 JSON 中提取内容
        -- 这是 DeepSeek 标准的返回格式
        json_path = "choices[0].message.content",

        -- 定义发送到 API 的数据模板
        -- {{prompt}} 将会被替换成你的问题
        template = {
          model = "deepseek-coder",
          messages = {
            { role = "user", content = "{{prompt}}" },
          },
          temperature = 0.1,
        },
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