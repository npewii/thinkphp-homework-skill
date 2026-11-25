# 第 1 页：封面

* **课程编号**：26-27-01
* **课程主题**：12 AI+VUE前端开发+登录验证

---

# 第 2 页：本次课内容

### AI 概述
* AI 工具使用技巧总结
* 通过 AI 工具认识 Vue
* Vue 小结

### 前端开发示范
* 登录验证开发流程
* 调试过程
* 可能的问题
* 测试
* 开发总结

---

# 第 3 页：AI 概述目录

1. AI 工具使用技巧总结
2. 通过 AI 工具认识 VUE
3. Vue 小结

---

# 第 4 页：AI 工具使用技巧（技巧一）

### 技巧一：先给上下文，再提要求
不要一上来就问“怎么写登录”。先告诉 AI：项目使用 Vue 3 + Vite + Pinia + Vue Router，UI 库是什么，后端返回的数据结构长什么样。上下文越具体，AI 的输出越可用。

* **A（反面示范）**：帮我开发一个登录验证的 vue 前端功能。
* **B（正面示范）**：项目是前后端分离开发，API 接口文档是……，后端使用的登录技术是……，token 返回的方式是……，前端要使用 VUE 开发，登录验证布局要求……，登录失败要求……，成功要求……，学生登录后……，老师登录后……

---

# 第 5 页：AI 工具使用技巧（技巧二）

### 技巧二：用“梳理”代替“生成”
当你手头已有接口文档或后端代码时，让 AI 先做梳理工作，而不是直接生成前端代码。例如：“请阅读以下 OpenAPI 文档，列出登录接口的请求方法、请求体字段、成功响应结构和失败响应结构。”这样你能先确认 AI 对接口的理解是否正确，再进入编码。

* **A（反面示范）**：登录验证的接口文档是……，请根据文档帮我生成前端的登录文件。
* **B（正面示范）**：登录验证的接口文档是……，请梳理请求和响应……；请根据梳理结果，完成编码。

---

# 第 6 页：AI 工具使用技巧（技巧三至六汇总）

* **技巧三：分阶段提问，不要一次性索要完整系统**
  一次性让 AI “生成一个带权限控制的登录系统”，结果往往是代码量巨大、结构混乱、难以调试。正确做法是按“接口理解 → 数据流设计 → 路由与守卫 → 页面组件 → 布局与菜单 → 调试”的顺序分步推进。
* **技巧四：要求 AI 解释“为什么”，而不仅仅是“是什么”**
  当 AI 给出路由守卫的写法时，追问一句：“如果用户已登录但角色不匹配，这段代码的跳转逻辑是什么？next 在哪些分支被调用、哪些分支没有被调用？”这能帮你真正理解代码，而不是盲目复制。
* **技巧五：利用 AI 做“错误预判”**
  在启动开发服务器之前，可以先问 AI：“按照这个配置，在 Vite + Vue Router 的组合下，跨域请求最可能在哪里出问题？我应该在浏览器哪个面板确认？”带着预判去调试，效率远高于盲目试错。
* **技巧六：让 AI 对照后端实现细节**
  前端登录验证的每一个判断（如“未登录跳登录页”、“角色不匹配拒绝访问”），都应能在后端找到对应依据。把后端登录验证的实现细节（如 token 生成方式、角色字段名、响应码约定）交给 AI，要求前端逻辑与后端严格对齐，避免“前端自己发明规则”。

---

# 第 7 页：AI 工具使用技巧（分步推进示例）

* **错误提问方式**：帮我生成所有的代码……
* **正确提问推进路径**：
  1. 梳理接口
  2. 流程分析
  3. **请列出文件清单**
  4. 路由守卫
  5. 页面组件
  6. 布局与菜单

---

# 第 8 页：AI 工具使用技巧（探究逻辑机制）

