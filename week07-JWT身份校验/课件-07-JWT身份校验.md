# 第 1 页：封面

* **课程编号**：26-27-01
* **课程主题**：07 JWT身份校验

---

# 第 2 页：4节课内容概览

### 完整 JwtService 服务类
* 完善校验方法 `verify`
* 完善令牌提取方法 `extractToken`

### 利用身份中间件完成 JWT 验证
* 改造 `AuthMiddleware`
* 控制器中获取当前用户

### 关于 JWT 身份认证的补充
* JWT 服务器行为补充
* 优化用户登录 —— 短 Access Token + Refresh Token

---

# 第 3 页：封装 JwtService 服务类目录

1. 完善校验方法 `verify`
2. 完善令牌提取方法 `extractToken`

---

# 第 4 页：封装 JwtService 服务类结构与规范

### 方法声明骨架
```php
/**
 * 验证并解析JWT Token
 * @param string $token Token字符串
 * @return object 解码后的payload对象
 * @throws \Exception 验证失败时抛出
 */
public function verify(string $token): object
{
    // ...
}

/**
 * 从Authorization头中提取Bearer Token
 * @param string $header Authorization头的值
 * @return string|null 纯Token字符串，提取失败返回null
 */
public static function extractToken(string $header): ?string
{
    // ...
}
```

### 设计说明：
1. **服务类性质**：`JwtService` 只是一个普通的业务工具类，不应该继承 `\think\Service`；
2. **HS256 算法**：`JWT::encode($payload, $secret, 'HS256')` 和 `JWT::decode($token, new Key($secret, 'HS256'))` 明确指定了 HS256 算法。这很重要 —— 如果验证时不指定算法，攻击者可能使用 `none` 算法绕过签名验证；
3. **异常分类捕获**：不同异常返回不同提示，便于调试。`ExpiredException` 表示 Token 过期，`SignatureInvalidException` 表示签名被篡改，`BeforeValidException` 表示 Token 在 `nbf` 之前被使用；
4. **Token 提取逻辑**：前端发送的格式是 `Authorization: Bearer eyJ...`。正则 `/^Bearer\s+(.+)$/i` 能兼容大小写和多余空格，比简单的 `str_replace` 更稳健。

---

# 第 5 页：完善 JwtService 类的方法（`verify` 实现）

```php
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
```

---

# 第 6 页：请求头的认证字段（Bearer Token 规范）

### Apifox 请求头配置示例
* **Method & URL**：`PATCH http://www.suit-edu.cn/guirzv2/users/2303010350`
* **Headers 参数**：
  * `Content-Type`: `application/json`
  * `Accept`: `application/json`
  * `Authorization`: `Bearer {{token}}`

### 标准规范与语义解析
* **为什么使用 `Bearer` 前缀**：
  在 `Authorization` 请求头中使用 `Bearer ` 前缀，根本原因是它遵循了 **OAuth 2.0 的 Bearer Token 规范 (RFC 6750)**。这个前缀并非 JWT 本身的要求，而是一种标准化的认证方案声明，用于告诉服务器“我提交的是一个持有者令牌”。
* **标准规范**：
  根据 RFC 6750 的定义，当客户端在 HTTP 请求头中传递访问令牌时，必须使用 `Bearer` 作为认证方案 (auth-scheme)。其语法格式明确为：
  `credentials = "Bearer" 1*SP b64token`，即 **Bearer 后跟一个或多个空格，再跟令牌本身**。
* **语义说明**：
  “Bearer” 一词的语义是“持有者”。它意味着**任何持有该令牌的主体（Bearer）都可以凭此访问资源**，无需再出示其他密钥或密码。JWT 作为一种自包含的令牌，通常就是作为这种“持有者令牌”来使用的。

---

# 第 7 页：练习：完善 Apifox 的登录后续操作

### 操作指引
* 在 Apifox 的“用户登录”接口中添加**后置操作**：
  * 操作类型：**提取变量**
  * 提取来源：`Response JSON`
  * JSONPath：`$.data.token`
  * 变量名称：`token`
  * 变量类型：`环境变量`

---

# 第 8 页：练习：完善资源页面的请求头设置

### 操作指引
* 在受保护接口（如“修改密码”）的 **Headers** 中配置：
  * 参数名：`Authorization`
  * 参数值：`Bearer {{token}}`
* **规则要求**：前缀必须是 `Bearer`，且 `Bearer` 后面必须有**至少一个空格**。

---

# 第 9 页：提取正则表达式说明

### 正则表达式匹配代码
```php
preg_match('/^Bearer\s+(.+)$/i', trim($header), $matches)
```

### 分组结构说明
这个正则里只有一对括号，即 `(.+)`，所以捕获组只有 1 个。`$matches` 数组的结构如下：

