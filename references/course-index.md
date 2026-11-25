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

## W08：查询接口设计（课程编号26-27-01，[实验]7-1/7-2/7-3）
- **三个查询接口**（Apifox均在反代v2域名guirzv2，Headers一律`Bearer {{token}}`）：
  - 教学任务：`GET /guirzv2/tasks`；Query `startTerm/endTerm/scoreStatus/teId/page/limit(默认15)`；`GET /edu/task/index`规范地址
  - 班级成绩：`GET /guirzv2/scores/task?taskId=1`；`GET /edu/score/getTaskScore/:taskId`
  - 个人成绩单：`GET /guirzv2/scores/student`；Query `taskId/startTerm/endTerm/stuId(默认Token提取防越权)/limit/page`；`GET /edu/score/getStuScore`
- **Apifox数据模型三件**（第7-9页）：`分页`(total/per_page/current_page/last_page/list)、`task`(ta_id/ta_term/ta_te/ta_class/ta_c_name/ta_c_type/ta_c_exam)、`成绩`(stu_id/stu_name/sc_normal/sc_lab/sc_midterm/sc_final/sc_overall/sc_makeup/sc_again)；响应data引用分页模型，list ITEMS关联task模型。
- **参数分组原则**（第12页）：业务参数`$params=$request->only(['startTerm'=>0,'endTerm'=>0,'scoreStatus'=>0])`（白名单+默认值）；流程控制参数不入$params——`$teId=$request->get('teId',$request->jwtUser['uid'])`（防越权）、`$limit=(int)$request->get('limit',15)`。
- **[实验]7-1 教学任务**（第13-21页）：Task::index try-catch+`$code=$th->getCode()?:500`+total==0→'查询成功，但暂无教学任务'；`TermTool::thisTerm`（第15页）：月≤8=上y年02学期，`(int)(startY.(startY+1).termNo)`（2026.10→262701、2026.5→252602）；Task::getTask（第19-20页）：`with(['classInfo','teInfo','courseInfo'])->hasWhere('teInfo',['te_id'=>$teId])->where('ta_score',$scoreStatus)`，**hasWhere必须在where之前**；都0→TermTool精确、否则whereBetween；`array_merge($params,['teId'=>$teId])`入query保翻页；paginate后手动组装五键；foreach `?->`/`??'未知教师/未知课程/未知班级'`；task表**新增`ta_score`列**（0未提交/1部分录入/2已提交）。paginate机制（第17页）：page隐式、query保条件、越界999返空数组total真实。`?->`规则（第18页）：防脏数据恒null→Fatal，`??`兜底；主模型字段不加?->。验收：无参/区间242501-262702/limit=5/page=999四情形。
- **[实验]7-2 班级成绩**（第23-31页）：Task::getTaskScore（第29-30页）：`with(['teInfo','classInfo.stus','scores','courseInfo'])`+`!$taskRe`→`throw new \Exception('教学任务编号错误',404)`；task头部`?->/??`（ta_c_type→'未知考试类型'）；**哈希字典优化**：`$scoresDict[$score->sc_stu]=$score`（键取成绩模型sc_stu，**不是stu_id**），遍历学生`$scoresDict[$stu->stu_id]?? null`再`$stuScore?->sc_*`→未录入为null；O(N²)→O(N)空间换时间，防串号/集体变最后一条；防御`if($taskRe->classInfo && $taskRe->classInfo->stus)`；return `['task'=>$task,'score'=>$scores]`。
- **[实验]7-3 个人成绩单**（第33-41页）：Score::getStuScore（第37页）参数分组同7-1（`only(['taskId'=>0,'startTerm'=>0,'endTerm'=>0])`+stuId=jwtUser+total==0'查询成功，但暂无成绩'）；Score::getScore（第38-40页）：`with(['taskInfo.courseInfo','taskInfo.teInfo','stuInfo'])`；区间判定：都0→`hasWhere('taskInfo',['ta_term'=>term])`直传，否则`hasWhere('taskInfo',fn($q)=>$q->where('ta_term','between',...))`闭包（ThinkPHP8闭包针对关联模型）；`taskId!=0`→`where('sc_task',taskId)`；`where('sc_stu',$stuId)->paginate`+query带stuId；多级`?->`组装`stu_id/stu_name/c_name('未知')/te_name('未知')`+sc_*直接获取。
- **依赖关系补充**（课件未给定义，按W05规则补齐）：`Task::classInfo/teInfo/courseInfo`属于belongsTo、`Task::scores`hasMany(Score,'sc_task','ta_id')、`ClassModel::stus`用W05已有、`Score::taskInfo`belongsTo(Task,'sc_task','ta_id')、`Score::stuInfo`belongsTo(Stu,'sc_stu',Stu主键)；方法名与with大小写一致。
- **作业W08**：[实验]7-1~7-3合并8图一文件：TermTool服务类 / 控制器Task / 控制器Score / 模型Task / 模型Score代码（作业1~3共用）+3个Apifox预览（老师查自己任务、老师查某任务班级成绩、学生查本人某课或历年成绩）。标识：图水印学号+方法注释学号。