```ts
// 路由守卫
router.beforeEach((to, _from, next) => {
  const userStore = useUserStore()
  userStore.restore()

  if (to.meta.public) {
    if (userStore.isLoggedIn) return next(userStore.homePath)
    return next()
  }

  if (!userStore.isLoggedIn) {
    // 弹窗提示
    ElMessage.warning('您还未登录，请先登录！')
    // 跳转到登录页，并记录原本想去的地址 (to.fullPath)
    return next({ path: '/login', query: { redirect: to.fullPath } })
  }

  const requiredRole = to.meta.role as string | undefined
  if (requiredRole && userStore.urole !== requiredRole) {
    return next(userStore.homePath)
  }

  // 深入提问：这段代码什么时候运行，有什么效果？next() 是如何调度的？
  next()
})
```

---

# 第 9 页：AI 工具使用技巧（错误预判与后端对齐）

* **利用 AI 做“错误预判”**：带着针对性去排查控制台与网络面板。
* **对照后端实现细节**：将后端的 JWT 结构（如 `uid`, `urole: 'stu'|'te'`）与错误状态码原原本本同步给 AI，保证类型与分支判断严丝合缝。

---

# 第 10 页：AI 工具使用技巧（技巧七：复述认知）

### 技巧七：别总让 AI 回答问题，更要“用自己的话复述、让 AI 审视”
不要把 AI 当成只会答题的搜索引擎。正确做法是：先结合 AI 的回答形成自己的认知，然后用自己的语言把它输出出来，再把这个“自己的版本”交给 AI：“请审核我的理解是否有偏差、遗漏或表述不准确的地方。”
* **核心价值**：能用自己的语言讲出来，才是真正的掌握；让 AI 当“审稿人”，比让它当“答题人”更有价值。

---

# 第 11 页：通过 AI 工具认识 VUE

1. 环境搭建
2. Vue 文档结构
3. 组件与布局
4. 路由基础
5. 路由守卫
6. Pinia 状态管理
7. API 请求与拦截器
8. 跨域与部署

---

# 第 12 页：十个核心问题

* **问题一（环境搭建）**：使用 Vue 3 官方推荐的方式创建一个新项目，命令行应该输入什么？创建过程中会询问是否添加 Router、Pinia、ESLint 等选项，如果本项目需要路由、状态管理和接口请求，应分别选择哪些？创建完成后，`npm run dev` 启动的是什么服务器，默认端口是多少？
* **问题二（环境搭建与依赖）**：项目需要发送 HTTP 请求，Vue 官方推荐搭配哪个库？如何安装？在 `main.js` 中如何挂载？请说明为什么通常不直接在组件里写 `fetch`，而要用统一的请求封装（如 axios 实例 + 拦截器）。
* **问题三（Vue 文档结构）**：一个由 Vite 创建的 Vue 3 项目，`index.html`、`src/main.js`、`src/App.vue`、`src/router/index.js`、`src/stores/` 各自承担什么职责？`main.js` 中 `createApp(App).use(router).use(pinia).mount('#app')` 这一行链式调用，每一步分别在做什么？
* **问题四（组件与布局）**：Vue 3 中如何实现“登录页使用独立布局、登录后页面共享顶层导航栏”的效果？有哪几种常见方案（如嵌套路由 + 布局组件、`<router-view>` 多级嵌套、动态组件）？请比较它们的优缺点，并说明本项目应选哪种。
* **问题五（路由基础）**：Vue Router 中如何定义一条路由？`path`、`name`、`component`、`meta` 分别是什么含义？什么是嵌套路由，`children` 配置后父组件的 `<router-view>` 起什么作用？请结合“顶层导航栏 + 下方内容区”的布局说明。
* **问题六（路由守卫）**：Vue Router 的全局前置守卫 `router.beforeEach((to, from, next) => {...})` 在什么时候执行？`to`、`from`、`next` 各是什么？如何用 `meta` 字段标记“需要登录”的页面，并在守卫中拦截未登录访问？`next()`、`next('/login')`、`next(false)` 三种调用分别是什么效果？
* **问题七（路由守卫与角色）**：在路由守卫中如何实现“分角色访问”：学生只能进学生功能页，教师只能进教师功能页？应该在路由的 `meta` 中放什么信息，守卫中如何比较当前登录用户角色？如果已登录但角色不匹配，应该跳到哪里、如何给出提示？
* **问题八（Pinia 状态管理）**：Pinia 是什么，与 Vuex 有何区别？如何定义一个 store (`defineStore`)？`state`、`getters`、`actions` 分别对应什么？登录成功后，用户信息（token、角色、姓名）应放在 Pinia 的哪里？刷新页面后 Pinia 状态会丢失，通常如何配合 localStorage 持久化？
* **问题九（API 请求与拦截器）**：如何用 axios 创建一个实例，配置 baseURL、超时时间？请求拦截器里如何统一携带 token（如 Authorization 头）？响应拦截器里如何统一处理 code 非 200 的情况、如何把后端的 message 交给弹窗组件显示？请结合第九章响应格式 `{code, message, data}` 说明。
* **问题十（跨域与部署）**：开发阶段前端 `localhost:5173` 请求后端 `localhost:8000` 为什么会产生跨域？Vite 的 `server.proxy` 如何配置代理解决？上线后前后端部署在同一域名下（反向代理），为什么就不存在跨域问题了？请分别说明开发阶段与生产阶段的请求路径差异。

