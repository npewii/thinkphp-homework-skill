# 第 1 页：封面

* **课程编号**：26-27-01
* **课程主题**：06 JWT 身份认证

---

# 第 2 页：4节课内容概览

### JWT 认证与 Session 认证的对比
* 状态保存位置的差异
* 关键差异对比

### JWT 的结构与原理
* JWT 的三段式结构
* HS256 签名机制
* Payload 中的标准声明

### JWT 令牌的产生与验证
* 安装 JWT 库
* 配置 HS256 密钥
* 封装 JwtService 服务类

### BaseController 统一响应封装
* 基类方法的完善
* 自定义类继承基类

### 模型登录方法的完善
* 模型登录方法的定义
* 登录控制器方法与 Token 签发

### 利用身份中间件完成 JWT 验证
* 改造 AuthMiddleware
* 控制器中获取当前用户

---

# 第 3 页：JWT 认证与 Session 认证的对比目录

1. 状态保存位置的差异
2. 关键差异对比

---

# 第 4 页：状态保存位置的差异

### 1. Session 认证
* **核心思想**：**服务器记住用户**。
* **流程**：用户登录成功后，服务器创建一个 Session 记录（通常存储在文件或 Redis 中），然后向客户端返回一个简短的 `SessionID`（一般放在 Cookie 里）。此后每次请求，浏览器自动携带这个 Cookie，服务器根据 SessionID 查找存储中的用户信息。

### 2. JWT 认证
* **核心思想**：**客户端持有并自行保存凭证**。
* **流程**：用户登录成功后，服务器用密钥对用户信息进行签名，生成一个字符串（Token）返回给客户端。客户端需要自己保存这个 Token（通常放在 localStorage 或内存中），并在每次请求时通过 `Authorization` 头发送。服务器收到 Token 后，用密钥验证签名，并从 Token 中直接解出用户信息，无需查询任何存储。

---

# 第 5 页：JWT 认证 vs Session 认证

### 表 6-1 JWT 认证 vs Session 认证

| 对比维度 | Session 认证 | JWT 认证 |
| :--- | :--- | :--- |
| **状态存储** | 服务端（文件/Redis/数据库） | 客户端（Token 自包含） |
| **服务端开销** | 每个用户占用存储，高并发时压力大 | 无状态，签名验证消耗 CPU 但不占存储 |
| **分布式扩展** | 需要 Session 共享（Redis 等） | 天然支持，无需共享 |
| **即时撤销** | 可以，删除 Session 即可 | 困难，需配合黑名单或短过期时间 |
| **移动端/跨域** | Cookie 机制受限，跨域麻烦 | 放在 Header 中，天然跨域友好 |
| **载荷安全性** | 数据在服务端，相对安全 | Payload 仅 Base64 编码，不能存敏感信息 |
| **Token 长度** | 短（仅 SessionID） | 长（包含用户信息 + 签名） |

---

# 第 6 页：JWT 的结构与原理目录

1. JWT 的三段式结构
2. HS256 签名机制
3. Payload 中的标准声明

---

# 第 7 页：JWT 的令牌格式

### Token 字符串示例
```text
"token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE3OTE0NTI5Njgsim5iZiI6MTc5MTQ1Mjk2OCwiZXhwIjoxNzkxNDYwMTY4LCJ1aWQiOjI0MDMwMTAxNzEsInVyb2xlIjoic3R1In0.XvtI3V7y_JlvJq1i5x7kiPZXW8hceLeiG3qcxS10Cqc"
```

### 表 6-2 JWT 字符串详解

| 部分 | 名称 | 内容 | 是否加密 |
| :--- | :--- | :--- | :--- |
| **第一段** | Header (头部) | 算法类型（HS256）、Token 类型（JWT） | 否，仅 Base64 编码 |
| **第二段** | Payload (载荷) | 用户信息（`uid`、`uname`、`urole`）和标准声明（`iat`、`exp`） | 否，仅 Base64 编码 |
| **第三段** | Signature (签名) | 用密钥对前两段进行 HS256 签名 | 是，无法逆向 |

