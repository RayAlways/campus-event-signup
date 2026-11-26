# 版本与事实核查评审

结论：**通过，附一项非阻断的证据补充建议。** 本轮架构所绑定的具名技术及版本均有官方来源或本机现实核查依据；未发现依靠训练记忆选版本、沿用过时脚手架默认值或把查询记录误当成已选依赖的情况。此结论针对架构种子与已有核查材料，不代表依赖安装、编译或集成测试通过。

评审日：2026-11-26。依据已安装 `bmad-architecture/customize.toml` 的第一个必跑 reviewer lens，检查 `ARCHITECTURE-SPINE.md`、`version-research.md`、`source-versions.json`，并读取已有的 `work/architecture-initializr-pom.xml` 及 `work/architecture-extra-check.json`。遵循本轮范围，不重复下载或访问 GitHub。

| 对象 | 现有证据与判断 |
|---|---|
| Spring Boot / Initializr | JSON 保存实时 `metadata/client` 的默认 `4.2.0.RELEASE`（显示名称 4.2.0）和 Java 21 选项；已有生成 POM 的 parent 为 4.2.0、`java.version` 为 21，MVC starter 为 `spring-boot-starter-webmvc`，包含 JDBC、Security、Validation、Flyway/MySQL 及相应测试 starter。实时默认与生成结果一致。 |
| Maven / Wrapper | `starter_check.wrapper` 保存生成属性：Wrapper 3.3.4、Maven 分发 URL 为 3.10.1。来源为本次官方 Initializr 生成 ZIP，不是独立猜测。 |
| Framework / Security / Connector/J / Flyway | Boot 4.2.0 官方发布 BOM 的字段摘要分别为 7.1.0 / 7.2.0 / 26.10.0 / 12.11.0，与 Stack 一致，并明确由 BOM 管理。 |
| Java 21 / MySQL 8.4.13.1 | 研究记录给出 Boot 系统要求、Java LTS 路线图、MySQL 8.4.13.1 官方发布说明与 Connector/J 兼容说明。Java 是编译目标；JDK 发行版补丁已明确留待环境故事。MySQL 仅声称官方发布，未声称镜像或安装包已验证存在。不存在把待办误报为已完成的问题。 |
| create-vue / Vue / Router | 官方 registry 记录生成器 3.24.0；归档模板摘录为 Vue `^3.5.42`、Router `^5.3.1`。选定 3.5.46 / 5.5.1 均在对应范围内，Router 的 Vue peer 范围包含 3.5.46。 |
| Vite / plugin-vue | 模板为 `^8.2.2` / `^6.0.8`，选定 8.3.8 / 6.0.8 均满足；plugin-vue 的 Vue、Vite peer 范围包含本组合。 |
| Vitest | 所选是归档模板 `^4.1.11` 中的 4.1.11；其精确版本官方 metadata 单独保存在 `bom_check.vitest`，Vite peer 包含 8.x。JSON 中 latest 5.1.2 不是项目选择，不能据此否定架构。 |
| Node.js / npm | 已有本机命令结果为 22.22.2 / 10.9.7；Node 22.22.2 同时满足生成器、Vite、plugin-vue、Vitest 保存的 engines 范围。没有要求所有软件本轮完成安装的理由。 |
| 机制与未选项 | 事务、锁定读、时间函数、CSRF 与密码编码均在研究记录中给出官方依据，并将项目规则与官方机制分开。TypeScript 7.0.2、Vitest latest 5.1.2 和其他未选模板片段只是查询记录，不构成新增的架构依赖。 |

唯一证据缺口（低优先级，补充即可）：`source-versions.json` 的 `registry_and_initializr.vue-router.peerDependencies` 列出 `pinia`、`@pinia/colada` 等 peer，但摘要未保留 `peerDependenciesMeta`。因此，现存摘录本身尚不能证明这些 peer 是否为 optional，而架构明确不增加 Pinia。建议在脚手架故事或下次证据整理时保存 **vue-router 5.5.1 的官方 `peerDependenciesMeta`**，或保留无 Pinia 方案的实际依赖解析结果。不据此认定组合不兼容，也不要求增加 Pinia。

处置：保留本轮版本建议。完整依赖锁定、MySQL 供应方式与实际构建/连接验证继续按文档已列明的环境及脚手架故事执行；这些已声明的后续工作不是本轮未披露的证据缺口。


父代理处理结果：已再次读取 https://registry.npmjs.org/vue-router/5.5.1 ，补入peerDependenciesMeta，确认Pinia、@pinia/colada等peer为optional；version-research.md同步记录。未安装依赖，未宣称运行兼容性。