---

# 第 13 页：额外说明：每个问题都要“追问三轮 + 复述审核”

* **第一步，追问至少三个问题**：
  * **深挖机制**：追问底层实现（如：“`beforeEach` 是同步还是异步执行？如果守卫里发起了异步请求，`next` 应该什么时候调用？”）；
  * **追问边界**：追问异常场景（如：“如果 localStorage 里存了旧格式数据，页面加载时如何处理才不会报错？”）；
  * **对比辨析**：追问易混概念（如：“`next(false)` 和 `next('/login')` 在浏览器地址栏和历史记录上的表现有何不同？”）；
  * **落到本项目**：追问“这个知识点在我们这个登录验证项目里具体用在哪一步？”
* **第二步，用自己的语言描述认知**：合上 AI 答案，自己复述一遍，梳理逻辑。
* **第三步，让 AI 审核认知偏差**：输入复述内容，让 AI 逐条挑刺、补齐漏洞。

---

# 第 14 页：作业一

### [实验] 10-1：提问 AI，回顾 Vue（追问三轮 + 总结）
* **要求**：
  1. 针对十个问题，向 AI 提问并记录原始回答；
  2. 每个问题至少追问 3 个方向（机制、边界、对比、项目落地）；
  3. 自己写出认知总结并提交 AI 进行审查修正；
  4. 整理成一份规范的实验报告（含一级、二级标题）。
* **验收标准**：覆盖 10 个问题；追问记录完整；包含“自己总结”与“AI 审阅修正意见”；能口头解释关键机制。

---

# 第 15 页：Vue 小结——前端项目部署（版本与指令）

1. **Vue2 or Vue3？**：全面采用 Vue 3（Composition API + `<script setup>`）。
2. **安装命令？**：使用官方推荐工具链。

---

# 第 16 页：Vue 小结——前端项目部署（项目初始化）

### 初始化指令
```bash
npm create vue@latest
```

### 脚手架选项配置示范
* Project name: `2026vue`
* Add TypeScript? **Yes**
* Add JSX Support? **No**
* Add Vue Router for Single Page Application development? **Yes**
* Add Pinia for state management? **Yes**
* Add Vitest for Unit Testing? **No**
* Add an End-to-End Testing Solution? **No / Playwright**
* Add ESLint for code quality? **Yes**
* Add Prettier for code formatting? **Yes**

### 启动流程
```bash
cd 2026vue
npm install
npm run format
npm run dev
```

---

# 第 17 页：Vue 小结——Node.js 与 npm 认知

1. **为啥要装 Node.js？**：提供本地 JavaScript 运行环境，驱动 Vite 开发服务器、打包构建工具及管理 npm 模块。
2. **上线了还需要 Node.js 吗？**：纯前端静态项目打包（`npm run build`）后生成纯 HTML/JS/CSS，由 Nginx/Apache 托管，**线上服务器不需要安装 Node.js**。
3. **npm 是什么？**：Node Package Manager，Node.js 官方的包管理工具。

---

# 第 18 页：升级 Node.js

* 建议安装 Node.js **v22.x LTS** 长期支持版本（如 `v22.23.5 LTS`）。
* Windows 用户下载 `.msi` 安装包安装，并在配置中确认添加系统 PATH 环境变量。

---

# 第 19 页：升级 npm

