# 《企业级网站开发与部署》课 (Week 01)

## 第 1 页：封面
* **课程编号**：26-27-01
* **课程名称**：企业级网站开发与部署
* **日期**：2026-8-29

---

## 第 2 页：课前预习
* **预习时间**：10分钟
* **任务清单**：
  1. 阅读讲义
  2. 完成课前测试（上午10点关闭）

---

## 第 3 页：前言 —— 关于本课程
* **课程定位**：《企业级网站开发与部署》是计算机应用技术专业的一门核心课程。
* **前序课程**：《web项目应用》。
* **培养目标**：
  * 旨在让学生了解基于 MVC 框架的项目开发思路。
  * 课程选用 **ThinkPHP** 作为 MVC 的框架平台。
  * 通过课程学习，熟悉基于 ThinkPHP 的 PHP 网站开发流程，并能够独立承担网站开发过程中的任何一个开发环节。
  * 以 ThinkPHP 为契机，举一反三，未来能够基于更多的框架产品平台实施项目的开发工作。

---

## 第 4 页：教学日历与进度规划

| 周次 | 授课日期 | 授课章节及内容提要 | 课型 | 学时 | 教具 | 阶段划分 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 9.2 | ThinkPHP 安装与环境熟悉 | 讲练结合 | 4 | 计算机 | **框架开发 V2版后台** |
| 2 | 9.9 | 控制器开发与调试工具 | 讲练结合 | 4 | 计算机 | 框架开发 V2版后台 |
| 3 | 9.16 | 强制路由入口+Apifox定义 | 讲练结合 | 4 | 计算机 | 框架开发 V2版后台 |
| 4 | 9.23 | 路由分组与中间件 | 讲练结合 | 4 | 计算机 | 框架开发 V2版后台 |
| 5 | 9.30 | JWT概述+登录接口完善 | 讲练结合 | 4 | 计算机 | 框架开发 V2版后台 |
| 6 | 10.7 | 国庆放假 | - | - | - | - |
| 7 | 10.14 | 教学任务列表接口开发 | 讲练结合 | 4 | 计算机 | 框架开发 V2版后台 |
| 8 | 10.21 | 成绩批量录入 | 讲练结合 | 4 | 计算机 | 框架开发 V2版后台 |
| 9 | 10.28 | 成绩录入性能优化 | 讲练结合 | 4 | 计算机 | 框架开发 V2版后台 |
| 10 | 11.4 | 头像更新——文件上传 | 讲练结合 | 4 | 计算机 | 框架开发 V2版后台 |
| 11 | 11.11 | 阶段验收——v1 vs v2性能对比 | 讲练结合 | 4 | 计算机 | 框架开发 V2版后台 |
| 12 | 11.18 | Vue3环境+Axios双基址 | 讲练结合 | 4 | 计算机 | **AI辅助开发 VUE前端** |
| 13 | 11.25 | AI生成登录页+路由守卫 | 讲练结合 | 4 | 计算机 | AI辅助开发 VUE前端 |
| 14 | 12.2 | AI生成教学任务列表（v1接口） | 讲练结合 | 4 | 计算机 | AI辅助开发 VUE前端 |
| 15 | 12.9 | AI生成成绩录入页（v2接口） | 讲练结合 | 4 | 计算机 | AI辅助开发 VUE前端 |
| 16 | 12.16 | 项目整合+部署 | 讲练结合 | 4 | 计算机 | AI辅助开发 VUE前端 |

---

## 第 5 页：本次课内容目录
1. **MVC概述**
   * 从 PHP 项目开发引出开发模式的改进
   * MVC 模式下的站点开发
   * MVC 的工作原理
2. **基于 MVC 框架的网站开发入门**
   * 常见的 MVC 框架
   * 开发环境的搭建
   * 作业一
3. **TP框架**
   * 入口文件
   * 单应用和多应用
4. **反向代理实现**
   * 反向代理介绍
   * 统一入口和路径路由的实现
   * 作业二
5. **课程概述**
   * 学习平台 + 组队说明
   * 课程的评价体系

---

## 第 6 页：第1部分 —— MVC概述（导航）
1. 从 PHP 自研开发引出开发模式的改进
2. MVC 模式下的站点开发
3. MVC 的工作原理

---

