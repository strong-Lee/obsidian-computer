```lua
-- ~/.config/nvim/lua/plugins/colorscheme.lua
-- 在此文件中管理你的所有主题方案。
-- 通过注释和取消注释来轻松切换。

return {

  -- ===================================================================
  -- ★★★ 方案一：Catppuccin (当前启用) ★★★
  -- 柔和、现代的粉彩主题，非常受欢迎。
  -- ===================================================================
  -- {
  --   "catppuccin/nvim",
  --   name = "catppuccin",
  --   priority = 1000, -- 设置高优先级，确保它能覆盖任何默认主题
  --   config = function()
  --     require("catppuccin").setup({
  --       flavour = "frappe", -- 可选: "latte", "frappe", "macchiato", "mocha"
  --       integrations = {
  --         cmp = true,
  --         gitsigns = true,
  --         telescope = true,
  --         treesitter = true,
  --         -- 添加对 lazyvim-extra 插件的支持，让界面更统一
  --         native_lsp = {
  --           enabled = true,
  --           underlines = {
  --             errors = { "underline" },
  --             hints = { "underline" },
  --             warnings = { "underline" },
  --             information = { "underline" },
  --           },
  --         },
  --       },
  --     })
  --     -- 应用主题
  --     vim.cmd.colorscheme("catppuccin")
  --   end,
  -- },

  -- ===================================================================
  -- ▼▼▼ 方案二：Everforest (备选，当前已注释) ▼▼▼
  -- 护眼的自然绿色调主题。
  -- ===================================================================
  {
    "sainnhe/everforest",
    name = "everforest",
    priority = 1000,
    config = function()
      vim.g.everforest_background = "soft" -- 'soft', 'medium', 'hard'
      vim.g.everforest_better_performance = 1
      vim.cmd.colorscheme("everforest")
    end,
  },
}
```

#lua #lazyvim 