### 升级命令
```bash
npm install -g npm@latest
```
* **注意**：
  * macOS / Linux 若报错需前置 `sudo`；
  * Windows 用户若升级后版本未变，检查环境变量中 `%appdata%\npm` 是否排在 `C:\Program Files\nodejs` 前面。

---

# 第 20 页：升级 npm 后安装 package.json 依赖

```bash
node -v   # 确认 node 版本 (如 v22.23.5)
npm -v    # 确认 npm 版本 (如 12.2.3)
npm install
```

---

# 第 21 页：其他依赖安装

```bash
# 核心依赖
npm install pinia pinia-plugin-persistedstate    # 状态持久化插件
npm install element-plus                        # Element Plus UI库
npm install @element-plus/icons-vue             # 独立图标库
npm install vue-router@4                        # 官方路由库
npm install axios                               # HTTP请求库

# 样式增强开发依赖
npm install -D sass-embedded
```

---

# 第 22 页：依赖安装示范

```bash
npm install element-plus
```
* 终端显示 `added ... packages, audited ... packages in ...s` 即为成功。

---

# 第 23 页：本地运行与打包区别提问

1. 如何启动一个本地开发服务器？
2. 如何打包？
3. 两者有何区别？

---

# 第 24 页：本地开发服务器运行效果

* 默认访问地址：`http://localhost:5173/`
* 展示 Vite + Vue 3 欢迎首页。

---

# 第 25 页：两者的核心区别（开发环境 vs 生产构建）

| 对比项 | `npm run dev` | `npm run build` |
| :--- | :--- | :--- |
| **主要用途** | 本地开发、联调调试 | 生产环境发布上线 |
| **代码状态** | 不压缩、保留 SourceMap、可读性强 | 深度压缩、变量混淆、分包优化 |
| **运行方式** | 启动一个常驻的 Node 本地开发服务器 | 执行一次性构建流程，完成后立即退出 |
| **热更新** | 支持 HMR（修改代码即时局部更新） | 不支持（纯静态输出） |
| **输出结果** | 编译结果置于内存，无实体文件 | 输出 `dist/` 实体文件目录 |
| **使用时机** | 日常开发编写代码时 | 开发完成、提测或部署上线前 |

---

# 第 26 页：前端开发示范目录

1. 登录验证开发流程
2. 调试过程
3. 可能的问题
4. 测试
5. 开发总结

---

# 第 27 页：登录认证界面——学生登录失败示范

* 页面展示登录卡片，支持“学生 / 教师”Tab 切换；
* 账号密码错误时，顶部滑出 Element Plus 错误弹窗：`登录认证失败`。

---

# 第 28 页：学生登录后的界面布局

* **顶层头部**：
  * 左侧：`Logo + 个人签名照` + 教学管理系统
  * 右侧：登录信息及注销选项（`曾家豪 (学生)`，`退出登录` 按钮）
* **二级导航栏**：
  * 个性化横向菜单：包含【查询教学任务】、【查询成绩】（学生专属视角）
* **内容展示区**：
  * 默认展示查询教学任务（学生）缺省状态。

---

# 第 29 页：老师登录界面示范

* Tab 选择【教师】；
* 输入教工号（如 `2005100046`）和密码提交。

---

# 第 30 页：老师登录后的界面布局

* **顶层头部**：
  * 登录成功反馈（绿色弹窗提示）；
  * 右侧展示：`王焕军 (教师)`，`退出登录` 按钮；
* **二级个性化导航栏**：
  * 显示教师专属三项功能菜单：【查询教学任务】、【查询成绩】、【成绩录入】；
* **内容区**：
  * 呈现带学期、录入状态筛选的过滤表单与重置按钮。

---

# 第 31 页：教学任务查询结果展示

* 教师端展示教学任务完整列表：
  * 字段：学年学期、授课教师、课程名称、教学班级、课程类型、考核方式、平时/实验/期中/期末权重；
  * 底部集成标准分页栏（显示总条数、页码切换、每页容量下拉选择）。

---

# 第 32 页：教师查询成绩界面

* 点击具体教学任务后，下钻展示班级学生详细成绩列表；
* 支持一键【返回任务列表】；
* 列表清晰罗列学生学号、姓名、平时、实验、期中、期末、总评、补考、重修成绩。