## W10：成绩文件的上传及处理（课程编号26-27-01，[实验]8-1/8-2/8-3）
- **PhpSpreadsheet**（第4-13页）：`composer require phpoffice/phpspreadsheet`（需ext-zip/xml/gd）；`IOFactory::load($file)->getSheet(0)->toArray()`二维数组（列索引0起：A序号0/B班级1/C学号2/D姓名3/E起成绩4）；`array_shift`去表头；临时文件直接用`$this->request->file('scoreFile')->getPathname()`不持久化（不配filesystem.php）。
- **权重设置接口**（第16-22页）：task表加4列`ta_normal_weight/ta_lab_weight/ta_midterm_weight/ta_final_weight TINYINT UNSIGNED DEFAULT 0`；业务规则=整数+总和100+期末≠0；控制器`setWeight($taskId)`用`file_get_contents('php://input')`收JSON→`modelSetWeight`（`Db::connect('edu_ru')`先验存在404再update）；路由`PATCH tasks/:taskId`；重复提交同值仍200；权重字段须同步`$schema`并加入getTaskScore返回（第51页）。
- **成绩上传接口**（第24-49页）：`POST /v2/scores?taskId=N`+form-data`scoreFile`；核心="解析+合并+校验+组装"不入库；异常双轨制：前置校验（任务不存在404/未设权重400/学号列空400/班级无学生400）抛`\Exception`中断；行级校验（学号不属本班/表内重复）收集`$errorRows`统一`throw new ValidateException(json_encode(...,JSON_UNESCAPED_UNICODE))`；控制器catch(ValidateException)返`$this->error(400,'成绩文件读取失败',json_decode($th->getMessage()))`逐条展示；数组函数：`array_column`提取学号列/`array_unique`去重/`array_filter`过滤空/`array_map('strval')`统一类型/`in_array`校验；空值兜底=`Db::name('score')->column('*','sc_stu')`建existingMap按学号补全；`$returnArray`以学号为键、成绩为字符串。
- **后续复用**：W11+做成绩确认入库/性能优化时，dealScoreFile的$returnArray即待写入数据；setWeight的权重字段供总评计算；行级校验JSON数组返回模式可复用。
- **本项目落地**：`app/edu/controller/Task.php::setWeight`、`app/edu/model/Task.php::modelSetWeight`+getTaskScore权重返回、`app/edu/controller/Score.php::importScoreFile`、`app/edu/model/Score.php::dealScoreFile`、路由`PATCH tasks/:taskId`+`POST scores`；测试产物在`homework/week10/`（README-作业说明.md含全部实测响应）。

