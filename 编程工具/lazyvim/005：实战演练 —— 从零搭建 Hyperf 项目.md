	理论终须实践。让我们用刚刚搭建好的环境，从零开始创建一个简单的 Hyperf 应用，来串联所有功能。

1. **创建 Hyperf 项目**  

	在你的工作目录下打开终端：

```bash
composer create-project hyperf/hyperf-skeleton
```

2. **用 LazyVim 打开项目**

```bash
cd hyperf-skeleton
nvim .
```



>[!INFO]
首次打开，LSP (intelephense) 会开始索引项目文件。右下角可能会有进度提示。这可能需要几十秒，之后操作就会非常流畅。

3. **文件树与代码导航**
    
    - 按 `<Space`> e 打开文件树。
        
    - 使用 j/k 上下移动，l 进入目录，h 返回上级。
        
    - 导航到 app/Controller/IndexController.php 并按 Enter 打开它。
        
4. **编写控制器方法 (LSP 和代码片段)**
    
    - 在 index() 方法下面，输入我们之前定义的片段触发词 pubf，然后按 `<Tab`>。
	
    - 一个 public function methodName() {} 模板就生成了。将 methodName 修改为 user。
        
    - 在 user 方法内，我们来模拟一个复杂点的逻辑：

```php
// 在 class IndexController 顶部添加 use 语句，LSP 会在你输入时提供自动补全
use Hyperf\HttpServer\Contract\RequestInterface;
// 假设我们有一个 App\Model\User 模型
use App\Model\User;

// ...

public function user(RequestInterface $request)
{
    $id = (int) $request->input('id', 1);

    // 将光标放在 User 上，按 gd，应该能跳转到 User 模型文件 (如果存在)
    $user = User::query()->find($id);

    if (!$user) {
        return $this->response->json(['message' => 'User not found'])->withStatus(404);
    }

    // 故意写一个错误的方法调用，比如 $user->getProfileData()
    // LSP 会立刻在 getProfileData 下画出波浪线，提示方法未定义
    // 将光标移到上面按 K 可以看到详细错误信息。现在删掉这行错误代码。

    return $this->response->json($user);
}
```

5. **AI 辅助开发 (Cursor 模式)**
    
    - **场景**: 我们想重构 user 方法，让它更健壮。
        
    - 用 V 进入可视模式，选中整个 user 方法。
        
    - 按 `<Space`> a E (Edit/Fix)，在弹出的输入框中输入指令: 添加 try-catch 块捕获数据库异常，并在 catch 中记录日志并返回 500 错误
        
    - 回车后，AI 会在右侧展示修改后的代码。你可以按 `<C-y`> (**Y**es) 接受修改，或 `<C-n`> (**N**o) 拒绝。
        
6. **手动格式化**
    
    - 在 IndexController.php 文件里，故意把一些代码的缩进搞乱。
        
    - 按 `<Space`> c f。
        
    - 观察！代码瞬间恢复了整洁的 PSR-12 格式。
        
7. **Git 提交**
    
    - 我们已经做了很多修改，是时候提交了。
        
    - 按 `<Space`> g g 打开 Lazygit。
        
    - 你会看到所有新建和修改的文件都列在右侧。
        
    - 按 a 暂存所有更改 (stage all)。
        
    - 按 c 进入提交模式，在弹出的窗口中输入你的提交信息，如 feat: Add user endpoint and model。
        
    - 按 Enter 确认提交。
        
    - 按 P (大写) 可以将你的提交**P**ush（推送）到远程仓库。
        
    - 按 q 退出 Lazygit，返回 Neovim。
        
8. **运行和测试**
    
    - 按 `<Space`> / 打开浮动终端。
        
    - 输入 php bin/hyperf.php start 启动服务。
        
    - 再打开一个新的终端窗口或 Tab，使用 curl 测试你的新接口：

```bash
curl http://127.0.0.1:9501/index/user?id=1
```

---

下一章：[[006：总结与展望]]

#lazyvim 