---

# 第 33 页：前端开发流程标准推进顺序

```text
[1. 喂文档] ──> [2. 梳理接口] ──> [3. 对齐后端] ──> [4. 定布局]
                                                       │
[8. 最终测试] <── [7. 联调调试] <── [6. 写代码] <── [5. 提需求]
```
> **原则**：每一步都对应一次目标明确、边界清晰的 AI 提问。

---

# 第 34 页：第一步：发送 OpenAPI 格式的 API 文档

* **规范**：将后端导出的 OpenAPI (Swagger) JSON 或 YAML 文档完整提交给 AI；
* **Prompt 指令**：“这是本项目的后端接口文档，请先不要写代码，只做阅读和确认。文档中应包含登录接口、学生查询个人成绩接口、教师查询教学任务接口、查询教学任务成绩接口、成绩录入接口。”

---

# 第 35 页：接口文档导出流程

* 在 Apifox 项目中：
  1. 选中接口分组目录；
  2. 点击【更多】→【导出】；
  3. 选择数据格式为 **OpenAPI Spec (3.0 / 3.1)**；
  4. 导出为 **JSON** 文件供 AI 解析。

---

# 第 36 页：第二步：让 AI 梳理 API 的请求和响应

* **梳理清单要点**：
  * 请求方法（GET/POST/PATCH 等）、路径、传参位置（Header/Query/Body）；
  * 登录请求体（用户名、密码）与响应凭证（token、角色字段、姓名）；
  * 业务接口是否统一为 `{code, message, data}` 结构；
  * 错误返回时提取 `message` 的字段规则。
* **人工审查死线**：核对字段名（如学号是 `uid` 还是 `stu_id`）与角色标识（是 `stu`/`te` 还是数字），**这一步错了后面全错**。

---

# 第 37 页：第三步：让 AI 了解后端登录验证的实现细节

* **约定规则同步**：
  * Token 传递规范：统一在请求头携带 `Authorization: Bearer <token>`；
  * 路由权限划分：
    * 学生角色：`"urole": "stu"`
    * 教师角色：`"urole": "te"`

---

# 第 38 页：第四步：提出布局要求

1. **登录页**：独立路由与纯净卡片布局，不共享任何系统顶栏；
2. **内部资源页**：统一共享顶层导航栏（Logo + 个人签名照 + 系统名称 + 用户信息 + 注销按钮）；
3. **二级菜单栏**：横向排布于顶栏下方，根据登录角色动态渲染菜单项。

---

# 第 39 页：第五步：提出前端登录验证的要求

* **权限约束**：
  * 未登录时阻断对所有内部路由的访问；
  * 角色隔离（学生不可越权访问教师教学任务，反之亦然）；
* **提问提示词**：“基于以上接口约定、布局要求和业务要求，请先给出路由结构设计（含嵌套关系与 meta 字段）和 Pinia store 设计，暂不写页面代码。”

---

# 第 40 页：第六步：分模块生成代码并逐块审查

### 推荐实现流水线
```text
[1. Axios 封装与拦截器]
       │
[2. Pinia 用户 Store (持久化)]
       │
[3. 路由定义 (基础路由 + 嵌套路由)]
       │
[4. 全局路由守卫]
       │
[5. Login.vue 登录页面]
       │
[6. 顶层导航栏与菜单布局组件]
       │
[7. 各业务占位与对接页面]
```

---

# 第 41 页：关键设计说明（路由结构）

```ts
// src/router/index.ts
import { createRouter, createWebHistory, type RouteRecordRaw } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { ElMessage } from 'element-plus'

const routes: RouteRecordRaw[] = [
  // 1. 登录页独立布局
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/Login.vue'),
    meta: { public: true },
  },
  // 2. 内部业务共享布局
  {
    path: '/',
    component: () => import('@/layouts/MainLayout.vue'),
    redirect: '/login',
    children: [
      // 内部子路由由子页面承接，并配置 meta: { requiresAuth: true, role: 'stu'|'te' }
    ],
  },
  // 3. 通配兜底重定向
  { path: '/:pathMatch(.*)*', redirect: '/login' },
]

const router = createRouter({
  history: createWebHistory('/'),
  routes,
})

export default router
```

