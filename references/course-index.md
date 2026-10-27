# 课程索引（供AI判断回顾）

> 做 W06+ 作业前先读本文件，再按需深读对应课件全文。

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

## W03：中间件与请求验证（课程编号26-27-01）
- **中间件四种**：全局`app/middleware.php` / 应用`app/应用/middleware.php` / 路由`->middleware(X::class)`（本课重点）/ 控制器`protected $middleware`。配置`config/middleware.php`管别名+priority，注册文件管启用。区分：文件名含middleware=配置/注册，文件夹含middleware=本体。
- **生成+骨架**：
  ```bash
  php think make:middleware AuthMiddleware
  php think make:middleware weibo@Check  # 指定应用，落 app/weibo/middleware/Check.php
  php think make:validate User           # 生成 app/validate/User.php
  ```
  入口必须是`handle(Request $request,\Closure $next)`返回`Response`；`$next之前=前置`（鉴权/校验/日志），`之后=后置`；`end(Response)`收尾不能输出（后置可能被拦截跳过，但end一定执行）。
- **[实验]3-1 AuthMiddleware**：`app/middleware/AuthMiddleware.php`前置查`$request->header('Authorization')`，缺失`return json(['code'=>401,'message'=>'未登录或 Token 无效','data'=>null])`，否则`return $next($request)`。路由`route/edu.php`给logout+info绑：`Route::group('user',fn()=>{...})->middleware(AuthMiddleware::class)`。
- **验证器三属性**：`$rule`（如`name require|max:16|min:3`、`pwd require|max:16|min:6`、`pwd2 require|confirm:pwd`、`email require|email`、`tel require|mobile`，登录场景`uid/upassword/urole`必填+长度+格式）/`$message`/`$scene`。控制器裸调`validate(LogUser::class)->batch(true)->scene('reg')->check([...])`会抛`ValidateException in Validate.php`崩溃页，必须`try-catch(ValidateException $e){return json($e->getError()/getMessage())}`再加`catch(\Exception)`兜底。
- **[实验]3-2 LoginValidate+ValidateMiddleware**：`handle(Request $request,\Closure $next,string $validateClass="",string $scene="")`，空类抛`InvalidArgumentException`（配置错误，走全局异常：开发`APP_DEBUG=true`返traces，生产返`code 0 页面错误`+写`runtime/log/`）；`try{json_decode(php://input)->validate($validateClass)->scene($scene)->check($data);$request->validatedData=$data;return $next($request);}catch(ValidateException $e){return json 400}`。路由`Route::post('', 'login')->middleware(ValidateMiddleware::class,LoginValidate::class,'login')`（三参顺序对应形参），控制器`login(Request $request){return json($request->validatedData);}`不再重复解析。
- **作业W03**：[实验]3-1+[实验]3-2合并7图一文件：Auth代码、Auth成功+失败2预览、LoginValidate代码、ValidateMiddleware代码、login方法代码、login成功1预览、login失败3预览（含Apifox地址+json+响应）。标识：图水印学号+方法注释学号。

## W04：数据库配置与访问（课程编号26-27-01，[实验]4-1/4-2/4-3）
- **配置三件套**：项目根`.env`（ini格式，`APP_DEBUG=true`，`DB_TYPE/HOST/NAME/PORT/CHARSET/PREFIX=guirz_`，`DB_DRIVER=edu_r`默认只读，`[EDU_R]/[EDU_RU]/[EDU_RC]`分组配`DB_USER/DB_PASS`）→ `config/database.php`（`default=>env('DB_DRIVER','edu_r')`，`connections.edu_r/edu_ru/edu_rc`全字段`env()`读取，`trigger_sql=>env('APP_DEBUG',true)`）→ 多应用下`app/应用/config/`覆盖全局`config/`。生产用系统环境变量、.env不进Git。
- **[实验]4-1 三连接**：`edu_r`只读（查询）、`edu_ru`读+更新、`edu_rc`读+插入；验收`Db::connect('edu_r')`可查；`connect`必须链式首位、仅当次有效。
- **Db查询构造器**：`use think\facade\Db`门面；顺序`connect→table/name/where/field/order/limit/page→find/select/value/column/count/insert/update/delete`；`name()`自动加前缀 vs `table()`全名；`find(无null)/select(toArray)/value(无null)/column(无[])`；`insertGetId/insertAll`增、`update/inc`改（返回影响条数，0=未改）、`delete(true)`删（无条件必须带true）。
- **[实验]4-2 login查库**：`User::login()`按`urole`分`stu/te`表，`Db::connect('edu_r')->name('stu')->where('stu_id',$uid)->find()`，账号不存在→401，密码比对失败→401，成功→200+用户信息；全程只读连接。
- **SQL日志调试**：`runtime/log/YYYYMM/DD.log`，三段式`CONNECT / SHOW FULL COLUMNS / SELECT ... LIMIT 1 [RunTime]`；场景：空结果（核对条件）、密码失败（确认查到用户→核密码/md5）、表不存在1146（核前缀/连接/表）；技巧：`tail -f`(Linux)/`Get-Content`(Win)实时看、按sql/error过滤、按学号搜、盯RunTime、生产关日志。
- **作业W04**：[实验]4-1+[实验]4-2+[实验]4-3合并5图一文件：.env代码、database.php代码、login方法代码、login预览4种（学生/老师×成功/失败）、日志文件标注关键信息。标识：图水印学号+方法注释学号。

