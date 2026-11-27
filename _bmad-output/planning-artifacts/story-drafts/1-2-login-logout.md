---
storyId: '1.2'
epic: 1
title: 登录并安全退出登录
status: draft
approvalStatus: pending
implementationStatus: not-started
created: 2026-11-27
updated: 2026-11-27
dependsOn: ['1.1']
requirements: [FR-01, FR-02, NFR-02, NFR-03]
architecture: [AD-1, AD-5, AD-7, AD-8]
additionalRequirements: [AR-06, AR-08, AR-10, AR-11, AR-12, AR-14, AR-15]
---

### Story 1.2: 登录并安全退出登录

As a 已经注册的同学，
I want 使用自己的用户名和密码登录，并在使用后退出，
So that 系统能够识别本人，且退出后的旧凭据不能继续访问需登录的能力。

本条为待审阅草稿，尚未批准、实施或测试。唯一前置故事为 1.1；通过当前身份接口和测试作用域受保护处理器即可独立验证会话，不依赖未来的活动、报名或管理接口。用户名和密码沿用 D-017 已确认的处理方式，不重新提议另一套标准化规则。

**Acceptance Criteria:**

#### AC-1.2-01 正确凭据建立可用会话

**Given** 用户通过 Story 1.1 注册，当前未登录，并取得与当前匿名会话对应的 CSRF 令牌，
**When** 通过登录页面提交正确用户名和原密码，
**Then** `POST /api/auth/login` 返回 HTTP 200 和统一 JSON 成功结果，建立本人服务端会话，
**And** 用户名按已确认方式去首尾空白并转小写查找；密码按原值验证，不裁剪、转大小写或截断，
**And** 登录时轮换此前匿名会话标识；紧接着用当前 Cookie 请求 `/api/auth/me` 能返回同一用户的字符串 ID、用户名和昵称，不能只在登录那一个请求内看似成功，
**And** 前端显示本人登录状态，不靠 localStorage 里的“已登录”或自填 userId 宣布认证成功。

#### AC-1.2-02 错误凭据和无效输入有明确结果

**Given** 当前未登录且 CSRF 有效，
**When** 提交格式合法但不存在的用户名，或正确用户名配错误密码，
**Then** 均返回 HTTP 401 和相同的“用户名或密码错误”业务提示，不暴露哪个字段命中数据库，且不建立认证状态，
**And** 用户名不符合 D-017 格式、必填项缺失/为空白，或密码 UTF-8 超过 72 字节时返回 HTTP 400；密码值不因登录而改变，普通非空错误密码返回 401，不把密码校验异常变为成功，
**And** 页面显示可理解的失败原因，不返回默认 HTML 登录页、302 跳转或 SQL/堆栈详情。

#### AC-1.2-03 刷新页面以服务端身份恢复状态

**Given** 用户已经登录，
**When** 刷新页面或重新进入前端应用，
**Then** 前端通过 `/api/auth/me` 确认当前身份，当前会话有效则显示本人信息，
**And** 未登录、失效 Cookie 或另一个匿名客户端请求该接口时返回 HTTP 401 JSON，不从浏览器历史或本地缓存恢复已失效权限，
**And** 身份响应不包含密码、密码编码、其他用户资料或可由前端自行授予的全局组织者/管理员权限；当前身份、登录及退出响应设置 `Cache-Control: no-store`。

#### AC-1.2-04 登录和退出均保留 CSRF 保护

**Given** 页面使用同源 `/api` 及服务端会话，
**When** 提交登录或退出请求，
**Then** 缺少/伪造 CSRF 令牌的写请求返回 HTTP 403 JSON，不建立登录或执行退出，
**And** 登录成功或退出后重新通过 `/api/auth/csrf` 获取当前令牌，之后合法的写流程仍可继续，
**And** 使用旧令牌配新的当前会话写入时被拒绝，不能通过关闭 CSRF、改用 GET 执行退出或放开任意来源来规避联调问题。

#### AC-1.2-05 退出使旧会话失效