---

# 第 8 页：HS256 的工作方式

### 签名计算公式
```text
签名 = HMAC-SHA256(
    Base64(Header) + "." + Base64(Payload),
    密钥
)
```

### 验证原理
验证时，服务器用同样的密钥重新计算签名，与 Token 中的第三段比对：
* **一致**：说明 Token 未被篡改；
* **不一致**：说明 Token 被伪造或修改过。

---

# 第 9 页：Payload 中的标准声明

### 表 6-3 Payload 中的常用字段

| 字段 | 全称 | 含义 | 用途 |
| :--- | :--- | :--- | :--- |
| `iat` | Issued At | 签发时间 | 记录 Token 何时生成 |
| `nbf` | Not Before | 生效时间 | 在此之前 Token 无效 |
| `exp` | Expiration Time | 过期时间 | 超过此时间 Token 失效 |

### 说明：
* 这些字段都是 **Unix 时间戳（秒）**。
* `firebase/php-jwt` 库在解码时会自动校验 `exp` 和 `nbf`，无需手动判断。

---

# 第 10 页：JWT 令牌的产生与验证目录

1. 安装 JWT 库
2. 配置 HS256 密钥
3. 封装 JwtService 服务类

---

# 第 11 页：安装 JWT 库

* **官方 Packagist**：`firebase/php-jwt`
* **安装命令**：
  ```bash
  composer require firebase/php-jwt
  ```
* **功能描述**：用于在 PHP 中对 JSON Web Tokens (JWT) 进行编码和解码的开源类库。

---

# 第 12 页：配置 HS256 密钥

### 1. `.env` 配置
```ini
[JWT]
SECRET = lzgwliyk5DVNDt+tCSIz/OjMnV44Yww9Q4sJ0XkdcEA=
EXPIRE = 7200
```

### 2. 在 `config/app.php` 中读取配置
```php
// 实例 1：在 config/app.php 中配置 Jwt 密钥
return [
    // ... 其他配置
    'jwt' => [
        'secret' => env('JWT.SECRET'),
        'expire' => env('JWT.EXPIRE', 7200),
    ],
];
```

### 说明：
1. **密钥要求**：`firebase/php-jwt` 库本身不强制校验长度，但短密钥会降低安全性，**推荐始终使用 32 字节（256 位）以上的密钥**；
2. **密钥生成**：可以使用以下命令生成一个强随机密钥：
   ```bash
   openssl rand -base64 32
   ```

---

# 第 13 页：封装 JwtService 服务类（设计规范）

### 文件路径：`app/service/JwtService.php`

### 核心设计规范说明：
1. **服务类性质**：`JwtService` 只是一个普通的业务工具类，**不应该继承 `\think\Service`**；
2. **HS256 算法**：`JWT::encode($payload, $secret, 'HS256')` 和 `JWT::decode($token, new Key($secret, 'HS256'))` 必须明确指定 HS256 算法。这很重要——如果验证时不指定算法，攻击者可能使用 `none` 算法绕过签名验证；
3. **异常分类捕获**：不同异常返回不同提示，便于调试：
   * `ExpiredException`：表示 Token 过期；
   * `SignatureInvalidException`：表示签名被篡改；
   * `BeforeValidException`：表示 Token 在 `nbf` 之前被使用；
4. **Token 提取逻辑**：前端发送格式为 `Authorization: Bearer eyJ...`。使用正则 `/^Bearer\s+(.+)$/i` 兼容大小写和多余空格，比简单的 `str_replace` 更稳健。

---

# 第 14 页：封装 JwtService 服务类（Token 生成）

