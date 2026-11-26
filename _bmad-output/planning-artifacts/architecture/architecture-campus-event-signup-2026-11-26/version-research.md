# 版本、脚手架与技术依据核查

核查日：2026-11-26（Asia/Shanghai）。本文记录公开官方文档、发布元数据和脚手架内容的核查。**未安装项目依赖，未编译应用，未运行数据库或集成测试。**原始字段摘要在 [source-versions.json](source-versions.json)。版本是架构初稿的种子，后续构建以提交的 POM、Wrapper 和 package-lock.json 为准；发现不兼容时记录调整，不能以本次核查代替实测。

## 后端

| 核查对象 | 实际结果 | 直接来源 |
|---|---|---|
| Spring Initializr 元数据 | 本次默认稳定 Boot 为 4.2.0，列有 Java 21；没有沿用训练记忆中的旧默认版本 | [实时 metadata/client](https://start.spring.io/metadata/client) |
| Initializr 生成的 ZIP | 用 Boot 4.2.0、Java 21、Maven 及 web/security/jdbc/mysql/flyway/validation 生成至临时目录；POM 的 MVC starter 实际为 spring-boot-starter-webmvc，并含 flyway-mysql 与对应测试 starter | [官方生成端点](https://start.spring.io/starter.zip?type=maven-project&language=java&bootVersion=4.2.0&baseDir=campus-event-signup&groupId=io.github.rayalways&artifactId=campus-event-signup&javaVersion=21&dependencies=web,security,jdbc,mysql,flyway,validation) |
| Maven Wrapper | 生成文件记录 Wrapper 3.3.4，Maven 分发地址为 3.10.1 | 同上生成 ZIP；原始属性见 JSON 摘要 |
| Boot 4.2.0 BOM | 实查属性：Framework 7.1.0、Security 7.2.0、Flyway 12.11.0、Connector/J 26.10.0；由父 BOM 统一管理，不逐个猜测覆盖 | [Maven Central 官方发布 POM](https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-dependencies/4.2.0/spring-boot-dependencies-4.2.0.pom) |
| Java 兼容性 | Boot 4.2.0 文档要求至少 Java 17，Java 21 在其兼容区间内；21 为 LTS。具体 JDK 发行版、补丁和安装仍待环境故事完成 | [Boot 系统要求](https://docs.spring.io/spring-boot/system-requirements.html)、[Java 路线图](https://www.oracle.com/java/technologies/java-se-support-roadmap.html) |
| MySQL 目标补丁 | 采用当前 8.4 系列已公布的 8.4.13.1；发布说明日期为 2026-11-24。安装包或容器供应尚未验证，不承诺某个镜像标签已存在 | [8.4.13.1 发布说明](https://dev.mysql.com/doc/relnotes/mysql/8.4/en/news-8-4-13-1.html) |
| Connector/J 兼容范围 | 26.10 官方说明支持 MySQL 8.4 及以上、JRE 8 及以上；本轮仍须在选定组合上实际连接验证 | [Connector/J 兼容说明](https://dev.mysql.com/doc/connector-j/en/connector-j-versions.html) |

本次只检查 ZIP 内的 POM 和 Wrapper 属性，没有把第三方生成的 Java 代码当作本项目实现提交。

## 前端

| 核查对象 | 实际结果 | 直接来源 |
|---|---|---|
| 官方生成器 | create-vue 3.24.0；官方快速开始采用 create-vue/Vite，生成器要求 Node ^22.18.0 或 ≥24.12.0 | [Vue 快速开始](https://vuejs.org/guide/quick-start.html)、[create-vue 3.24.0 发布元数据](https://registry.npmjs.org/create-vue/3.24.0) |
| 归档模板 | 只读 npm 官方包归档内 package.json：base 为 Vue ^3.5.42、Vite ^8.2.2、plugin-vue ^6.0.8；Router 模板为 ^5.3.1；Vitest 模板为 ^4.1.11 | 上述 npm 元数据的 dist.tarball；对应字段已摘要保存 |
| 本轮固定建议 | Vue 3.5.46、Vue Router 5.5.1、Vite 8.3.8，均在读取的模板范围内；plugin-vue 固定模板已核查值 6.0.8 | [Vue 元数据](https://registry.npmjs.org/vue/3.5.46)、[Router 元数据](https://registry.npmjs.org/vue-router/5.5.1)、[Vite 元数据](https://registry.npmjs.org/vite/8.3.8)、[plugin-vue 元数据](https://registry.npmjs.org/@vitejs/plugin-vue/6.0.8) |
| Router 可选 peer | 补查 5.5.1 的 peerDependenciesMeta，vite、pinia、@pinia/colada、@vue/compiler-sfc 均标为 optional；原始字段已补入 JSON，当前不要求引入 Pinia | [Router 5.5.1 精确版本元数据](https://registry.npmjs.org/vue-router/5.5.1) |
| 测试工具 | Vitest 4.1.11 的 peer 范围包含 Vite 8；没有因为 latest 元数据出现 5.x 就跨模板主版本升级 | [Vitest 4.1.11 元数据](https://registry.npmjs.org/vitest/4.1.11) |
| 本机工具 | node --version 返回 v22.22.2；npm --version 返回 10.9.7，符合生成器和所选 Vite/plugin/Vitest 的 Node 范围 | 本次只读本机命令；版本范围见已保存元数据 |

前端先采用 JavaScript 是技术建议，尚未推定组员熟悉 TypeScript。未选择的 TypeScript 或 latest Vitest 元数据仅是查询原始记录，不代表已加入项目依赖。完整依赖解析与 npm ci 构建在脚手架故事中完成。

## 机制依据与设计推论

- [InnoDB 锁定读](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html)说明锁在事务中保护所读记录；以活动根为串行化边界是本项目据此作出的设计选择，文档本身不会自动保证所有代码路径遵守该边界。
- [MySQL 时间函数](https://dev.mysql.com/doc/refman/8.4/en/date-and-time-functions.html)的取时语义促使本设计在锁查询返回后另发语句取时；等待前采样的时间不能作为锁后业务判断依据。
- [Spring 事务传播](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html)是共享事务与禁止独立提交的依据；是否确实使用同一连接仍需集成测试。
- [Spring Security CSRF](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html)与 [PasswordEncoder](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/password-encoder.html)支持所选保护机制；会话失效、令牌更新及资源权限必须由本项目配置和验证。

后续验证负责人为两位成员：环境故事记录实际安装、依赖解析及构建；身份故事验证 Session/CSRF；一致性故事验证事务、补录、重复请求和回滚。当前没有测试结果可报告。