## W11：成绩保存与事务控制（课程编号26-27-01，[实验]9-1/9-2/9-3）
- **业务场景**（第4-6页）：第八章上传Excel后系统"解析+合并+校验+组装"返待确认表单，教师核对修改后点"保存"写库，本章实现最后一步；两大核心难点=写入原子性(多条update+insert+ta_score状态更新同生共死)与并发安全(教师A保存时教师B提交ta_score=2则A不生效)→分别由事务控制与悲观锁解决。
- **接口设计**（第6-9页）：`PATCH /v2/scores?taskId=7`（Apifox `PATCH /guirzv2/scores`，Headers `Authorization: Bearer {{token}}`+`Content-Type: application/json`），Body JSON数组每行含`sc_stu_id/sc_normal/sc_lab/sc_midterm/sc_final`（字段只能多不能少，可含sc_class/sc_stu_name展示字段）；响应`code 200 message 成绩保存成功 data{update_count,insert_count}`；选PATCH因"部分更新已存在资源集合"，W10的POST scores是创建处理过程，同路径不同method双重匹配不冲突。
- **控制器saveScore**（第11页）：`Score.php::saveScore($taskId)`读`json_decode(file_get_contents('php://input'),true)`+`!is_array`返`error(400,'请求数据格式错误')`；调`(new ModelScore())->modelSaveScore($taskId,$scoreData)`；`success(200,'成绩保存成功',$scores)`；catch(ValidateException)返`error(400,'成绩保存失败',json_decode($th->getMessage()))`（模型层抛的是JSON字符串）；catch(\Throwable)`$code=$th->getCode()>200?$th->getCode():500`。
- **事务ACID**（第13-15页）：表9-2 原子(更新旧+插新+置状态同时成功)/一致(保存后ta_score=1且记录与提交一致)/隔离(行锁防并发覆盖)/持久(commit后崩溃不丢)；不用事务时插入失败会留"部分新成绩部分旧成绩"脏数据且ta_score未更新；TP事务API=`$conn->startTrans();try{...;$conn->commit();}catch(\Throwable $th){$conn->rollback();throw $th;}`。
- **[实验]9-1 连接配置**（第16-21页）：事务会话级命令START TRANSACTION/COMMIT/ROLLBACK只作用执行它的连接，update走A、insert走B则A的rollback撤不掉B的插入→**事务内所有写必须同一连接**；`edu_rc`重命名为`edu_rw`(Read&Write)三处同步(config/database.php connections键、模型/控制器`Db::connect('edu_rc')`、.env[EDU_RC]分组)；MySQL授权`GRANT SELECT,INSERT,UPDATE ON edu_db.* TO 'edu_rw_user'@'%';FLUSH PRIVILEGES;`(严谨先REVOKE ALL再GRANT)，`SHOW GRANTS FOR 'edu_rw_user'@'%';`须含UPDATE；验收用edu_rw对guirz_score执行一条UPDATE成功再改回。
- **[实验]9-2 简约版无校验**（第29-35页）：三步关系表9-6(步骤一组装沿用第八章字典结构取值改按键名/空值0，步骤二事务全新+lock(true)核心，步骤三校验9项新增4/9)；事务四要点=整体置于事务(startTrans开头,校验失败空回滚但并发概率小影响面小)、悲观锁前置`lock(true)->find`一次完成存在+已提交+加锁、更新用主键`where('sc_id',$existingMap[$row['sc_stu']])`走聚簇索引只更五成绩字段、异常rollback后必须throw；写入`update逐条`+`array_chunk(insertArray,500)`分批insertAll+task置ta_score=1+commit返`{update_count,insert_count}`；验收2全新insert2/重复update2/1+1各1/制造插入错误零写入ta_score未变/POST返405。
- **排他锁机制**（第34页）：阻止同排他锁/同UPDATE/DELETE/共享锁读；不阻止普通SELECT快照读/其他行加锁更新/score表插入/task其他行插入——写写互斥读写不互斥，把并发写串行化但不伤并发查询。
- **[实验]9-3 补全校验**（第36-43页）：表9-5九项校验=前置致命1学号非空(400)/2任务存在(404)/3权重已设(400)/4**权重合计100(400带合计值,新增)**/5班级有学生(400)，行级6学号空/7非本班/8表内重复/9**成绩0~100(新增,越界`continue 2`)**；行级统一收`$errorRows`，循环后`throw new ValidateException(json_encode($errorRows,JSON_UNESCAPED_UNICODE))`一次性拦截；`$validStu=array_flip(array_map('strval',$conn->name('stu')->where('stu_class',$taskInfo->ta_class)->column('stu_id')))`哈希表，`isset`比`in_array`高效；校验先于事务→错误时零写入不空回滚。
- **三步合并完整方法**（第44-46页）：`modelSaveScore($taskId,$scoreData)`全文流程=`array_unique(array_column($scoreData,'sc_stu_id'))`+`array_filter(trim!='')`+校验1→`Db::connect('edu_rw')`+startTrans→`name('task')->lock(true)->find`+校验2/ta_score===2→组`$overallCalculate`(ta_normal/lab/midterm/final_weight)→校验3/4(array_sum(intval)!==100)→查$validStu+校验5→`name('score')->Lock(true)->column('sc_id','sc_stu')`建$existingMap[(string)$stuId]→循环取成绩`round((float)trim(...),1)`缺省0+校验6~9+算总评`round(sum(score*weight/100),1)`组$saveScore+按existingMap分流update/insert→抛ValidateException→逐条update(主键)+分批insertAll+置ta_score=1+commit返计数→catch rollback+throw；课件`$score['sc_lab']=$taskInfo->ta_Lab_weight`大小写混用照抄可运行。
- **路由**（第48页）：`Route::group('scores',fn(){Route::get('task','getTaskScore');Route::get('student','getStuScore');Route::post('','importScoreFile');Route::patch('','saveScore');})->middleware(AuthMiddleware::class)->class(Score::class);`；实验9-2要求`PATCH /v2/scores/:taskId`路径参数式亦可。
- **后续扩展**（第50-51页）：成绩提交接口ta_score=2锁定需管理员解锁/事务内操作日志/乐观锁version字段/联合唯一索引`UNIQUE uk_sctask_scstu(sc_task,sc_stu)`+`INSERT...ON DUPLICATE KEY UPDATE`一条SQL合并更新插入(第51页DDL含score表完整结构)。
- **作业W11**（第52页+作业-11.md）：[实验]9-1~9-3合并6图一文件：edu_rw配置方案代码 / Apifox成绩保存接口地址 / 控制器saveScore代码 / 模型modelSaveScore代码 / 成绩保存接口调试成功界面(老师登录→上传Excel接口→响应复制到保存body→设taskId→提交看成功) / mysql命令行score表保存前后对比。标识：图水印学号+方法注释学号。

