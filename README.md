# 2025 年必须尝试的 5 个 Laravel 新特性

## 引言

Laravel 一直在向世人证明，为什么它是最受欢迎的 PHP 框架之一。每一次版本更新，都能带来让日常开发更顺手、更干净、也更高效的新能力。如果你还没来得及研究最近的改动，下文这 **5 个全新特性** 值得你马上关注——从更聪明的事务回滚回调，到更干净的资源定义方式，统统帮你减轻心智负担。

## 1. afterRollback(): 事务失败时的自动响应

你大概已经习惯用 `DB::transaction()` 来保证数据一致性，对吧？Laravel 现在在事务工具箱里塞进了一个新帮手 —— `afterRollback()`。它会在事务失败时自动触发，让你不需要额外写 try-catch 就能处理清理、记录日志或发送通知等动作。

```
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::afterCommit(function () {
        // 事务提交成功时执行
    });
    DB::afterRollback(function () {
        // 事务回滚时执行
    });
    // 你的事务性代码...
});
```

这个特性自 **Laravel v12.32.0** 起可用。简单好用，尤其适合在回滚后记录日志、写入审计记录或清理临时数据，再也不用在事务块外层手动兜底。

## 2. Request Batching：一口气处理多个 HTTP 请求

一次性发起多个 HTTP 请求，现在变得更简洁了。借助全新的 **Request Batching**，你可以优雅地收集多个调用并统一发送，同时在生命周期的不同阶段加入回调。

```
use Illuminate\Http\Client\Batch;
use Illuminate\Support\Facades\Http;

Http::batch(function (Batch $batch) {
    return [
        $batch->get('https://batch.example/one'),
        $batch->get('https://batch.example/two'),
        $batch->get('https://batch.example/three'),
    ];
})

->before(fn (Batch $b) => logger()->info("Batch created with {$b->totalRequests} requests"))
->progress(fn (Batch $b, $key, $response) => logger()->info("Request {$key} finished!"))
->then(fn () => logger()->info("All requests completed!"))
->send();
```

你甚至可以给每个请求命名，方便后续读取：

```
$responses = Http::batch(fn (Batch $b) => [
    $b->as('users')->get('https://api.example.local/users'),
    $b->as('orders')->get('https://api.example.local/orders'),
])->send();

$users = $responses['users']->json();
```

对比以前需要手写 `Http::pool()` 并循环处理响应，Batching 让流程更清晰、结构更稳定，也更易于维护。

## 3. Dynamic Wheres：更优雅的条件查询

一个不大却格外顺手的改进：Eloquent 现在可以通过 **动态 where 方法** 来组合条件。过去你可能这么写：

```
// 以前
$order = Order::where('invoice', '123')->where('status', 'pending')->first();

// 现在
$order = Order::whereInvoiceAndStatus('123', 'pending')->first();
```

Laravel 会自动解析方法名并拼接对应的查询条件。小改动，大顺手，让链式查询的表达更贴近自然语言。

> 但个人建议不要使用，太魔法，不利于后期代码维护

## 4. 登录后的自动重定向

登录后的重定向体验也被打磨得更丝滑。借助 `redirect()->intended()`，你可以轻松把用户送回他们原本想访问的页面：

```
// middleware
return redirect()->guest(route('admin.login'))->with('error', 'Please login first');

// after login
return redirect()->intended('/admin/dashboard');
```

Laravel 会自动记住用户登录前尝试访问的地址，并在认证成功后送他们回去。体验更连贯，不需要额外维护 session 或 query 参数。

## 5. 模型资源的 PHP Attributes，让定义更简洁

以前要把模型转成资源，需要每次显式指定资源类：

```
// Before
$userData = $user->toResource(UserProfileResource::class);
$userList = $userCollection->toResourceCollection(UserCollectionResource::class);
```

现在，Laravel 允许你在模型上用 **PHP Attributes** 直接声明默认资源：

```
// Now
#[UseResource(UserDetailResource::class)]
#[UseResourceCollection(UserListResource::class)]
class User extends Model
{
    protected $fillable = ['name', 'email', 'profile_picture', 'bio'];
}

$userData = $user->toResource();
$userList = $userCollection->toResourceCollection();
```

告别重复的样板代码。在大型项目里尤为有用：资源映射统一放在模型上，逻辑更集中、调用也更干净。

## 总结

这 5 个 Laravel 新功能看似轻量，却能在日常开发中带来扎实的效率提升：

* `afterRollback()` 让事务失败后的补救动作自动化；
* Request Batching 用一套更优雅的语法覆盖多请求场景；
* Dynamic Wheres 让条件查询读起来更像自然语言；
* 登录自动重定向提升了用户体验；
* 用 PHP Attributes 声明资源，让模型与资源的衔接更顺畅。

Laravel 一如既往地在提升开发者体验上持续打磨，而不会引入不必要的复杂度。

[原文链接-2025 年必须尝试的 5 个 Laravel 新特性](https://github.com)

本博客参考[橘子云加速器](https://juziyunapp.vip)。转载请注明出处！
