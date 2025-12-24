### 第一部分：主要功能

1. **聊天 (Chat)**：
    
    - 这是一个交互式聊天界面，你可以像和人对话一样向 Cody 提问。
        
    - **核心优势**：**会自动利用你当前打开的项目文件作为上下文**来回答问题。你不需要手动复制粘贴大量代码。
        
2. **命令 (Commands)**：
    
    - /explain 或 :CodyExplain：解释选中的代码。
            
    - /test 或 :CodyTest：为选中的代码生成单元测试。
            
    - /doc 或 :CodyDoc：为选中的代码编写文档注释。
            
    - /fix 或 :CodyFix：尝试修复选中代码中的问题。
            
    - /smell：分析代码中的“坏味道”或可以改进的地方。
            
3. **自动补全 (Autocomplete)**：
    
    - 它生成的建议会**参考当前文件的内容和风格**，使得补全的代码更加贴合上下文。
        
4. **上下文管理 (Context Management)**：
    
    - 这是 Cody 的灵魂。它通过多种方式获取上下文：
        
        - 你当前在编辑器中打开的文件。
            
        - 你光标选中的代码片段。
            
        - 通过在聊天中 @ 符号手动引用特定文件或符号 (@app/Controller/IndexController.php)。
            
        - (Pro/企业版) 通过代码图谱和 Embeddings 技术对整个代码库建立索引，实现更深层次的理解。免费版主要依赖前几种方式，但对于中小型项目已经非常强大。
	
### 第二部分：安装Cody

```rust
-- ~/.config/nvim/lua/plugins/cody.lua

return {
  "sourcegraph/cody.vim",
  -- 建议配置一些快捷键来方便使用
  keys = {
    {
      "<leader>ac", -- "AI Chat"
      "<Cmd>CodyChat<CR>",
      mode = "n",
      desc = "Cody - Open Chat",
    },
    {
      "<leader>aq", -- "AI Question" (Quick question)
      function()
        local input = vim.fn.input("Cody > ")
        if input ~= "" then
          vim.cmd("CodyChat " .. input)
        end
      end,
      mode = "n",
      desc = "Cody - Ask a quick question",
    },
    {
      "<leader>ae", -- "AI Explain"
      "<Cmd>CodyExplain<CR>",
      mode = "v", -- v 代表在 visual 模式下使用
      desc = "Cody - Explain selected code",
    },
    {
      "<leader>at", -- "AI Test"
      "<Cmd>CodyTest<CR>",
      mode = "v",
      desc = "Cody - Generate test for selected code",
    },
  },
  -- 插件加载后执行的函数
  config = function()
    -- 你可以在这里进行一些自定义设置，但通常默认设置就很好用
    -- 例如，你可以设置 Cody 的默认聊天行为等
    -- vim.g.cody_default_chat_mode = "edit" -- 'edit' or 'insert'
  end,
}
```

1. 在 Neovim 的普通模式下，输入命令 :CodyEnable 来确保插件已启用。
    
2. 然后输入命令 :CodyAuth 并回车。
    
3. 它会自动打开你的默认浏览器，并跳转到 Sourcegraph 的授权页面。点击 "Authorize" 按钮。
    
4. 授权成功后，回到 Neovim，Cody 就已经准备就绪了。自动补全功能会开始工作。


### 第三部分：在项目中的实际应用

现在，让我们把 Cody 应用到你的 hyperf2.2 项目中，解决一些实际问题。

cd /path/to/your/hyperf-project  
nvim .

#### 示例 1：创建商场列表查询功能 (你的核心需求)

1. 打开你的 app/Controller/MallController.php 文件。如果文件不存在，先创建它。
    
2. 按下 <leader>ac (空格 + ac) 打开 Cody 聊天窗口。
    
3. 在聊天输入框中，输入一段详细的指令：
    
    > 我正在使用 Hyperf 2.2。请在当前文件 app/Controller/MallController.php 中，帮我创建一个名为 list 的方法。
    > 
    > 要求如下：
    > 
    > 1. 使用 Hyperf\HttpServer\Annotation\RequestMapping 注解，设置路由为 GET /malls。
    >     
    > 2. 方法需要注入 Hyperf\HttpServer\Contract\RequestInterface。
    >     
    > 3. 从请求中获取 page 和 per_page 参数用于分页，默认值分别为 1 和 15。
    >     
    > 4. 从请求中获取 name 参数，如果存在，则对 mall_name 字段进行模糊搜索。
    >     
    > 5. 假设我有一个 Eloquent 模型 App\Model\Mall。请使用此模型进行查询。
    >     
    > 6. 最后，返回一个包含分页数据和总数的标准 JSON 响应。
    >     
    
4. Cody 会分析你的需求，并结合 Hyperf 的特性，生成完整的 PHP 方法代码，你可以直接复制粘贴或使用 Cody 提供的插入功能。
    

#### 示例 2：解释一个复杂的依赖注入

Hyperf 的依赖注入和 AOP 很强大但也可能令人困惑。

1. 打开一个你不太理解的 Service 文件，或者 config/autoload/dependencies.php 文件。
    
2. 用 V 进入可视化模式，选中你看不懂的那一段代码，比如一个复杂的闭包工厂。
    
3. 按下 <leader>ae (空格 + ae)。
    
4. Cody 会在聊天窗口中逐行解释这段代码的作用、它解决了什么问题、以及它是如何工作的。
    

#### 示例 3：为 Controller 方法生成单元测试

1. 在 MallController.php 文件中，将光标放在你刚刚创建的 list 方法内部。
    
2. 用 V 选中整个 list 方法。
    
3. 按下 <leader>at (空格 + at)。
    
4. 在弹出的聊天窗口中，Cody 会自动开始为你生成单元测试。它可能会问你需要什么样的测试用例。你可以进一步指示它：
    
    > 请为这个方法生成 PHPUnit 测试。需要覆盖两种情况：
    > 
    > 1. 不带任何参数，验证默认分页是否正确。
    >     
    > 2. 带上 name 参数，验证搜索逻辑是否生效，并断言返回结果的数量。  
    >     请使用 Hyperf\Testing\Client 来模拟 HTTP 请求。
    >     
    

#### 示例 4：快速修复代码中的 Bug

假设你的同事写了一段代码，但是有 bug。

```php
// app/Service/OrderService.php
public function getRecentOrders(int $userId)
{
    // Bug: 这里应该用 created_at 排序，而不是 id
    return Order::where('user_id', $userId)->orderBy('id', 'desc')->limit(10);
}
```

1. 选中 return 这一行。
    
2. 打开聊天窗口，输入 /fix 或者直接提问：
    
    > 这段代码想获取最近的订单，但是排序不对，请帮我修复。应该按创建时间倒序。
    
3. Cody 会立刻给出修正后的代码： return Order::where('user_id', $userId)->orderBy('created_at', 'desc')->limit(10)->get(); (甚至可能帮你补上遗漏的 ->get())。
    

通过以上步骤和示例，你应该可以在你的环境中无缝地集成并高效地使用 Sourcegraph Cody 了。它将成为你在 Hyperf 开发中的得力助手。


