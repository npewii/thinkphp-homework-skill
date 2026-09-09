# 课程索引（供AI判断回顾）

> 做 W03+ 作业前先读本文件，再按需深读对应课件全文。

## W01：ThinkPHP安装与环境熟悉（26-27-01，9.2）
- **MVC理论**：M=Model表映射关联，V=View模板，C=Controller收参调度。铁律：客户端只跟Controller打交道。优点：关注点分离、并行开发、开闭原则。
- **自研痛点**：手写DB/Route/Response轮子、低效、高耦合（`DB::getInstance`硬编码）、SQL注入需手动防。
- **框架选型**：ThinkPHP（大道至简，国内社区大）vs Laravel（优雅生态大）vs Symfony（组件库）vs Yii（高性能+Gii）。
- **环境**：AppServ 9.3 / WampServer 3.2.6 + VSCode + VC运行库。`http://localhost` 验证。
- **安装**：
  ```bash
  cd D:\wamp64\www
  composer create-project topthink/think [姓名拼音缩写+学号后两位]  # 自动匹配最新兼容版
  composer require topthink/think-multi-app
  ```
  PHP要求：Composer≥5.3.2，TP8要求PHP≥8.0，`topthink/framework ^8.0`。
- **核心**：`public/index.php`单一入口，DocumentRoot必须指public。单应用`app/controller/` vs 多应用`app/应用名/controller/`。默认首页 `.../public/index.php/index/index`。
- **反向代理**：`httpd.conf`开`proxy/proxy_http/headers/rewrite`，`httpd-vhosts.conf`写：
  ```apache
  ProxyPass /v1/ http://localhost:8001/
  ProxyPass /v2/ http://localhost:8002/
  DocumentRoot .../2026vue/dist  # 前端放最后 + FallbackResource /index.html
  ```
- **作业W01**：[实验]1-1欢迎页 + [实验]1-2反代。提交6图合一长图：安装日志、首页URL、目录标注、多应用安装、vhosts代码、反代域名访问。标识：水印学号+目录名拼音+学号。

## W02：控制器的定义和路由的引入
- **控制器**：广义访问控制器，可不继承基类。单应用`app\controller\UserController`，多应用`app\index\controller\user\Blog`，访问`user.blog/index`。
- **生成**：
  ```bash
  php think build edu
  php think make:controller edu@User
  ```
  生成`app/edu/controller/User.php`，`namespace app\edu\controller`，方法`login/logout/info($id)`。
  直接访问（无路由）：`http://localhost/edu/user/login`。
- **路由四步**：定义（人工）→检测→解析→调度（后三自动）。好处：美化（`edu/user/login`→`sessions`）、隐藏控制器、参数校验、分组提效。
- **配置**：`config/app.php with_route=true`，`config/route.php url_route_must/url_lazy_route/pathinfo_depr`。文件：单应用`route/app.php`，多应用`app/edu/route/app.php`（不能省应用名）。
- **注册**：
  ```php
  use think\facade\Route;
  Route::rule('sessions','User/login','post');
  Route::post('sessions','User/login');
  Route::get('user/:id','User/info')->pattern(['id'=>'\d+']);
  Route::group('sessions', function(){
    Route::post('', 'login');
    Route::delete('', 'logout');
  })->class(User::class);
  ```
  变量规则：`\d+`数字，`\w+`字母数字下划线，`[a-zA-Z]+`字母。静态/动态/`/[:id]`可选/混合。
- **Apifox RESTful**：`POST .../v2/sessions` Body `{"uid":2403010171,"upassword":"sziit","urole":"stu"}`。前缀v1→v2。
- **JSON**：`return json(['code'=>200,'message'=>'成功','data'=>$data])`，data含`uid,urole,uname,token`。
- **模式**：普通（with_route=false）/混合（true+must=false）/强制（true+must=true，MISS即强制）。MISS：`Route::miss('Error/miss')`或闭包404。空控制器`Error#__call`。
- **作业W02**：[实验]2-1：User三方法+2个分组路由（info带学号/工号pattern）+Apifox测。[实验]2-2：三方法JSON化+Apifox设计三接口。提交4图：标准URL三方法预览、路由文件全文、User全文、路由URL+JSON预览。标识：图水印学号+方法注释学号。

## W03-W16 待补充（已从W01教学日历提取标题）
- W03 强制路由入口+Apifox定义 / W04 路由分组与中间件 / W05 JWT+登录完善 / W07 教学任务列表 / W08 成绩批量录入 / W09 录入性能优化 / W10 头像上传 / W11 v1 vs v2验收 / W12 Vue3+Axios双基址 / W13 登录页+路由守卫 / W14 任务列表页 / W15 成绩录入页 / W16 整合部署
- 每学完一周，往此处追加：主题、关键命令/文件、复用点。保持“一周一段”。