```php
<?php
declare(strict_types=1);

namespace app\service;

use Firebase\JWT\JWT;

class JwtService
{
    /**
     * @var string
     */
    private $secret;

    /**
     * @var int
     */
    private $expire;

    public function __construct()
    {
        $this->secret = config('app.jwt.secret');
        $this->expire = config('app.jwt.expire');
    }

    /**
     * 生成 JWT Token
     * @param array $userData 用户数据 (uid, urole)
     * @return string Token 字符串
     */
    public function generate(array $userData): string
    {
        $currentTime = time();

        $payload = [
            'iat'   => $currentTime,                        // 签发时间
            'nbf'   => $currentTime,                        // 立即生效
            'exp'   => $currentTime + $this->expire,        // 过期时间
            'uid'   => $userData['uid'],                    // 用户 ID
            'urole' => $userData['urole']                   // 角色
        ];

        // hs256 算法生成 Token
        return JWT::encode($payload, $this->secret, 'HS256');
    }
}
```

---

# 第 15 页：封装 JwtService 服务类（验证与提取）

```php
use Firebase\JWT\Key;
use Firebase\JWT\ExpiredException;
use Firebase\JWT\SignatureInvalidException;
use Firebase\JWT\BeforeValidException;

/**
 * 验证并解析 JWT Token
 * @param string $token Token 字符串
 * @return object 解码后的 payload 对象
 * @throws \Exception 验证失败时抛出
 */
public function verify(string $token): object
{
    try {
        return JWT::decode($token, new Key($this->secret, 'HS256'));
    } catch (ExpiredException $th) {
        throw new \Exception('登录已经过期，请重新登录');
    } catch (SignatureInvalidException $th) {
        throw new \Exception('Token 签名无效');
    } catch (BeforeValidException $th) {
        throw new \Exception('Token 尚未生效');
    } catch (\Exception $th) {
        throw new \Exception('Token 解析失败');
    }
}

/**
 * 从 Authorization 头中提取 Bearer Token
 * @param string $header Authorization 头的值
 * @return string|null 纯 Token 字符串，提取失败返回 null
 */
public static function extractToken(string $header): ?string
{
    if (preg_match('/^Bearer\s+(.+)$/i', trim($header), $matches)) {
        return $matches[1];
    }
    return null;
}
```

---

# 第 16 页：作业一

### [实验] 6-1：安装 JWT 库并配置密钥
* **要求**：
  1. 使用 Composer 安装 `firebase/php-jwt`；
  2. 在 `.env` 中配置 `JWT_SECRET`（至少 32 字节）和 `JWT_EXPIRE`；
  3. 创建 `JwtService` 服务类，实现 `generate()` 和 `verify()` 方法；
  4. 编写一个测试路由，手动生成一个 Token 并打印，验证 `JwtService` 可以正常工作。
* **验收标准**：
  * `composer.json` 中出现了 `firebase/php-jwt` 依赖；
  * Token 生成成功，且包含 `iat`、`exp`、`uid` 等字段；
  * Token 验证通过，篡改后的 Token 验证失败。

---

# 第 17 页：BaseController 统一响应封装目录

1. 基类方法的完善
2. 自定义类继承基类

---

# 第 18 页：基类方法的完善（success 和 error）

### `BaseController.php` 响应方法封装
```php
/**
 * 成功响应
 * @param int $code 状态码
 * @param string $message 提示消息
 * @param array $data 响应数据
 * @return \think\response\Json
 */
public function success($code = 200, $message = '成功', $data = [])
{
    return json(
        [
            'code'    => $code,
            'message' => $message,
            'data'    => $data
        ],
        $code
    );
}
```

### 说明：
1. **关于 `json()` 的第二个参数**：`json($data, $code)` 中的第二个参数是 **HTTP 状态码**，它显示在响应头第一行，与 JSON body 里的 `code`（业务码）不同。APIFox 中前者显示在 Status 列，后者显示在 Body 的 JSON 里。
2. **业务码与 HTTP 码的统一策略**：RESTful API 规范建议 HTTP 状态码表示“协议层面是否成功”，业务码表示“业务是否成功”。本课程采用**业务码与 HTTP 码保持一致**的简化策略，降低学习成本。
3. **`error` 方法的实现**：同理，根据上述示范可封装返回错误信息的 `error($code, $message)` 方法。