## W12：AI+VUE前端开发+登录验证（课程编号26-27-01，[实验]10-1/10-2）
- **整体内容**（第2-3/26页）：两大块=AI概述（工具使用技巧+通过AI认识Vue+Vue小结）与前端开发示范（登录验证开发流程/调试过程/可能的问题/测试/开发总结）；课程性质从后端PHP转前端Vue3，作业1是"与AI互动的实验报告"，作业2是"可运行的前端登录验证代码+界面/调试截图"。
- **AI工具七技巧**（第4-10页）：①先给上下文再提要求（先说Vue3+Vite+Pinia+Vue Router+UI库+后端数据结构，反面"帮我写登录"vs正面给接口文档/token方式/布局/失败成功/学生老师分支）；②用"梳理"代替"生成"（先让AI列请求方法/字段/成功失败响应，确认理解再编码）；③分阶段提问（接口理解→数据流设计→路由守卫→页面组件→布局菜单→调试）；④追问"为什么"（如角色不匹配时next在哪些分支调用）；⑤错误预判（跨域最可能在哪出问题、看浏览器哪个面板）；⑥对照后端实现细节（token生成方式/角色字段名/响应码约定，禁前端发明规则）；⑦复述认知让AI审视（用自己的话讲出来再交AI当审稿人挑刺）。
- **通过AI认识Vue十问**（第12-14页）：①环境搭建（create命令/Router Pinia ESLint选项/dev服务器默认5173）②HTTP库（axios安装/main.js挂载/为何封装拦截器而非组件裸fetch）③文档结构（index.html/main.js/App.vue/router/stores职责+createApp链式调用）④组件布局（登录独立/内部共享顶栏；嵌套路由+布局组件 vs router-view多级 vs 动态组件比较）⑤路由基础（path/name/component/meta含义、children与父router-view）⑥全局前置守卫（beforeEach执行时机/to/from/next、meta标记需登录、next()/next('/login')/next(false)三种效果）⑦分角色（meta.role与urole比较、不匹配跳哪里+提示）⑧Pinia（defineStore/state/getters/actions、token角色姓名放哪、localStorage持久化）⑨axios实例（baseURL/timeout、请求拦截Bearer、响应拦截code非200弹窗、结合`{code,message,data}`）⑩跨域（5173→8000为何跨域、server.proxy、上线同域反代无跨域）。每问"追问三轮"（机制/边界/对比辨析/落到本项目）+自己复述总结+AI审核修正；验收=覆盖10问、追问记录完整、含自己总结与AI审阅意见、能口头解释关键机制。
- **[实验]10-1**（第14页+作业-12.md）：与AI互动完成10个问题+各3个追问+各1个总结，规范实验报告（一级+二级标题+AI原始回答截图）。
- **Vue小结与环境**（第15-25页）：Vue3+Composition API+`<script setup>`；`npm create vue@latest`脚手架选项示范项目名`2026vue`、TS Yes/JSX No/Router Yes/Pinia Yes/Vitest No/E2E No/ESLint Yes/Prettier Yes；`cd 2026vue; npm install; npm run format; npm run dev`（localhost:5173 Vite欢迎页）；Node.js提供本地JS运行环境驱动Vite/构建/npm模块管理，线上纯静态dist由Nginx/Apache托管**不需要Node.js**；npm=Node Package Manager；建议Node v22.x LTS（如v22.23.5）、`npm install -g npm@latest`（Win检查`%appdata%\npm`在PATH前）；依赖`npm install pinia pinia-plugin-persistedstate element-plus @element-plus/icons-vue vue-router@4 axios`+`npm install -D sass-embedded`；`dev`内存编译+HMR vs `build`输出dist压缩混淆分包、一次性构建（对比表第25页）。
- **开发流水线八步**（第33-40页）：喂OpenAPI文档（Apifox导出OpenAPI 3.0/3.1 JSON，先读确认不写码）→梳理请求响应（方法/路径/传参位、登录体与token/角色/姓名字段、统一`{code,message,data}`）→对齐后端约定（`Authorization: Bearer <token>`、`urole:'stu'|'te'`、响应码）→定布局（登录页独立/内部共享顶栏+动态二级菜单）→提需求先出路由结构（嵌套关系+meta）与Pinia设计、暂不写页面→分模块生成逐块审查（Axios封装→Pinia Store持久化→路由定义→全局守卫→Login.vue→布局组件→业务占位页）。**人工审查死线**（第36页）：核对字段名（uid还是stu_id）与角色标识（stu/te还是数字），这一步错了后面全错。
- **关键代码**（第41-44页）：`src/router/index.ts`=三条顶层路由（`/login` meta.public独立、`/` MainLayout嵌套children业务页meta.requiresAuth/role、`/:pathMatch(.*)*` redirect '/login'）+`createRouter(createWebHistory('/'))`；`src/stores/user.ts`=defineStore state{uid,uname,urole,token}/getters{isLoggedIn:!!token,homePath:stu→/student/score,te→/teacher/task}/actions{setUserInfo(写localStorage'user_info'),logout(清state+removeItem),restore(读取+try-catch坏数据)}；`src/router/auth.ts`（作业-12要求代码截图，守卫独立成文件）=`router.beforeEach`先restore→public已登录跳homePath→未登录ElMessage.warning+`next({path:'/login',query:{redirect:to.fullPath}})`→角色不匹配ElMessage.error+`next(homePath)`（表10-1亦可用`next(false)`停当前页）→`next()`；表10-1对比：next()放行/next('/login')跳登录页改变地址栏并产生历史/next(false)停当前页地址栏不变无历史；`src/api/request.ts`=axios.create({baseURL:'/guirzv2',timeout:5000})+请求拦截注入`Authorization: Bearer ${userStore.token}`+响应拦截`res.code&&res.code!==200`弹错并reject、HTTP401→logout+push('/login')+'登录失效，请重新登录'；`src/views/Login.vue`=Element Plus学生/教师Tab+账号密码表单+错误弹窗`登录认证失败`，登录复用W02 `POST /sessions`。
- **界面与测试**（第27-32/45-47页）：登录卡师生Tab、顶栏Logo+个人签名照+系统名+用户信息/注销（曾家豪(学生)/王焕军(教师)）、二级菜单学生2项（教学任务/成绩）教师3项（+成绩录入）、任务列表含权重与标准分页、点击任务下钻班级成绩列表（含补考/重修）；F12 Network勾Preserve log看URL/Status/Headers/Payload/Response+Console看守卫输出与拦截器错误堆栈；测试四项=错误登录输入/清Token手输内部路径被守卫拦并弹回/学生越权教师路径被拦留学生首页/注销后Pinia+localStorage清空且不能后退。
- **502与部署**（第46/17/24页）：dev同源`/guirzv2`由Vite server.proxy转发后端（如localhost:8000），502 Bad Gateway=网关连不上上游（后端PHP/Apache/Nginx未启动或vite proxy target端口错）；上线同域反代无跨域，`npm run build`输出dist静态文件由Web服务器托管、线上不需Node.js。
- **作业W12**（第48-49页+作业-12.md）：[实验]10-1报告 + [实验]10-2前端登录验证，合并9图或完整PDF/Word：①AI互动10问×3追问×1总结报告（一级二级标题+截图）②router/index.ts+auth.ts ③stores/user.ts ④api/request.ts ⑤views/Login.vue ⑥学生登录失败+成功访问学生资源首页 ⑦老师登录失败+成功访问老师资源首页 ⑧学生登录后F12 Application持久化token ⑨老师登录后F12 Network的Headers/Payload/Response；防伪=Logo+个人签名照是关键标识、图片显眼处打个人学号、代码方法注释写个人学号。
- **后续复用**：W13+继续做任务列表页/成绩录入页/整合部署时，直接扩展本项目MainLayout动态菜单与业务占位页，所有接口沿用`baseURL:'/guirzv2'`+W08/W10/W11后端地址，token/角色/错误信息一律以W01-W11 OpenAPI为准。