## W05：模型：关联与预载入（课程编号26-27-01，[实验]5-1/5-2/5-3/5-4）
- **为什么用模型**：Db是SQL生成器面向表字段；模型是业务对象（代言人），统一获取器格式化（如性别1/2）、声明式关联、只读/废弃字段保护、复用业务规则。
- **[实验]5-1 八模型**（第9/14页）：`php think make:model edu@Stu/ClassModel/Major/Dep/Te/Course/Task/Score`落`app/edu/model/`；`Class`保留字用`ClassModel`；配`$table/$pk/$schema`/只读属性；`find()`验查；命名=去前缀表名驼峰首大写。
- **属性**（第11-12页）：`name/table/suffix/pk/connection/query/field/schema/type/strict/disuse`；默认pk=`id`，示例`$pk='uid'`；`$schema`显式定义省一次自动获取查询。
- **[实验]5-2 双向关联**（第17/19页）：`Stu::classInfo(){belongsTo(ClassModel::class,'stu_class','cl_id')}`反向；`ClassModel::stus(){hasMany(Stu::class,'stu_class','cl_id')}`正向（`$table='class'`）；验`$stu->classInfo->cl_name`、`$class->stus`。
- **表5-4六组关系**（第20页）：belongsTo放子表/hasMany放父表：`Stu→ClassModel(stu_class)`、`ClassModel→Stu`、`ClassModel→Major(cl_major)`、`Major→ClassModel`、`Major→Dep(m_dep)`、`Dep→Major`。
- **[实验]5-3 多级链**（第22页）：`ClassModel::majorInfo belongsTo Major`、`Major::classes hasMany ClassModel`、`Major::depInfo belongsTo Dep`、`Dep::majors hasMany Major`；验`$stu->classInfo->majorInfo->depInfo->dep_name`、`$dep->majors`。
- **预载入vs懒加载**（第24-27页）：懒加载`Stu::select()+foreach->$stu->classInfo`即N+1（100学生101查）；预载入`Stu::with('classInfo')->select()`恒2查（主SELECT+关联WHERE IN）；表5-5：时机/次数1+N vs 1+1/内存/简洁度/场景；列表必用预载入，单条/分支可用懒加载。
- **多级预载入**（第28-30页）：`with(['classInfo','classInfo.majorInfo','classInfo.majorInfo.depInfo'])`或简写`with(['classInfo.majorInfo.depInfo'])`；取`cl_name/m_name/dep_name`；100学生懒加载301查 vs 预载入4查；方法名大小写敏感；点号嵌套；单层可闭包`field`限字段，嵌套不限（内层闭包报错+多查）。
- **[实验]5-4 stuInfo**（第31页）：`User::stuInfo()`收学号，`with(['classInfo.majorInfo.depInfo'])->where('stu_id')->find()`返班级/专业/系部名；开SQL日志计数：单条预载入4次、单条懒加载4次（列表场景差距才大）；交日志截图注关键信息。
- **作业W05**：[实验]5-1~5-4合并4图一文件：8模型代码 / stuInfo代码 / 关联预载入日志 / 关联预载入预览。标识：图水印学号+方法注释学号。

