个推是国内领先的推送技术服务商,提供安卓(Android)和iOS推送SDK,为APP开发者提供高效稳定推送技术服务;
每个APP都需要推送，在做后端的时候，我们肯定是需要个推来实现对APP推送消息，个推的使用功能特别多，如果自己单独去开发，一定浪费大量的时间，所以我集成了Laravel个推拓展，方便大家学习使用

# 开发前的准备  
1. 安装Laravel  
1. 安装二维码生成器`QrCode`,没有安装也可以，接下来会安装

# 安装拓展
1、运行如下代码安装拓展包：
```
composer require "earnp/laravel-google-authenticator:dev-master"
# 安装二维码生成器
composer require simplesoftwareio/simple-qrcode 1.3.*

```
3.等待下载安装完成，需要在`config/app.php`中注册服务提供者同时注册下相应门面：
```php
'providers' => [
    //........
    Earnp\GoogleAuthenticator\GoogleAuthenticatorServiceprovider::class,
    SimpleSoftwareIO\QrCode\QrCodeServiceProvider::class,
],

'aliases' => [
     //..........
    'Google' => Earnp\GoogleAuthenticator\Facades\GoogleAuthenticator::class,
    'QrCode' => SimpleSoftwareIO\QrCode\Facades\QrCode::class
],
```
服务注入以后，如果要使用自定义的配置，还可以发布配置文件到config/views目录：
```php
php artisan vendor:publish
```

# 使用
使用方法非常简单，主要为生成验证码和教研验证码
### 1、生产验证码
生产验证码使用`CreateSecret`即可，你需要将其内容生成二维码供手机APP扫描,具体内容在`google.blade.php`中已经配置成功
```
// 创建谷歌验证码
$createSecret = Google::CreateSecret();
// 您自定义的参数，随表单返回
$parameter = [["name"=>"usename","value"=>"123"]];
return view('login.google.google', ['createSecret' => $createSecret,"parameter" => $parameter]);
```

### 2、校验验证码
校验验证码一般用于绑定，登录认证中，使用`CheckCode`方法即可，需要传入`secrect`和`onecode`即验证码即可进行校验，第一个为`secrect`；返回`true`或`false`

```
if(Google::CheckCode($google,$request->onecode)) {
    // 绑定场景：绑定成功，向数据库插入google参数，跳转到登录界面让用户登录
    // 登录认证场景：认证成功，执行认证操作
    dd("认证成功");
}
else
{
    // 绑定场景：认证失败，返回重新绑定，刷新新的二维码
    return back()->with('msg','请正确输入手机上google验证码 ！')->withInput();
    // 登录认证场景：认证失败，返回重新绑定，刷新新的二维码
    return back()->with('msg','验证码错误，请输入正确的验证码 ！')->withInput();
}
```

这里有一个具体的实际事例：

```
use Google;


if ($request->isMethod('post')) {
    if (empty($request->onecode) && strlen($request->onecode) != 6) return back()->with('msg','请正确输入手机上google验证码 ！')->withInput();
    // google密钥，绑定的时候为生成的密钥；如果是绑定后登录，从数据库取以前绑定的密钥
    $google = $request->google;
    // 验证验证码和密钥是否相同
    if(Google::CheckCode($google,$request->onecode)) {
        // 绑定场景：绑定成功，向数据库插入google参数，跳转到登录界面让用户登录
        // 登录认证场景：认证成功，执行认证操作
        dd("认证成功");
    }
    else
    {
        // 绑定场景：认证失败，返回重新绑定，刷新新的二维码
        return back()->with('msg','请正确输入手机上google验证码 ！')->withInput();
        // 登录认证场景：认证失败，返回重新绑定，刷新新的二维码
        return back()->with('msg','验证码错误，请输入正确的验证码 ！')->withInput();
    }
}
else
{
    // 创建谷歌验证码
    $createSecret = Google::CreateSecret();
    // 您自定义的参数，随表单返回
    $parameter = [["name"=>"usename","value"=>"123"]];
    return view('login.google.google', ['createSecret' => $createSecret,"parameter" => $parameter]);
}
```