---

# 第 19 页：自定义类继承 BaseController 类

所有的用户自定义类继承基类后，即可直接调用基类的方法。

### 实例 3：继承基类
```php
<?php
declare(strict_types=1);

namespace app\edu\controller;

use app\BaseController;

class User extends BaseController
{
    public function login()
    {
        // ... 其他业务代码
        return $this->success(200, 'jwt 登录认证成功', $data);
    }
}
```

---

# 第 20 页：模型登录方法的完善目录

1. 模型登录方法的定义
2. 登录控制器方法与 Token 签发

---

# 第 21 页：Stu 模型的 login 方法

```php
/**
 * jwt 登录验证
 * @param array $data 用户登录数据 (uid, upassword, urole)
 * @return array|null 登录成功返回用户详细信息 | 失败返回 null
 */
function login(array $data)
{
    // 关联预载入
    $user = $this->with('classInfo.majorInfo.depInfo')->find($data['uid']);
    
    // 比对密码
    if (!$user || $user->stu_pwd != md5($data['upassword'])) {
        return null;
    }
    
    // 返回详细信息
    return [
        'uid'   => $data['uid'],
        'uname' => $user->stu_name,
        'urole' => 'stu',
        'udep'  => $user->classInfo->majorInfo->depInfo->dep_name
    ];
}
```

### 说明：
1. 返回值采用数组格式，字段命名与 APIFox 接口定义保持一致（`uid`、`uname`、`urole`）；
2. 密码校验规则（`md5`）与第四章数据库存储一致，后续可平滑升级为 `password_hash`；
3. 模型仅返回业务所需的用户字段，不包含认证凭证；
4. 依此逻辑同样可以编写 `Te`（教师）模型的 `login` 方法。

---

# 第 22 页：登录控制器方法与 Token 签发

```php
/**
 * 登录认证
 * @return \think\Response
 */
public function login()
{
    // 通过基类的属性 $request 获取验证中间件挂载的数据
    $data = $this->request->validatedData;
    
    if ($data['urole'] == 'stu') {
        # 学生登录认证
        $userLogin = (new Stu)->login($data);
    } else {
        # 教师登录认证
        $userLogin = (new Te)->login($data);
    }

    if (!$userLogin) {
        return $this->error(401, '登录认证失败');
    }

    // jwt 令牌签发
    $token = (new JwtService)->generate($data);
    
    // 在响应数据中添加 token
    $userLogin['token'] = $token;

    // 返回成功响应
    return $this->success(200, 'jwt登录认证成功', $userLogin);
}
```

### 说明：
1. `$this->request->validatedData` 是第三章 `ValidateMiddleware` 挂载的验证后数据；
2. 根据 `$urole` 分发到不同的模型方法，实现了多角色统一登录；
3. 登录失败调用 `$this->error(401, '账号或密码错误')`，返回 HTTP 401；
4. 登录成功调用 `$this->success(200, '登录成功', $userLogin)`，返回 HTTP 200；
5. Token 被放在响应数据的 `data.token` 中返回给客户端，前端保存后在后续请求的 `Authorization: Bearer <token>` 头中携带。

---

# 第 23 页：作业二

### [实验] 6-2：在 Stu 和 Te 模型中实现 login 方法
* **要求**：
  1. 在 `Stu` 模型中编写 `login` 方法；
  2. 在 `Te` 模型中编写 `login` 方法；
  3. 两个方法都返回包含 `uid`、`uname`、`urole` 的数组，验证失败返回 `null`；
  4. 使用命令行或控制器测试两个模型的登录逻辑。
* **验收标准**：
  * 使用正确的学号/工号和密码调用，返回用户信息；
  * 使用错误的密码调用，返回 `null`；
  * 使用不存在的账号调用，返回 `null`。