**Given** 用户持有有效登录 Cookie 和当前 CSRF 令牌，
**When** 提交 `POST /api/auth/logout`，
**Then** 返回 HTTP 200 JSON，服务端注销会话与认证上下文，客户端清除登录态并提示已退出，
**And** 保存并重用退出前 Cookie 请求 `/api/auth/me` 仍为 HTTP 401；不能只删除浏览器中的显示标记，
**And** 旧 Cookie 对测试作用域受保护写路径的请求被拒绝（按实际触发条件为 401/403），其处理器未执行；另以有效匿名 CSRF 请求验证身份校验本身返回 401，
**And** 该受保护处理器只存在测试代码，不随生产应用交付，不以创建未来活动功能来证明本故事完成。

#### AC-1.2-06 过期与正常重启后可重新登录

**Given** 账号已持久化且用户曾登录，
**When** 会话按测试配置过期，或在当前内存会话方案下正常重启后端，
**Then** 旧会话不再被识别为本人，前端收到 401 后清理失效登录显示并提示重新登录，
**And** 原账号和密码编码不变，重新获取 CSRF 后可以用原凭据再次登录，
**And** 不把业务数据持久化误解为必须永久保存登录会话，也不新增“记住我”或自动续登功能。生产会话时长在运行配置中明确，测试使用可控短时长，不伪造实测结果。

#### AC-1.2-07 登录退出不泄露凭据或损坏账号

**Given** 测试了正确登录、失败登录、CSRF 失败、退出及过期路径，
**When** 核对请求处理结果、日志、Cookie 和数据库，
**Then** 响应不含密码或密码编码，日志不含原密码、会话标识或 CSRF 令牌，身份数据不被缓存为可继续授权的依据，
**And** 会话 Cookie 继续满足 HttpOnly、SameSite=Lax，HTTPS 时 Secure；前端不保存可供脚本读取的会话令牌，
**And** 登录、退出和失败请求不修改原用户名、昵称或密码编码，不为本故事创建活动、报名、候补或通知业务表。

#### AC-1.2-08 独立验证并保留证据

**Given** Story 1.1 已实施，本故事可以只使用账号数据运行，
**When** 执行真实账号/数据库、Spring Security 过滤器和会话链路的集成验证，并从浏览器操作页面，
**Then** 保存登录成功/失败、跨请求身份恢复、匿名拒绝、CSRF 更新、旧 Cookie 拒绝、过期/重启再登录的实际结果，
**And** 不能仅依赖手工伪造“已认证用户”的测试注解代替登录/退出链路；受保护测试处理器在交付物中不存在，
**And** 注册功能回归通过，没有依赖尚未创建的 Story 1.3 或后续史诗；本故事不代表未来活动资源权限已全部验证。

#### 接口及实施建议（待本故事审阅）

| 接口 | 建议约定 |
|---|---|
| GET /api/auth/csrf | 沿用 1.1 的令牌获取流程；登录与退出后重新获取 |
| POST /api/auth/login | 表单编码的 username/password，带 CSRF Header；成功 200、错误凭据 401、字段错误 400、CSRF 错误 403；均为 JSON |
| GET /api/auth/me | 从服务端会话返回本人 ID/用户名/昵称；有效 200，未登录或失效 401；不能由请求 userId 决定本人 |
| POST /api/auth/logout | 合法 CSRF 请求成功时返回 200 JSON并注销当前会话；不以 GET 执行退出 |

本草稿不为“已退出后再次点击退出”强加新的业务承诺：前端以服务端身份和有效 CSRF 状态恢复界面，不将旧令牌拒绝误报成仍在登录。精确字段、错误码、Cookie 配置和测试路径须在实施前纳入单一接口契约。

框架依据已核对 [会话持久与身份恢复](https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html)、[退出处理](https://docs.spring.io/spring-security/reference/servlet/authentication/logout.html)、[CSRF](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html)。若自定义认证入口，必须正确保存认证上下文，不能只设置当前线程；本项目要求的 JSON 成功/失败响应也需要明确配置，不能把框架默认页面或跳转当作最终接口。

**边界：**本故事只交付登录、当前身份与退出；不包含找回密码、修改资料、第三方登录、多端会话控制、活动管理或报名。以上 AC 为待审阅规范，不是已经执行的测试结果。

来源：[PRD FR-01/FR-02 与 NFR-02/NFR-03](../prds/prd-campus-event-signup-2026-11-25/prd.md)、[正式 Story 1.1](../epics.md#story-11-通过网页注册账号)、[架构 AD-5/AD-7/AD-8](../architecture/architecture-campus-event-signup-2026-11-26/ARCHITECTURE-SPINE.md)。