## 第 7 页：课程《web项目应用》的自研开发方式痛点
* **既有目录架构（自研方式）**：
  * `api/` (Score.php, Stu.php, Task.php, Te.php, User.php)
  * `config/` (app.php, database.php)
  * `core/` (ApiResponse.php, DB.php, EduError.php, InputTool.php, Route.php, Sess.php, TimeTool.php)
  * `data/`、`lib/`、`test/`、`vendor/`
* **自研存在的核心问题**：
  1. **开发效率低**：需要开发者自行开发大量底层类（DB、Route、Response等）+ 业务类。
  2. **很难一步到位设计完善的文件结构**。
  3. **数据访问安全性需要手动防护**，极易遗漏（如 SQL 注入防护）。
  4. **验证规则及验证功能需要手动完善**，重复造轮子。
  5. **类的实例化不利于测试，系统解耦难度高**。

---

## 第 8 页：《web项目应用》问题剖析：开发效率低
* **底层轮子负担沉重**：开发者需自行维护底层支持文件：
  * `core/DB.php`：手写 PDO 封装、单例管理（约 200 行）。
  * `core/ApiResponse.php`：统一 JSON 返回响应规范封装。
  * `core/Route.php`：手动解析 URL 路由分发（约 230 行）。
* 导致开发业务功能前，必须耗费大量精力搭脚手架。

---

## 第 9 页：《web项目应用》问题剖析：类的实例化与硬编码高耦合
* **典型硬编码代码分析**：
```php
private function doLogin() {
    try {
        $data = InputTool::dealJson();
        InputTool::checkId($data['uid']);
        InputTool::checkPwd($data['upassword']);
        InputTool::checkRole($data['urole']);
        
        // 登录验证实施（硬编码强耦合）
        $db = DB::getInstance('guirz_r');
        if (strtolower($data['urole']) == 'stu') {
            $user = new Stu($db);
        } else {
            $user = new Te($db);
        }
        return $user->Login($data);
    } catch (\Throwable $th) {
        return ApiResponse::error($th->getCode(), $th->getMessage(), $th->getCode());
    }
}
```
* **问题诊断**：
  * 一旦写成 `$db = DB::getInstance('guirz_r')`，`doLogin` 就和 `DB` 底层类牢牢绑定。
  * 将来若想更换为其他数据库驱动（如 Mongo、Redis 或分布式代理），必须直接修改 `doLogin` 的业务源代码。
  * 这就是硬编码带来的**高耦合**，极其不利于单元测试和后续重构。

---

## 第 10 页：现有需求：自研模式能否应对？
现代企业级系统面临的多重核心挑战：
* **团队合作**（多人并行协作规范）
* **提升效率**（快速交付迭代）
* **大型项目**（复杂业务场景与高并发）
* **保证质量**（代码健壮度、标准化）
* **方便维护**（后期人员交接与升级）

---

## 第 11 页：结论
**结论：自研模式需要改进！**

---

## 第 12 页：怎么改？
围绕五大核心目标进行架构升级：**团队合作、提升效率、大型项目、保证质量、方便维护**。解决方案即：引入成熟的行业通用架构模式与框架体系（MVC）。

---

## 第 13 页：MVC 概述 —— MVC模式下的站点开发
1. 从 PHP 项目开发引出开发模式的改进
2. **MVC 模式下的站点开发**
3. MVC 的工作原理

---

## 第 14 页：基于 MVC 下的任务分离效果
* **M —— Model（数据库访问模型）**：负责数据库映射、关联关系定义及业务持久化逻辑。
* **V —— View（视图）**：负责模板渲染、HTML展示与用户交互页面。
* **C —— Controller（控制器）**：负责接收请求、参数校验、调用业务逻辑并调度视图渲染。

---

## 第 15 页：M —— Model 数据库访问模型代码解析
```php
<?php
namespace app\guirz_forum\model;

use think\Model;

class Mes extends Model
{
    // 设置表名：用于类名与表名（除去前缀）不一致的情况
    protected $table = 'guirz_mes';
    // 设置主键
    protected $pk = 'mid';
    
    // 初始化操作
    protected static function init()
    {
        parent::init();
    }
    
    // belongsTo 模式关联版块表
    public function section()
    {
        return $this->belongsTo(Section::class, 'msid');
    }
    
    // hasMany 模式关联回帖表 Res
    public function res()
    {
        return $this->hasMany(Res::class, 'rmid');
    }
}
```