---

# 第 42 页：关键设计说明（Pinia 用户 Store）

```ts
// src/stores/user.ts
import { defineStore } from 'pinia'
import type { UserRole } from '@/api/types'

interface UserState {
  uid: number | null
  uname: string
  urole: UserRole | ''
  token: string
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    uid: null,
    uname: '',
    urole: '',
    token: ''
  }),
  getters: {
    isLoggedIn: (state) => !!state.token,
    homePath: (state) => (state.urole === 'stu' ? '/student/score' : '/teacher/task')
  },
  actions: {
    setUserInfo(info: UserState) {
      this.uid = info.uid
      this.uname = info.uname
      this.urole = info.urole
      this.token = info.token
      localStorage.setItem('user_info', JSON.stringify(info))
    },
    logout() {
      this.uid = null
      this.uname = ''
      this.urole = ''
      this.token = ''
      localStorage.removeItem('user_info')
    },
    restore() {
      const data = localStorage.getItem('user_info')
      if (data) {
        try {
          const info = JSON.parse(data)
          this.uid = info.uid
          this.uname = info.uname
          this.urole = info.urole
          this.token = info.token
        } catch {
          this.logout()
        }
      }
    }
  }
})
```

---

# 第 43 页：关键设计说明（路由守卫与 next 行为）

### 表 10-1 守卫中 next 三种调用效果对比

| 调用方式 | 行为效果 | 在本项目中的具体应用分支 |
| :--- | :--- | :--- |
| `next()` | 放行当前导航，按既定目标正常进入路由 | 已登录且角色匹配，或目标页面为公共白名单页面 |
| `next('/login')` | 中断当前导航并**跳转至登录页**，地址栏改变且产生历史记录 | 未登录非法访问受保护页面时触发 |
| `next(false)` | 中断导航，**停留在当前页**，地址栏不变且不产生历史记录 | 已登录但角色不匹配，阻止越权访问对方业务页面 |

```ts
// src/router/index.ts (守卫逻辑)
router.beforeEach((to, _from, next) => {
  const userStore = useUserStore()
  userStore.restore()

  // 公共页面处理
  if (to.meta.public) {
    if (userStore.isLoggedIn) return next(userStore.homePath)
    return next()
  }

  // 未登录拦截
  if (!userStore.isLoggedIn) {
    ElMessage.warning('您还未登录，请先登录！')
    return next({ path: '/login', query: { redirect: to.fullPath } })
  }

  // 角色权限校验
  const requiredRole = to.meta.role as string | undefined
  if (requiredRole && userStore.urole !== requiredRole) {
    ElMessage.error('无权访问该角色页面！')
    return next(userStore.homePath)
  }

  next()
})
```

---

# 第 44 页：关键设计说明（Axios 响应拦截器）

```ts
// src/api/request.ts
import axios from 'axios'
import type { AxiosResponse } from 'axios'
import { useUserStore } from '@/stores/user'
import router from '@/router'
import { ElMessage } from 'element-plus'

const request = axios.create({
  baseURL: '/guirzv2',
  timeout: 5000,
})

// 请求拦截器：自动注入 Bearer Token
request.interceptors.request.use((config) => {
  const userStore = useUserStore()
  if (userStore.token) {
    config.headers.Authorization = `Bearer ${userStore.token}`
  }
  return config
})

// 响应拦截器：统一业务处理
request.interceptors.response.use(
  (response: AxiosResponse) => {
    const res = response.data
    // 业务 code 校验：非 200 抛出异常
    if (res.code && res.code !== 200) {
      ElMessage.error(res.message || '业务处理失败')
      return Promise.reject(new Error(res.message || '请求失败'))
    }
    return res
  },
  (error) => {
    // 处理 HTTP 协议层错误 (如 401 凭证失效)
    if (error.response?.status === 401) {
      const userStore = useUserStore()
      userStore.logout()
      router.push('/login')
      ElMessage.error('登录失效，请重新登录')
    } else {
      const errMsg = error.response?.data?.message || error.message || '网络异常'
      ElMessage.error(errMsg)
    }
    return Promise.reject(error)
  }
)

export default request
```