| 索引 | 内容 | 说明 |
| :--- | :--- | :--- |
| `$matches[0]` | 整个匹配到的字符串 | 完整匹配（第 0 组永远是整体匹配结果） |
| `$matches[1]` | 第一个捕获组的内容 | 即 `(.+)` 匹配到的纯 Token 字符串 |

---

# 第 10 页：完善 JwtService 类的方法（`extractToken` 实现）

```php
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

# 第 11 页：作业一

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

# 第 12 页：利用身份中间件完成 JWT 验证目录

1. 改造 `AuthMiddleware`
2. 控制器中获取当前用户

---

# 第 13 页：改造 AuthMiddleware

```php
<?php
declare (strict_types = 1);

namespace app\middleware;

use app\service\JwtService;
use think\Response;

class AuthMiddleware
{
    /**
     * 处理请求
     *
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
1. **剥离 Bearer 前缀**：`JwtService::extractToken()` 从 `Bearer eyJ...` 中提取纯 Token 部分，这是 JWT 认证的标准格式；
2. **异常捕获与统一响应**：任何验证失败都返回 401 和统一的 JSON 格式，与 API 返回规范一致；
3. **用户信息注入**：验证通过后，将 payload 中的用户信息挂载到 `$request->jwtUser`，控制器可直接读取注入的属性，无需重复查询数据库；
4. **统一响应格式**：中间件不继承 `BaseController`，无法调用 `$this->error()`。中间件中直接使用 `json()` 助手函数构造响应，但格式要与基类的 `error()` 保持一致（包含 `code`、`message`、`data` 三个字段）。

---

# 第 14 页：挂载荷载数据的用处——获取当前用户信息

### 控制器实现（实例 4: User 控制器的 `info` 方法）
```php
/**
 * 获取当前登录用户的信息
 * @return array 登录用户的信息
 */
public function info()
{
    return $this->success(200, '信息获取成功', $this->request->jwtUser);
}
```

### 说明：
* 需配合修改 `info` 方法的路由定义；
* `info` 方法必须经过中间件 `AuthMiddleware` 的验证。

---

# 第 15 页：作业二

### [实验] 6-3：改造 AuthMiddleware 完成 JWT 验证
* **要求**：
  1. 修改 `AuthMiddleware`，实现 Bearer Token 提取、JWT 验证、用户信息注入；
  2. 为 `logout` 和 `info` 接口绑定该中间件；
  3. 在 `User::login()` 中调用模型登录并生成 Token 返回；
  4. 在 `info` 方法中输出登录用户的信息；
  5. 使用 Apifox 测试完整流程。
* **验收标准**：
  * 登录成功，返回包含 `token` 的 JSON；
  * 不带 Token 访问 `/info`，返回 401；
  * 携带有效 Token 访问 `/info`，返回用户信息；
  * 携带过期或篡改的 Token，返回 401。

---

# 第 16 页：关于 JWT 身份认证的补充目录

1. JWT 服务器行为补充
2. 优化用户登录 —— 短 Access Token + Refresh Token

---

# 第 17 页：注销——安全级别低的做法

* JWT 的特点是服务器不保存状态，Token 本身携带签名和过期时间；
* 客户端删除本地 Token 后，后续请求不再带上它，服务器自然无法识别用户身份；
* 这种场景下，**服务器端可以“什么都不用做”**。

---

# 第 18 页：安全级别高的网站要解决的三个问题

1. **单点即时失效**：让未过期的 Access Token（单一的 Token）立即失效；
2. **多端同步失效**：让该用户所有旧 Access Token 立即失效（多设备登录管理）；
3. **安全风险控制**：限制 Access Token 泄露后的损失窗口，并提供安全续期能力。

---

# 第 19 页：高安全级别的服务行为补充

| 方案 | 解决的核心问题 | 作用对象 |
| :--- | :--- | :--- |
| **黑名单** | 让未过期的 Access Token 立即失效 | Access Token |
| **版本号 / 白名单** | 让该用户所有旧 Access Token 立即失效 | Access Token |
| **短过期 + Refresh Token** | 限制 Access Token 泄露后的损失窗口，并提供续期能力 | Access Token 的“寿命” + Refresh Token 的“可吊销” |

---

# 第 20 页：需要把 Access Token 的 id 写入黑名单的场景

| 场景 | 写什么 | 为什么不用版本号 |
| :--- | :--- | :--- |
| **单设备注销**（只退出当前设备） | 当前 Access Token 的 `jti` | 版本号会踢掉该用户所有设备，误伤其他会话 |
| **踢掉某一台指定设备**（如“登录设备管理”里移除某台） | 那台设备对应 Access Token 的 `jti` | 同上，只需精确吊销这一个 |
| **单个 token 被判定泄露/异常**（如异地异常使用） | 该 token 的 `jti` | 只阻断这一个可疑凭证，其他正常会话不受影响 |
| **管理员精确吊销某个会话** | 目标会话的 `jti` | 粒度是“一个会话”，不是“整个用户” |
| **Refresh Token 轮换后，旧 Access Token 想立即作废**（可选） | 旧 Access Token 的 `jti` | 只针对这一条链，不涉及用户其他登录 |
| **令牌被用户手动“撤销此设备授权”** | 该设备 token 的 `jti` | 精确到设备 |

---

# 第 21 页：需要增加 Access Token 版本号的场景

| 场景 | 为什么递增 | 是否通常同时吊销 Refresh Token |
| :--- | :--- | :--- |
| **用户主动注销（全端登出）** | 让所有设备上的旧 Access Token 立即失效 | 是 |
| **修改密码** | 密码变了，旧 token 不应再有效，防止旧密码泄露后被继续利用 | 是 |
| **账号被封禁/禁用** | 立即剥夺该用户所有访问能力 | 是 |
| **检测到 token 被盗用/异常登录** | 立即阻断攻击者手中的所有旧 token | 是 |
| **管理员强制下线某用户** | 运营/安全管控需要 | 是 |
| **用户主动“登出所有设备”** | 一次性踢掉全部会话 | 是 |
| **重置密码（忘记密码流程）** | 同改密码，旧会话必须失效 | 是 |
| **安全事件**（如数据库泄露后强制全员重置） | 批量作废旧 token | 是 |

### 数据表改造 DDL
```sql
CREATE TABLE users (
    id             BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username       VARCHAR(64) NOT NULL,
    password_hash  VARCHAR(255) NOT NULL,
    email          VARCHAR(128),
    status         TINYINT NOT NULL DEFAULT 1,
    created_at     DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    -- 新增版本号字段：
    token_version  INT UNSIGNED NOT NULL DEFAULT 1
);
```
* **说明**：`token_version` 是给每个用户维护的一个整数版本号，初始为 1。

---

# 第 22 页：用户登录体验需求

* **长期在线**：登录一次，在线很久（无感刷新）。
* **多设备支持**：多设备登录，互不干扰，可独立管理。

---

# 第 23 页：优化用户登录——短过期 + Refresh Token

### 1. 短 Access Token
* **签发策略**：设定极短的 Access Token 有效期；
* **机制**：在认证服务器生成令牌时，将 Access Token 的过期时间（`exp`）设置为 **5 到 15 分钟**。这意味着即使这个令牌被泄露，攻击者能利用的时间窗口也非常有限。

### 2. Refresh Token
* **签发策略**：设定一个极长时间的 Refresh Token；
* **机制**：在 Access Token 快过期时，客户端拿着 Refresh Token 到服务器端领一个新的 Access Token；服务器端产生一个新的 Access Token（最好也同时产生一个新的 Refresh Token）下发给客户端。

---

# 第 24 页：无状态 → 部分有状态

### 服务器端维护 Refresh Token 状态
* **存储位置**：认证服务器；
* **存储方式**：数据表或 Redis；
* **核心作用**：让服务器能判断“**这个 Refresh Token 还能不能用**”，并支持主动作废。
* **架构边界**：它不承载用户资料、权限等业务状态 —— 那些业务数据仍然存放在 JWT 荷载中或单独查询。

---

# 第 25 页：数据表设计（`refresh_tokens`）

```sql
CREATE TABLE refresh_tokens (
    id           BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    
    -- 令牌哈希（唯一索引，查询用）：存储 Refresh Token 的哈希值
    token_hash   CHAR(64) NOT NULL,
    
    -- 归属：对应的客户
    user_id      BIGINT UNSIGNED NOT NULL,
    client_id    VARCHAR(64) NULL, -- 哪个客户端/设备
    device_info  VARCHAR(255) NULL, -- UA、设备名（可选，便于用户管理会话）
    
    -- 生命周期：Refresh Token 的有效时间字段
    expires_at   DATETIME NOT NULL,
    created_at   DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    revoked_at   DATETIME NULL, -- NULL 表示未吊销
    
    -- 轮换链（支持 Rotation 和重用检测）：Refresh Token 轮换的历史记录
    parent_id    BIGINT UNSIGNED NULL, -- 由哪个 token 轮换而来
    replaced_by  BIGINT UNSIGNED NULL, -- 被哪个新 token 替代
    
    -- 元数据
    ip_address   VARCHAR(45) NULL,
    user_agent   VARCHAR(512) NULL,

    UNIQUE KEY uk_token_hash (token_hash),
    KEY idx_user_id (user_id),
    KEY idx_expires_at (expires_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

# 第 26 页：解读客户端登录后 Access Token 的更新过程

1. **接收与校验**：收到 refresh token ──> 算哈希 ──> 查表校验（必须满足：存在、未吊销、未过期、类型正确）；
2. **【关键防重放】**：校验通过后，立即标记旧 token 为 **"已使用/已轮换"**（防止并发重复使用）；
3. **生成新凭证**：生成新 refresh token ──> 算哈希 ──> 插入新行；
4. **维护关系链**：更新旧行数据：`replaced_by = 新行id, revoked_at = NOW()`；
5. **签发新 AT**：生成新 access token（纯无状态，不落库）；
6. **响应返回**：返回 `{ new_access_token, new_refresh_token }`。

---

# 第 27 页：解读客户端登录后 Access Token 的更新流程图

```text
[客户端 + Refresh Token (RT)]
         │
  (1) AT 马上到期，发起刷新请求
         │
         ▼
  (2) [服务器端处理请求] ──> 验证 RT 是否合法
         │
    ┌────┴─────────────────────────────┐
    │                                  │
[合法 RT]                          [非法 RT]
    │                                  │
    ▼ (3)                              ▼ (3)
[服务器：改 + 写数据表]             [服务器：吊销异常用户整条 RT 链路]
 ├── 修改旧 RT 的状态为已吊销/已轮换
 ├── 新 RT 写入新行
 └── 更新旧 RT 与新 RT 的轮换关系
    │
    ▼ (4)
[服务器：生成新的 AT]
    │
    ▼
[把新 AT + 新 RT 发送回客户端]
```

---

# 第 28 页：案例分析

1. **首次登录**：客户端获取 `AT`（假设 10 分钟有效） + `RT`（假设 1 周有效）；
2. **8 分钟后请求**：客户端访问资源，服务器端返回 `401`（Token 快过期或已过期）；
3. **自动续期触发**：客户端收到 401 响应后，后台静默拿着 `RT` 向服务器端申请新的 `AT`。RT 合法，服务器端记录新的 RT 信息，更新旧的 RT 信息；
4. **服务器下发凭证**：服务器端返回新的 `AT` + 新的 `RT`；
5. **重试原始请求**：客户端带着新的 `AT` 重新访问资源 —— **整个 AT 续期过程对用户完全透明无感**。

---

# 第 29 页：Refresh Token 失效的几种情况

| 失效来源 | 客户端表现 | 数据表操作 |
| :--- | :--- | :--- |
| **自然过期**（`expires_at` 已过） | 刷新失败 → 强制下线 | 无需立即操作，交给定时清理 |
| **被服务端主动吊销**（`revoked_at` 非空） | 刷新失败 → 强制下线 | 吊销时已写入，无需额外操作 |
| **用户主动注销** | 立即下线 | 立即 `UPDATE` 写入吊销标记 |
| **检测到重用 / 盗用** | 强制下线 | 立即吊销整条轮换链（全链失效） |

> **清理策略**：事件调度器分批执行 `DELETE` 操作，保留 N 天后彻底删除；可先归档；若使用 Redis 存储则直接依赖 `TTL` 自动过期。

---

# 第 30 页：课堂提问 1

### 提问
* **问题**：Access Token 未过期，Refresh Token 已过期，客户可以继续访问资源页面吗？
* **答案解析**：**可以**。因为业务接口鉴权只验证 Access Token 的有效性（验签与有效期），不会在每次业务请求时都去查验 Refresh Token 的状态，直到 Access Token 过期为止。

---

# 第 31 页：课堂提问 2

### 提问
* **问题**：Access Token 过期 + Refresh Token 也过期，客户端拿过期 Refresh Token 来刷新，会拿到新的 Access Token 吗？
* **答案解析**：**不会**。服务器端查表时会校验 Refresh Token 的 `expires_at`，若已过期则刷新失败，直接判定会话失效，客户端必须强制下线重新登录。

---

# 第 32 页：作业 1~2 的提交要求（第七次课）

### 提交内容（合并 4 张图片为一个文件，允许是图片或 PDF）
1. **作业 1**：`JwtService` 服务类的代码截图；
2. **作业 3**：`AuthMiddleware` 中间件代码截图；
3. **作业 3**：控制器 `User` 的 `info` 方法代码截图；
4. **作业 3**：Apifox 测试 `info` 方法（当前登录学生/老师的信息；token 有误）的预览截图。

### 提交标识和评分标准
* **图片显眼位置写入学号**；
* **方法的注释中写入学号**。

---

# 第 33 页：结束

**END**