## W09/W13-W16 待补充（已从W01教学日历提取标题）
- 初版日历：教学任务列表 / 成绩批量录入 / 录入性能优化 / 头像上传 / v1 vs v2验收 / Vue3+Axios双基址 / 登录页+路由守卫 / 任务列表页 / 成绩录入页 / 整合部署
- 实际进度对照：W08课件为"查询接口设计"（教学任务/班级成绩/个人成绩三查询接口），对应初版日历"教学任务列表"主题点，故后续周主题大概率顺延至W09起且可能合并删减，以课件实物为准。
- 注：W03实际课件为"中间件与请求验证"（含[实验]3-1/3-2），W04实际课件为"数据库配置与访问"（含[实验]4-1/4-2/4-3），W05实际课件为"模型：关联与预载入"（含[实验]5-1~5-4），W06实际课件为"JWT身份认证"（含[实验]6-1~6-3），W07实际课件为"JWT身份校验"（完善verify/extractToken+JWT进阶理论，实验沿用6-1/6-3），W08实际课件为"查询接口设计"（[实验]7-1~7-3，对应初版日历"教学任务列表"），W09仅W08课件副本留档无新课，W10实际课件为"成绩文件的上传及处理"（[实验]8-1~8-3，PhpSpreadsheet+权重+上传解析不入库），W11实际课件为"成绩保存与事务控制"（[实验]9-1~9-3，edu_rw+事务+悲观锁+九项校验），W12实际课件为"AI+VUE前端开发+登录验证"（[实验]10-1/10-2，AI七技巧+Vue十问三轮追问+Vue3/Vite/Pinia/Router/axios前端登录验证+9图）；上表标题系开学初版日历，以课件实物为准。
- 每学完一周，往此处追加：主题、关键命令/文件、复用点。保持"一周一段"。