## W06：JWT身份认证（课程编号26-27-01，[实验]6-1/6-2/6-3）
- **Session vs JWT**（第4-5页，表6-1）：Session=服务器记用户(SessionID放Cookie，需共享/占存储/可即时撤销)；JWT=客户端持凭证(Token自包含、无状态、天然分布式+跨域、难撤销靠短过期/黑名单、Payload禁敏感信息)。
- **三段式+HS256**（第7-9页，表6-2/6-3）：`Header(HS256/JWT).Payload(uid/uname/urole+iat/nbf/exp).Signature`，前两段仅Base64；`签名=HMAC-SHA256(Base64头.Base64体,密钥)`，验签重算比对；`iat签发/nbf生效/exp过期`皆Unix秒，`firebase/php-jwt`自动验exp/nbf。
- **[实验]6-1 安装配钥**（第11-12/16页）：`composer require firebase/php-jwt`；`.env[JWT]SECRET(32字节+，`openssl rand -base64 32`)+EXPIRE=7200`；`config/app.php 'jwt'=>['secret'=>env('JWT.SECRET'),'expire'=>env('JWT.EXPIRE',7200)]`；验收composer.json有依赖、Token含iat/exp/uid、篡改验签失败。注意作业-06写`JWT_SECRET/JWT_EXPIRE`即此分组含义，代码按分组写法。
- **JwtService**（第13-15页）：`app/service/JwtService.php`普通类禁继承Service；`generate`组payload后`JWT::encode HS256`；`verify`用`JWT::decode+Key(HS256)`+分类catch(Expired→过期重登/SignatureInvalid→签名无效/BeforeValid→尚未生效/兜底解析失败)；`extractToken`静态正则`/^Bearer\s+(.+)$/i`。
- **BaseController**（第18-19页）：`success($code,$message,$data)→json([...],$code)`+`error`同理，业务码=HTTP码（`json`第二参是HTTP码，Body内code是业务码）；`User extends BaseController`后调`$this->success(200,'jwt 登录认证成功',$data)`。
- **[实验]6-2 模型login**（第21/23页）：`Stu::login`用`with('classInfo.majorInfo.depInfo')->find(uid)`+`md5`比对`stu_pwd`+失败`null`+成功返`uid/uname(stu_name)/urole/udep(dep_name)`；`Te::login`同理（课件未给全文，参照Stu取教师名）；验收正确返用户、错密/无账号返null。
- **[实验]6-3 中间件+控制器**（第22/25-27页）：`User::login`读`$this->request->validatedData`(W03)按urole分发`Stu/Te::login`+失败`error(401)`+`generate($data)`+`data.token`+`success(200)`；`AuthMiddleware`全文第25页（extract→verify→`catch(\Throwable)`401→挂`$request->jwtUser=[uid,urole]`→`$next`，中间件内直接`json()`禁`$this->error`）；`User::info`返`success(200,'信息获取成功',jwtUser)`，路由绑AuthMiddleware；验收登录带token、无Token/过期/篡改访info均401。
- **作业W06**：[实验]6-1~6-3合并9图一文件：jwt安装 / .env / JwtService / Stu+Te的login / AuthMiddleware / User的login / User的info / Apifox登录(师生×成功失败) / Apifox的info(当前用户+token误)。标识：图水印学号+方法注释学号。

