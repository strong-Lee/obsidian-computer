我们将利用 Obsidian 强大的插件生态，实现一个神奇的效果：**您在 Obsidian 中看到和编辑的是完美渲染的、清晰的 Markdown 文档，但您只需点击一个按钮，就能将该文档的“纯文本源码”一键复制到剪贴板。**

#### **第一步：安装社区插件 "Buttons"**

这是我们实现魔法的核心。

1. 打开 Obsidian 的 设置。
    
2. 进入 第三方插件，关闭 安全模式。
    
3. 点击 社区插件 旁的 浏览 按钮。
    
4. 搜索 Buttons 并安装，然后启用它。
    

#### **第二步：重构您的 Prompt 母版文件**

现在，我们将您的 Prompt 文件从一个巨大的代码块，解放为一个**结构化的、可维护的 Markdown 文档**。

1. 创建一个新的笔记，例如 AI Prompt - Master Template.md。
    
2. **移除最外层的 ``` 代码块。**
    
3. 在文件的**最顶部**，添加我们的“魔法按钮”。
    
4. 在文件的**正文**，用**最标准、最清晰的 Markdown 语法**来书写您的所有内容（包括 JSON 和表格）。
    
5. 为了让按钮能定位到要复制的内容，我们给整个正文内容添加一个**块ID (Block ID)**。

您的 AI Prompt - Master Template.md 文件看起来会是这样：

```Markdown
`button-copy`

```button

name Copy Full Prompt 
type command 
action Templater: Copy active file content 
``` ^button-end 
--- 
_👆 点击上面的按钮，即可一键复制下方所有内容的纯文本源码_ 
--- ### **第一部分：核心指令集 (Constitution)** 
【最高指令】 本次任务不是简单的样式美化或"换肤"。你需要扮演一名资深的UX/UI设计师...
```

**重要提示：上面的按钮代码需要安装另一个核心插件 Templater 才能完美工作。**

**【更简单、无需 Templater 的按钮方案】**

如果您不想安装 Templater，Buttons 插件本身也能做到，但需要稍微变通一下结构：

1. 将您的所有 Prompt 内容，包裹在一个**命名的代码块**中。
    
2. 给这个代码块添加一个**块ID**。