---

# 第 45 页：调试工具——浏览器 F12 的使用

* **Network（网络面板）**：
  * 勾选 `Preserve log`（保留日志）；
  * 观察登录请求与接口通信的 URL、Status Code、Request Headers、Payload 及 Response JSON。
* **Console（控制台面板）**：
  * 查看 Vue Router 守卫输出、Axios 拦截捕获的错误堆栈、语法警告。

---

# 第 46 页：可能的问题（502 Bad Gateway）

### 502 Bad Gateway 问题的本质原因与排查
* **产生机制**：在本地开发阶段，浏览器请求 `http://localhost:5173/guirzv2/...`（同源无跨域），由 Vite 开发服务器的反向代理（`server.proxy`）在后台将请求转发给后端服务（如 `localhost:8000`）；
* **故障定位**：若出现 **502 Bad Gateway**，说明 Vite 作为网关无法连接到上游后端服务器。
  * **原因一**：后端 PHP/Apache/Nginx 服务未启动；
  * **原因二**：`vite.config.ts` 中的代理目标端口（`target`）配置错误。

---

# 第 47 页：功能测试规范

1. **测试一：错误登录输入**
   * 提交错误账号/密码，验证 Network 收到失败响应，页面弹出后端返回的实际 `message`，输入框不崩溃。
2. **测试二：路由守卫阻断**
   * 清除本地 Token，直接在浏览器地址栏手动键入内部受保护路径（如 `/student/score`），应被守卫拦截、弹出警告并弹回 `/login`。
3. **测试三：分角色越权访问**
   * 登录学生账号，验证横向菜单仅展示“查询个人成绩”；手动在地址栏键入教师路径，应被守卫拦截并留在学生首页。
4. **测试四：注销清理**
   * 点击注销，验证 Pinia 响应式状态与 `localStorage` 缓存均被清空，重定向至登录页且无法“后退”回系统内部。

---

# 第 48 页：作业二

### [实验] 10-2：开发前端的登录验证功能
* **要求**：
  1. **接口梳理**：通过 AI 规范解析 OpenAPI 文档，明确字段定义与角色字典；
  2. **请求封装**：在 `src/api/request.ts` 中封装 Axios 实例、请求拦截注入 Token、响应拦截统一解包与错误弹窗；
  3. **状态管理**：在 `src/stores/user.ts` 中实现用户信息持久化与注销清理；
  4. **路由守卫**：在 `src/router/index.ts` 中配置公共路由、嵌套布局路由以及双重全局前置守卫（登录拦截 + 角色校验）；
  5. **界面布局**：完成登录页面与主布局组件开发（Logo + 个人签名照 + 动态横向菜单）；
  6. **联调调试**：借助 Network 与 Console 确认流程无误。
* **验收标准**：登录流程顺畅；错误提示与后端一致；越权访问成功拦截；注销彻底无残留；页面刷新状态不丢失。

---

# 第 49 页：作业 1~2 提交要求（第十次课）

### 提交形式
合并 **9 张截图** 或生成一份完整 PDF/Word 文档。

### 详细内容清单：
1. **作业 1**：与 AI 互动完成 10 个问题 + 各 3 个追问 + 各 1 个总结的记录与截图（含一级、二级标题）；
2. **作业 2**：`router/index.ts` + `auth.ts` 代码截图；
3. **作业 2**：`stores/user.ts` 代码截图；
4. **作业 2**：`api/request.ts` 代码截图；
5. **作业 2**：`views/Login.vue` 代码截图；
6. **作业 2 测试**：学生登录失败界面截图 + 学生登录成功后访问首页界面截图；
7. **作业 2 测试**：老师登录失败界面截图 + 老师登录成功后访问首页界面截图；
8. **作业 2 调试截图**：学生登录成功后，F12 Application 面板中查看持久化存储 Token 信息的截图；
9. **作业 2 调试截图**：老师登录成功后，F12 Network 面板打开 Headers、Payload 与 Response 的抓包截图。

### 评分标准与防伪要求：
* **Logo 和个人签名照是关键防伪标识**；
* 图片显眼位置须打上**个人学号**；代码方法注释中须写入**个人学号**。