---

## 第 16 页：V —— View 视图代码解析
```html
<!-- login.html 片段 -->
<div id="login_content">
    <div id="login_form">
        <strong>用户登录</strong>
        <span>还没有账号？点击<a href="{:url('reg')}">注册</a></span>
        <form action="{:url('user/doLogin')}" method="post" id="mylogin">
            <ul>
                <li>用户名 <input type="text" name="unick" /></li>
                <li>密  码 <input type="password" name="upa" /></li>
                <li>验证码 <input type="text" name="code" />{:captcha_img()}</li>
                <li><input type="submit" value="登录" /> <a href="#">找回密码</a></li>
                <li>邮件激活失败？<a href="#">重新激活</a></li>
            </ul>
        </form>
    </div>
</div>
```

---

## 第 17 页：C —— Controller 控制器代码解析
```php
<?php
namespace app\guirz_forum\controller;

use think\facade\Db;
use think\facade\Filesystem;
use app\BaseController;

class User extends BaseController
{
    protected $middleware = [
        'Auth'     => ['only' => ['me', 'upMe']],
        'CheckLog' => ['only' => ['doLogIn']],
    ];

    public function logIn()
    {
        $sec = $this->showSec();
        return view('', ['sec' => $sec]);
    }

    public function doLogIn()
    {
        // 获取表单值
        $uNick = input('unick');
        $uPa = md5(input('upa'));
        $re = Db::name('user')
            ->field('uimg,upower')
            ->where('unick', $uNick)
            ->where('upa', $uPa)
            ->find();
            
        if ($re == null) {
            $this->error('登录失败！');
        } else if ($re['upower'] == -1) {
            // 无权登录处理...
        }
    }
}
```

---

## 第 18 页：基于 MVC 下的任务分离效果对比
* **页面效果**：前台登录界面（输入账号、密码、验证码，含页头页尾）。
* **任务分解**：
  * **M: Model 模型**：访问数据库的方法、属性定义，处理底层数据逻辑。
  * **V: View 视图**：展示页面的静态页面定义（HTML/CSS/模板语法）。
  * **C: Controller 控制器**：控制在页面中放什么数据，控制数据从哪里来，充当调度中枢。

---

## 第 19 页：基于 MVC 下的任务分离效果 —— 目录结构清晰
* **项目目录与职责映射关系**：
  * `app/guirz_forum/`
    * `controller/` $\rightarrow$ 映射至 **C (Controller 控制器)**
    * `model/` $\rightarrow$ 映射至 **M (Model 模型)**
    * `view/` $\rightarrow$ 映射至 **V (View 视图)**
    * `config/`、`route/`
* **价值**：职责物理分离，避免“一锅端”代码。

---

## 第 20 页：MVC 框架提供的底层功能
成熟框架内置的大量子系统，无需开发者重复编写：
* 数据库访问（ORM、查询构造器、连接池）
* 依赖注入容器（Container / Facade）
* 环境变量配置（.env 解析）
* 文件处理与上传系统
* 多驱动缓存系统（File, Redis, Memcached）
* 日志监控与回溯系统
* 强大的路由分发与解析机制
* 中间件、事件系统、数据验证器（Validate）等

---

## 第 21 页：MVC 概述 —— 工作原理（导航）
1. 从 PHP 项目开发引出开发模式的改进
2. MVC 模式下的站点开发
3. **MVC 的工作原理**

---

## 第 22 页：MVC 模式的运作思路
* **统一接口调度流程**：
  * **M (Model 模型)**：数据库访问的模型定义（哪个表？主键？如何访问？）。定义数据库访问方式。
  * **V (View 视图)**：设计页面，负责网页前台展现。
  * **C (Controller 控制器)**：数据操作 CRUD，把数据输出到前台页面，控制协调 Model 与 View。
* **总结**：MVC 拆分了一个程序的开发过程，并构建了完善的底层逻辑。

---

## 第 23 页：MVC 模式运作思路演进 —— 使用前端框架
* **前后端分离架构的演化**：
  * 将传统的 **V (View 视图层)** 从后端代码库中剥离出来。
  * 替换为独立的现代化前端单页面框架（如 **Vue.js**、React）。
  * 后端控制器（C）纯粹以 API 形式输出 JSON 数据，由前端负责页面的渲染与展示。