## W07：JWT身份校验（课程编号26-27-01，[实验]6-1/6-3 + JWT进阶理论）
- **内容性质**：前半（第4-15页）完善复习W06产物：`verify`完整实现（4类catch转中文提示，第5页）、`extractToken`实现+`$matches[0]/[1]`捕获组说明（第9-10页）；`AuthMiddleware`全文（第13页）与`User::info`全文（第14页）与W06相同但说明更细。实验沿用[实验]6-1（第11页）/6-3（第15页），**无6-2**。
- **Bearer规范**（第6页）：`PATCH http://www.suit-edu.cn/guirzv2/users/2303010350`，Headers配`Authorization: Bearer {{token}}`；`Bearer`前缀源自**OAuth 2.0 Bearer Token规范（RFC 6750）**：`credentials = "Bearer" 1*SP b64token`，Bearer后至少一个空格；语义="持有者令牌"，任何持令牌者皆可访问。
- **Apifox自动化**（第7-8页）：登录接口加"后置操作→提取变量"：来源`Response JSON`、JSONPath `$.data.token`、变量名`token`、类型`环境变量`；受保护接口Headers参数值`Bearer {{token}}`（前缀必须Bearer+至少一空格）。
- **作业W07**：[实验]6-1 JwtService代码截图 + [实验]6-3 AuthMiddleware代码/User::info代码/Apifox测info（当前登录学生/老师信息；token有误）预览，**合并4图一文件**。标识：图水印学号+方法注释学号。
- **JWT服务器行为补充**（第17-21页）：注销低安全=客户端删Token后服务器"什么都不用做"；高安全三问题=单点即时失效/多端同步失效/限额损窗口+安全续期；三方案（表第19页）：黑名单（解决单AT立即失效）、版本号/白名单（解决全端失效）、短过期+Refresh Token（限损+续期）；写jti黑名单场景（第20页）：单设备注销/踢指定设备/单token泄露/管理员吊销单会话/RT轮换后旧AT作废/手动撤销设备——原因是版本号会误伤其他会话；`token_version`递增场景（第21页）：全端注销/改密/封禁/被盗用/管理员强下线/重置密码/安全事件，通常同时吊销RT；users表DDL追加`token_version INT UNSIGNED NOT NULL DEFAULT 1`。
- **短AT+Refresh Token**（第22-29页）：需求=长期在线无感刷新+多设备互不干扰；AT=5-15分钟极短（限损窗口），RT极长、服务器存数据表或Redis（**无状态→部分有状态**，只存"还能不能用"，不存业务数据）；`refresh_tokens`表DDL（第25页）：`token_hash CHAR(64)唯一索引`、`user_id/client_id/device_info`归属、`expires_at/created_at/revoked_at`生命周期、`parent_id/replaced_by`轮换链、`ip_address/user_agent`元数据；刷新6步（第26页）：收RT→算hash查表（存在/未吊销/未过期/类型对）→**校验通过立即标记旧token已轮换（关键防重放）**→生成新RT算hash插新行→旧行`replaced_by=新行id,revoked_at=NOW()`→签发新AT（纯无状态不落库）→返回`{new_access_token, new_refresh_token}`；非法RT→吊销异常用户整条RT链路（第27页流程图）；案例（第28页）：登录得AT 10分钟+RT 1周→8分钟后401→后台静默用RT申请→服务器记账并返新AT+新RT→重试，对用户完全透明；RT失效四情况+清理策略（第29页）：自然过期交定时清理定时DELETE保留N天、主动吊销已写标记、主动注销立即UPDATE、重用/盗用立即吊销整链；Redis存储则靠TTL自动过期。
- **课堂考点**（第30-31页）：①AT未过期+RT已过期→**可以**继续访问资源（业务接口只验AT验签与有效期，不查RT）；②AT过期+RT也过期→拿过期RT刷新**拿不到**新AT（查表验`expires_at`失败），判定会话失效强制下线重登。
- **后续复用**：W08+需登录接口照常依赖JWT三件套；如遇撤销/续期/多端登录管理/改密踢人等功能题，按本节理论展开，页码引W07第17-31页。

## W08-W16 待补充（已从W01教学日历提取标题）
- W08 教学任务列表 / W09 成绩批量录入 / W10 录入性能优化 / W11 头像上传 / W12 v1 vs v2验收 / W13 Vue3+Axios双基址 / W14 登录页+路由守卫 / W15 任务列表页 / W16 整合部署
- 注：W03实际课件为"中间件与请求验证"（含[实验]3-1/3-2），W04实际课件为"数据库配置与访问"（含[实验]4-1/4-2/4-3），W05实际课件为"模型：关联与预载入"（含[实验]5-1~5-4），W06实际课件为"JWT身份认证"（含[实验]6-1~6-3），W07实际课件为"JWT身份校验"（完善verify/extractToken+JWT进阶理论，实验沿用6-1/6-3）；W07不在初版日历内，故后续周主题可能整体后移，上表标题系开学初版日历，以课件实物为准。
- 每学完一周，往此处追加：主题、关键命令/文件、复用点。保持"一周一段"。