---

# 第 24 页：利用身份中间件完成 JWT 验证目录

1. 改造 AuthMiddleware
2. 控制器中获取当前用户

---

# 第 25 页：改造 AuthMiddleware

```php
<?php
declare(strict_types=1);

namespace app\middleware;

use app\service\JwtService;
use think\Response;

class AuthMiddleware
{
    /**
     * 处理请求
     * @param \think\Request $request
     * @param \Closure $next
     * @return Response
     */
    public function handle($request, \Closure $next)
    {
        try {
            $header = $request->header('Authorization');
            // 提取 token
            $token = JwtService::extractToken($header);
            // 验证 token
            $payload = (new JwtService)->verify($token);
        } catch (\Throwable $th) {
            return json([
                'code'    => 401,
                'message' => $th->getMessage()
            ], 401);
        }

        // 挂载用户信息
        $request->jwtUser = [
            'uid'   => $payload->uid,
            'urole' => $payload->urole
        ];

        return $next($request);
    }
}
```

### 说明：
1. **剥离 Bearer 前缀**：`JwtService::extractToken()` 从 `Bearer eyJ...` 中提取纯 Token 部分；
2. **异常捕获与统一响应**：任何验证失败都返回 401 和统一的 JSON 格式；
3. **用户信息注入**：验证通过后，将 payload 中的用户信息挂载到 `$request->jwtUser`，控制器可直接读取注入属性，**无需重复查询数据库**；
4. **中间件不继承 BaseController**，无法调用 `$this->error()`。中间件中直接使用 `json()` 助手函数构造响应，但格式要与基类保持一致（`code`、`message` 字段）。

---

# 第 26 页：控制器中获取当前用户

### 实例 4：User 控制器的 info 方法
```php
/**
 * 获取当前登录用户的信息
 * @return array 登录用户的信息
 */
public function info()
{
    return $this->success(
        200,
        '信息获取成功',
        $this->request->jwtUser
    );
}
```

### 说明：
* 需配合修改 `info` 方法的路由定义；
* `info` 方法要经过中间件 `AuthMiddleware` 的验证。

---

# 第 27 页：作业三

### [实验] 6-3：改造 AuthMiddleware 完成 JWT 验证
* **要求**：
  1. 修改 `AuthMiddleware`，实现 Bearer Token 提取、JWT 验证、用户信息注入；
  2. 为 `logout` 和 `info` 接口绑定该中间件；
  3. 在 `User::login()` 中调用模型登录并生成 Token 返回；
  4. 在 `info` 方法中输出登录用户的信息；
  5. 使用 APIFox 测试完整流程。
* **验收标准**：
  * 登录成功，返回包含 token 的 JSON；
  * 不带 Token 访问 `/info`，返回 401；
  * 携带有效 Token 访问 `/info`，返回用户信息；
  * 携带过期或篡改的 Token，返回 401。

---

# 第 28 页：作业 1~3 的提交要求（第六次课）

### 提交内容（合并 9 张图片为一个文件，允许是图片或 PDF）：
1. **作业 1**：jwt 组件的安装过程截图；
2. **作业 1**：`.env` 代码截图；
3. **作业 1**：`JwtService` 服务类的代码截图；
4. **作业 2**：`Stu` 模型和 `Te` 模型的 `login` 方法代码截图；
5. **作业 3**：`AuthMiddleware` 中间件代码截图；
6. **作业 3**：控制器 `User` 的 `login` 方法代码截图；
7. **作业 3**：控制器 `User` 的 `info` 代码截图；
8. **作业 3**：Apifox 测试登录（学生、老师的成功 + 失败）的预览截图（含地址 + 响应）；
9. **作业 3**：Apifox 测试 info 方法（当前登录学生/老师的信息；token 有误）的预览截图。

### 提交标识和评分标准：
* 图片显眼位置写入学号；
* 方法的注释中写入学号。

---

# 第 29 页：结束

**END**