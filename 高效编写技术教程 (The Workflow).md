 ## 1. 掌握核心 Markdown 语法
 在 Obsidian 中，你只需要输入简单的符号，就能获得漂亮的格式。

# 一级标题 (文章大标题)
    
 ## 二级标题 (章节标题)
    
 **加粗文本** -> **加粗文本**
    
 *斜体文本* -> 斜体文本
    
- 无序列表项

1. 有序列表项

> 引用块，非常适合放注释或提示

## 2.  代码块：告别繁琐!
输入三个反引号 ``` 然后紧接着输入语言名称，比如 php，然后按回车。
```php
echo "hello World!";
```
Obsidian 会自动为你创建一个带有语法高亮的 PHP 代码块。你只需粘贴或编写代码即可。
```php
// 这是一段PHP代码
function hello_world(string $name): void
{
    echo "Hello, " . $name;
}
```
##  3. 内部链接：构建你的知识网络

假设你在写一篇关于 Composer 的教程，提到了 PHP。

你可以输入 [[PHP]]。

这个 "PHP" 就会变成一个可以点击的链接。如果你点击它，Obsidian 会自动为你创建一个名为 PHP.md 的新笔记，让你可以在里面专门记录 PHP 的知识。下次在任何地方再写 [[PHP]]，它都会链接到同一个地方。

## 4.  标签：灵活分类

在笔记的任何地方（通常是末尾），输入 # 加上你的标签，比如 #PHP #教程 #后端。

这可以让你快速地通过标签筛选和查找所有相关的笔记。

## 5. 终极定制：让注释变灰！

这是让你彻底爱上 Obsidian 的一步。我们将通过一个“CSS 代码片段”来实现这个效果。

1. **在 Mac 的 Obsidian 中**，点击左下角的 **设置** (齿轮图标)。
    
2. 在左侧菜单中选择 **"Appearance" (外观)**。
    
3. 向下滚动，找到 **"CSS snippets" (CSS 代码片段)** 部分。
    
4. 点击右侧的 **文件夹图标**。这会打开 Finder，定位到一个名为 snippets 的文件夹 (它在你的 TechVault/.obsidian/ 目录里)。
    
5. 在这个 snippets 文件夹里，创建一个新的纯文本文件，命名为 custom-comments.css。
```cmd
cd ~/Library/Mobile\ Documents/iCloud~md~obsidian/Documents/
cd ~/Library/Mobile\ Documents/iCloud~md~obsidian/Documents/Computer/.obsidian/snippets

touch custom-comments.css

```
    
6. 用任何文本编辑器打开这个 .css 文件，粘贴以下代码：

```css
/* 让代码块中的注释变为灰色并倾斜，增强可读性 */
.markdown-source-view.mod-cm6 .cm-comment {
  color: #999999; /* 一个舒适的灰色 */
  font-style: italic;
}
```