---

## 第 24 页：MVC 框架的工作原理与时序图
* **交互铁律**：**客户端总是跟控制器（Controller）打交道！**
* **数据流与控制流顺序**：
  1. 客户端浏览器发起请求申请资源（HTTP Request）。
  2. 控制器（Controller）接收到请求输入（Input / Route）。
  3. 控制器调用模型层业务逻辑（Model Business Layer）。
  4. 模型层对底层数据库执行存取数据操作（Database Query）。
  5. 控制器获取数据后，将数据交由视图层进行渲染处理（View Layer）。
  6. 视图层完成页面组装并生成显示结果。
  7. 控制器将最终渲染结果或数据响应返回给客户端浏览器（HTTP Response）。

---

## 第 25 页：MVC 模式优点总结（一）
1. **关注点分离（Separation of Concerns）**：
   * Model、View、Controller 各司其职，互不干涉。
2. **提高代码的可维护性与可读性**：
   * 目录结构清晰统一。
   * 物理隔离 HTML（展示）、SQL（存取）、PHP（业务）逻辑。
3. **有利于团队并行开发**：
   * 前端工程师专职编写 View。
   * 后端工程师专职编写 Controller 和业务 API。
   * 数据库工程师（DBA）可以专职优化 Model 层的 SQL 语句与索引结构。

---

## 第 26 页：MVC 模式优点总结（二）
4. **高度可扩展性，符合“开闭原则”（Open-Closed Principle）**：
   * 功能的增加容易，不影响旧代码。
   * 视图替换容易（可随时替换模板引擎或改换输出格式）。
   * 替换底层容易（如更换 Cache 驱动或 ORM 引擎）。
5. **增强系统的健壮性与安全性**：
   * **输入过滤集中在 Controller**：可以在控制器层统一接收和过滤用户输入，防止 XSS/越权。
   * **业务逻辑集中在 Model**：重要的数据校验和业务规则在 Model 层统一处理，保障数据一致性与 SQL 防注入。

---

## 第 27 页：一句话总结 MVC
> **MVC 模式通过“分层”这种物理隔离手段，把变化（View 的展示、Model 的数据）和不变（Controller 的调度）隔离开，从而让代码更清晰、更安全、更容易应对需求变化。**

---

## 第 28 页：现有需求：MVC 模式能否应对？
* 实践证明：通过标准的 MVC 架构分层，可以完美应对：**团队合作、提升效率、大型项目、保证质量、方便维护**。

---

## 第 29 页：第2部分 —— 基于 MVC 框架的网站开发入门（导航）
1. 常见的 MVC 框架
2. 开发环境的搭建

---

## 第 30 页：常见的 MVC 框架 —— Laravel
* **Slogan**：*The PHP Framework for Web Artisans*（为 Web 艺术家创造的 PHP 框架）。
* **特点**：语法优雅、生态极为丰富、支持 Composer 生态深度整合，全球范围内使用最广。

---

## 第 31 页：常见的 MVC 框架 —— Symfony
* **特点**：
  * 一组高度可复用的 PHP 独立组件库（Component-based）。
  * 构建最佳 PHP 程序的标准化基础（包含 30 多个独立解耦组件）。
  * 许多知名框架（包括 Laravel）底层均深度采用了 Symfony 组件。

---

## 第 32 页：常见的 MVC 框架 —— Yii
* **版本**：Yii 2.0
* **含义**：在中文中代表“简单而不断演变”（Simple and Evolutionary），亦代表 **Yes It Is!**
* **特点**：高性能、基于组件、极其高效的 Gii 代码自动生成工具，适合门户、电商、CMS 系统开发。

---

## 第 33 页：本课程选择的框架：ThinkPHP
* **官方网站**：[www.thinkphp.cn](http://www.thinkphp.cn)
* **简介**：ThinkPHP 是一个免费开源的、快速、简单的面向对象的**轻量级 PHP 开发框架**，创立于 2006 年初。
* **ThinkPHP 特点**：
  * 秉持“大道至简”的设计理念与简洁实用的设计原则。
  * 拥有众多的原创功能和符合国内开发者习惯的特性。
  * 拥有庞大的国内社区与众多商业/门户级成熟案例。

---

## 第 34 页：连连看（概念抽象层次认知）
* **左侧概念**：
  * ThinkPHP
  * MVC
  * Laravel
  * 论坛系统
* **右侧网络分层类比概念**：
  * 以太网 / FDDI / 令牌环（具体协议实现）
  * OSI 七层模型（设计规范/概念模型）
  * TCP/IP（应用层实现方案）

---

## 第 35 页：连连看（对应关系公布）
* **映射逻辑剖析**：
  * **MVC $\longleftrightarrow$ OSI七层模型**（设计思想、分层理念规范）
  * **ThinkPHP / Laravel $\longleftrightarrow$ 以太网 / FDDI / 令牌环**（遵循分层思想的具体框架实现标准）
  * **教务管理系统 / 论坛系统 $\longleftrightarrow$ TCP/IP / 具体的网络应用**（在具体技术栈上构建的最终业务落地系统）

---

## 第 36 页：资源平台：www.thinkphp.cn
* **可通过官网获取的资源**：
  1. 官方帮助手册（中文文档最为详尽权威）
  2. 论坛话题（问题解答与生态讨论）
  3. 企业解决方案、生态源码与市场扩展组件

---

## 第 37 页：安装工具与资源位置
* **包依赖管理工具**：**Composer**
* **中央仓库**：Packagist (`packagist.org`)
* **代码托管源**：GitHub (`github.com/topthink/think`)

---

## 第 38 页：基于 MVC 框架的网站开发入门 —— 开发环境搭建
1. 常见的 MVC 框架
2. **开发环境的搭建**

---

## 第 39 页：开发环境的搭建：第一步 PHP 的开发环境
* **集成开发环境（二选一）**：
  * **AppServ 9.3.0** (`www.appserv.org`)：Apache 2.4.41 + PHP 7.3.10 + MySQL 8.0.17。
  * **WampServer 64 BITS (X64) 3.2.6** (`www.wampserver.com`)：Apache 2.4.51 + PHP 7.4.26 / 8.0.13 / 8.1.0 + MySQL 5.7.36 / MariaDB。
* **脚本编辑平台**：Visual Studio Code (VS Code)。
* **底层系统依赖**：
  * **Visual C++ Redistributable**（微软运行库）：Visual Studio 2015-2019 x64 or x86（必须安装，否则 Apache/PHP 无法启动）。

---

## 第 40 页：集成环境测试
* 浏览器访问：`http://localhost`
* 能够成功打开 AppServ 或 WampServer 的控制面板即表明第一步基础环境就绪。

---

## 第 41 页：开发环境搭建：第二步 ThinkPHP 框架安装
* **安装 Composer**：
  * 国内镜像源：`https://www.phpcomposer.com/`
  * 官方源：`https://packagist.org/`
  * 利用 Composer 命令行部署对应版本的 ThinkPHP 站点。
* **Composer 安装报错排查要点**：
  * 检查 VC 运行库（Visual C++ Redistributable）是否安装齐全。
  * 检查系统环境变量中的 PHP 版本是否达到框架底线要求。

---

## 第 42 页：PHP 的版本约束条件
* **Composer 最低约束**：PHP 5.3.2+ 以上版本。
* **ThinkPHP 8.0 核心依赖要求**：
  * `php: >=8.0.0`
  * `topthink/framework: ^8.0`
  * `topthink/think-filesystem: ^2.0`
  * `topthink/think-orm: ^3.0`
* **扩展组件间接要求**：例如验证码扩展 `topthink/think-captcha (v3.0.7)` 要求 `topthink/framework: ^6.0.0`。

---

## 第 43 页：Composer 资源详情页
* 在 Packagist 上检索 `topthink/think`，可查阅当前框架的最新版本（如 v8.0.0）、发布日期、依赖树约束（Dependencies/Requires）及 License（Apache-2.0）。

---

## 第 44 页：利用 Composer 部署站点命令详解
* **操作前置条件**：必须先在终端中切换到 Web 服务器的根目录（如 `www` 目录），然后再执行安装命令。
* **命令格式一（指定版本）**：
  ```bash
  composer create-project topthink/think tp 6.0.*
  ```
  * `topthink/think`：框架包名称。
  * `tp`：自定义的本地站点目录名称。
  * `6.0.*`：明确指定的版本号规则。
* **命令格式二（自动安装匹配系统的最新兼容版本）**：
  ```bash
  composer create-project topthink/think tp
  ```
  * Composer 将根据当前系统的 PHP 环境配置自动计算并安装最合适的最新版本。

---

## 第 45 页：TP 框架安装过程实操
* **命令行执行示例**：
  ```bash
  D:\wamp64\www> composer create-project topthink/think forum2023 6.1.0
  ```
* **安装日志输出**：
  * Creating a "topthink/think" project at "./forum2023"
  * Downloading topthink/think (v6.1.0)
  * Installing dependencies (think-helper, think-orm, framework, flysystem, etc.)
  * Generating autoload files
  * `> @php think service:discover` $\rightarrow$ Succeed!
  * `> @php think vendor:publish` $\rightarrow$ Succeed!
* **生成目录**：`app/`, `config/`, `extend/`, `public/`, `route/`, `runtime/`, `vendor/`, `view/`, `.example.env`, `composer.json`, `think` 等。

---

## 第 46 页：ThinkPHP 默认首页访问路径
* **初始访问 URL**：
  `http://localhost:8080/[站点目录名]/public/index.php`
* **路由寻址原理解析**：
  * 请求首先击中公共入口文件 `public/index.php` 进行框架核心加载与初始化。
  * 框架随后路由至**默认应用**下的**默认控制器**的**默认方法**（即 `Index` 控制器的 `index()` 方法）。
  * 意味着实际执行的完整隐式 URL 为：
    * 单应用模式：`.../public/index.php/index/index`
    * 多应用模式：`.../public/index.php/index/index/index`

---

## 第 47 页：作业一（阶段展示）
* **任务目标**：安装并熟悉 TP 框架，访问默认首页，确认出现官方经典“ThinkPHP 欢迎使用”大字界面。

---

## 第 48 页：官方帮助手册查阅
* **文档中心**：ThinkPHP 官方手册（`v8.*` 或 `v6.*`）。
* **必读板块**：架构、路由、控制器、请求、响应、数据库、模型、视图、错误和日志、命令行。

---

## 第 49 页：作业一实验规格书：[实验] 1-1
* **任务说明**：安装 ThinkPHP 框架并预览欢迎页。
* **硬性要求**：
  1. 在本机安装 Composer，利用 Composer 在本机部署最新版本的 TP。
  2. 项目安装到 `wamp64/www/[姓名拼音缩写+学号后两位]/`（例如 `zhangs50/`）。
  3. 安装组件：多应用模式扩展 `think-multi-app`。
  4. 在浏览器中输入 TP 默认首页的 URL 地址，访问 TP 的欢迎页。
* **过程记录表（表 1-4）填报项**：
  * 控制器 Index 的命名空间：____________________
  * 访问 TP 欢迎页的 URL：____________________

---

## 第 50 页：相关组件安装 —— 多应用模式扩展
* **组件名称**：`topthink/think-multi-app`
* **安装命令**：
  ```bash
  composer require topthink/think-multi-app
  ```
* **功能**：使 ThinkPHP 6.x/8.x 支持多模块（多应用）独立目录组织结构。

---

## 第 51 页：组件安装实操示范
* **终端安装示例**：
  ```bash
  D:\wamp64\www\forum2023> composer require liliuwei/thinkphp-jump
  ```
* **变更记录**：
  * `composer.json` 与 `composer.lock` 自动更新。
  * 类库文件下载至 `vendor/liliuwei/thinkphp-jump` 目录。
  * 自动注册命名空间映射和服务发现。

---

## 第 52 页：TP 框架底层特性（导航）
1. 入口文件
2. 单应用和多应用

---

## 第 53 页：入口文件及其安全设计原则
* **入口文件路径**：`public/index.php`
* **核心代码**：
  ```php
  <?php
  namespace think;

  require __DIR__ . '/../vendor/autoload.php';

  // 执行HTTP应用并响应
  $http = (new App())->http;
  $response = $http->run();
  $response->send();
  $http->end($response);
  ```
* **核心安全规范**：
  * ThinkPHP 采用**单一入口模式**进行项目部署和访问。
  * Web 服务器的文档根目录（DocumentRoot）在生产中**必须直接指向 `public` 目录**。
  * **除了前端静态资源外，任何框架核心代码、配置文件、模型、控制器都绝不可以放到 `public` 目录下**，避免被恶意直连下载造成代码泄露。

---

## 第 54 页：ThinkPHP 应用结构演进：单应用 vs 多应用

### 单应用模式（适合中小型/轻量级 API 站点）
```text
project_root/
├─app/
│  ├─controller/      控制器目录
│  ├─model/           模型目录
│  ├─common.php       公共函数文件
│  └─event.php        事件定义文件
├─config/             配置目录
├─view/               视图目录
├─route/              路由定义目录
├─public/             对外访问目录 (index.php)
└─vendor/             第三方类库
```

### 多应用模式（适合复杂大型、前后台完全解耦项目）
```text
project_root/
├─app/
│  ├─app_name1/       应用1（如 index 前台模块）
│  │  ├─controller/   控制器
│  │  ├─model/        模型
│  │  ├─view/         视图
│  │  ├─config/       专属配置
│  │  └─route/        专属路由
│  ├─app_name2/       应用2（如 admin 后台管理模块）
│  ├─common.php       全局公共函数
│  └─event.php        全局事件
├─config/             全局配置目录
├─public/             对外访问目录 (index.php)
└─vendor/             第三方依赖
```

---

## 第 55 页：ThinkPHP 核心目录详细功用剖析
* **`app/`：应用代码主体目录**
  * 模块目录：`app_name`（开发者自建）
  * 控制器：`app/app_name/controller/`
  * 模型：`app/app_name/model/`
  * 视图：`app/app_name/view/`
* **`config/`：全局配置文件目录**
  * 包含 `app.php`、`database.php`、`route.php`、`cache.php` 等。
* **`public/`：公开资源目录（对外根目录）**
  * 入口文件 `index.php`。
  * 静态资源自建目录：`/public/static/`。
  * 子目录：图片 `/public/static/img/`、CSS/JS样式 `/public/static/inc/`。
* **`vendor/`：Composer 类库目录**
  * 托管框架源码与所有第三方扩展组件，禁止手动修改。

---

## 第 56 页：第4部分 —— 反向代理的实现（导航）
1. 反向代理介绍
2. 统一入口和路径路由的实现

---

## 第 57 页：反向代理机制介绍
* **多系统融合与统一入口需求表**：

| 服务类型 | 技术栈 | 本地原始访问地址 | 统一暴露的期望访问路径 |
| :--- | :--- | :--- | :--- |
| **前端站点** | Vue / React 静态构建文件 | `http://localhost:8080` | `http://zhangs.sziit.edu.cn/` |
| **后端 V1 版本** | 原生 PHP 自研系统 | `http://localhost:8001` | `http://zhangs.sziit.edu.cn/v1/` |
| **后端 V2 版本** | ThinkPHP 框架 API | `http://localhost:8002` | `http://zhangs.sziit.edu.cn/v2/` |

* **反向代理与统一入口的业务价值**：
  * **单点访问**：前端与不同版本后端 API 部署在同一域名下，对外只需暴露一个域名/网址。
  * **天然跨域解决**：由于客户端所有请求均发往统一入口域名，彻底抹平浏览器的同源策略限制（CORS），自动解决跨域问题。
  * **安全隔离**：对外彻底屏蔽内部服务器物理拓扑、内网 IP 及不同服务的监听端口。

---

## 第 58 页：统一入口和路径路由的实现步骤（Apache 配置）

### 步骤一：在 `httpd.conf` 中启用必要模块（取消注释）
```apache
# 反向代理核心模块
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so

# 负载均衡模块（根据需要添加或取消注释）
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so

# 辅助重写与协议头模块
LoadModule headers_module modules/mod_headers.so
LoadModule rewrite_module modules/mod_rewrite.so
```

### 步骤二：编辑 Apache 虚拟主机配置文件（`httpd-vhosts.conf`）
```apache
# ==========================================================
# 统一入口虚拟主机配置
# ==========================================================

# 2. 后端 V1 站点：/v1/ 路径代理到原生 PHP 服务
ProxyPass /v1/ http://localhost:8001/
ProxyPassReverse /v1/ http://localhost:8001/

# 3. 后端 V2 站点：/v2/ 路径代理到 ThinkPHP 服务
ProxyPass /v2/ http://localhost:8002/
ProxyPassReverse /v2/ http://localhost:8002/

# 1. 前端站点：前端静态文件直接服务（不转发，放置在最末尾防止覆盖 V1、V2 路由规则）
DocumentRoot "${INSTALL_DIR}/www/2026vue/dist"
<Directory "${INSTALL_DIR}/www/2026vue/dist">
    Options +Indexes +Includes +FollowSymLinks
    AllowOverride All
    Require all granted
    
    # SPA 单页面路由重写支持：解决前端路由刷新 404 问题
    FallbackResource /index.html
</Directory>
```

---

## 第 59 页：作业二实验规格书：[实验] 1-2
* **任务目标**：配置虚拟主机，利用反向代理机制将域名 `【姓名.sziit.cn】` 指向项目的前端、后端 V1 和后端 V2 (TP)。
* **环境配置规范**：
  1. **前端站点**：直接使用该域名，不转发，配置监听 **80 端口**。
  2. **后端 PHP 自研网站 (V1)**：使用 `/v1/` 路径代理，实际运行于 **8001 端口**。
  3. **后端 TP 网站 (V2)**：使用 `/v2/` 路径代理，实际运行于 **8002 端口**。
  4. **验证**：使用新的反代 URL 顺利访问 TP 的欢迎页面。

---

## 第 60 页：第5部分 —— 课程概述（导航）
1. 学习平台
2. 学习小组
3. 课程的评价体系

---

## 第 61 页：职教云教学平台登录说明
* **平台网址**：[https://user.icve.com.cn/cms/](https://user.icve.com.cn/cms/)
* **统一账户迁移规则说明**：
  * 学生和教师账户已全量同步到职教云新平台。
  * 迁移过程中因加密与防重策略，学生账号前**统一添加了 `szxx` 前缀**。
  * **登录账号格式**：`szxx` + 学号（例如：`szxx2005024118`）。
  * **首次登录默认密码**：`zhzj@123ABC`。

---

## 第 62 页：职教云学习任务布局
* **教学模块**：`01 ThinkPHP安装与环境熟悉`
  * `2.1 讲义+课件`
  * `2.2 1班课堂录屏`
  * `2.3 2班课堂录屏`
* **班级考核区**：发布作业（如 01作业）与课前测试（截止时间严格锁定，超时系统关闭）。

---

## 第 63 页：学习小组建设机制
* **组建学习小组的目的**：培养团队协作意识；促进互助学习；提升作业完成质量；为期末实训 4 人大型项目攻坚做好前期团队磨合。
* **组建方式**：
  * **两人一组**（坚持自愿组合原则）。
  * **第 1~3 周为磨合期**，第三周结束时提供一次且仅有一次重新洗牌组队的机会。
  * 磨合期后队伍锁定，整个学期期间以小组为最小组织单元提交作业与项目。
* **奖励机制**：
  * 期末综合评估各组协作情况，**小组最高可额外获得 5% 的期末总评加分**。
  * 个人独立不组队者，自动弃权，无此项加分资格。

---

## 第 64 页：课程评价体系（考核权重）
* **作业**：**45%**（全周期实验报告及项目交付物，占核心大头）
* **考勤**：**30%**（课堂出勤、互动及到课纪律）
* **课堂测试**：**10%**（课前预习测验及随堂随测）
* **期末考试**：**10%**（期末阶段理论及机考）
* **组队奖励**：**5%**（团队协作增益加分）

---

## 第 65 页：第一次作业汇总提交规范要求
* **提交流程规范**：将以下 **6 项截图合并为 1 张长图** 上传提交至学习平台。
* **长图必须包含的 6 大截图要素**：
  1. 安装最新版本 TP 框架的 **CMD 命令行完整运行过程截图**。
  2. **作业一：TP 首页截图**（浏览器窗口全屏，必须清晰包含地址栏 URL）。
  3. **作业一：框架目录截图**（必须显示绝对目录路径，并明确标注出各个关键核心目录的功用）。
  4. **作业一：多应用扩展安装过程截图**（Composer 执行状态与成功日志）。
  5. **作业二：Apache `httpd-vhosts.conf` 核心配置代码截图**。
  6. **作业二：利用反代域名访问 TP 欢迎首页的截图**（浏览器必须显式展现代理后的域名地址栏）。
* **防伪与防抄袭防重标识（极重要）**：
  * **图片显眼位置必须添加学生学号作为水印**。
  * **本地物理安装根目录名称必须包含：姓名拼音 + 学号！**

---

## 第 66 页：结束页
* **END**（第一讲完）