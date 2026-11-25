# BMAD 配置与使用记录

配置日期：2026 年 11 月 25 日。

## 安装结果

项目已安装官方 `bmad-method@6.12.1`，包括 `core` 和 `bmm` 两个模块，并生成 29 个 Codex 项目技能。该版本与课件采用的版本一致。安装器提示已有 6.12.4，本次为保持课程流程及两人环境一致，继续固定 6.12.1；没有使用 v7 预览版。

| 项目 | 配置 |
|---|---|
| 工具 | Codex，技能目录为 `.agents/skills/` |
| 核心方法 | `core@6.12.1`、`bmm@6.12.1` |
| 项目名称 | `campus-event-signup` |
| 对话语言 | 中文，团队默认值写入 `_bmad/custom/config.toml` |
| 文档语言 | 中文，写入安装器管理的共享配置 |
| 规划产物 | `_bmad-output/planning-artifacts/` |
| 实施产物 | `_bmad-output/implementation-artifacts/` |
| 项目知识 | `docs/` |
| 兼容 shim | 不安装；以本次安装生成的实际技能名为准 |
| 可选模块 | 未安装 TEA、bmb 等扩展 |

本次安装使用环境为 Node.js 22.22.2、npm 10.9.7、Git 2.54.0.windows.1、Python 3.13.15、uv 0.12.13。复现时以官方版本要求为准，不要求组员的补丁版本完全相同。

## 复现安装

在项目根目录执行以下 PowerShell 命令；命令只指定本次使用官方 npm 源，不更改全局 npm 设置：

```powershell
npx --yes --registry=https://registry.npmjs.org bmad-method@6.12.1 install --directory . --modules bmm --tools codex --communication-language Chinese --document-output-language Chinese --output-folder _bmad-output --set core.project_name=campus-event-signup --no-shims --yes
```

首次在本项目实际执行时还设置了 `--user-name RayAlways` 和 `--set bmm.user_skill_level=intermediate`。这些是个人偏好，不要求组员沿用同一个用户名。组员可加上自己的 `--user-name`，或接受系统用户名；Git 提交身份应另行配置为本人账号。

安装器要求 Node.js 20.12 或以上。脚本解析器要求 Python 3.11 或以上；`uv` 用于执行脚本、选择满足要求的解释器。需要 Python 依赖或解释器时，首次运行可能需要网络。

相关官方说明：[v6.12 安装指南](https://github.com/bmad-code-org/BMAD-METHOD/blob/V6.12/docs/start/install-bmad.md)。不要直接照抄 main 分支的 v7 预览安装方式。

## 配置的共享范围

- `_bmad/config.toml`、`_bmad/_config/`：安装器管理，提交到仓库，不直接修改生成内容。
- `_bmad/custom/config.toml`：团队覆盖配置，提交到仓库，安装器不会覆盖。
- `_bmad/config.user.toml`、`_bmad/custom/*.user.toml`：个人配置，忽略提交。克隆后即使没有个人配置，团队默认中文仍应生效。
- `_bmad/render/`：运行时渲染结果，由其目录内 `.gitignore` 排除，仅保留忽略规则。
- `.agents/skills/`、`_bmad/scripts/`：实际技能与运行脚本，提交到仓库，便于两人使用同一套流程。

运行环境及下载缓存不属于课程产物，不提交到仓库。

## 配置验证

从项目根目录执行：

```powershell
$bmadProjectRoot = (Get-Location).Path
uv run _bmad/scripts/resolve_config.py --project-root $bmadProjectRoot --key core.project_name --key core.communication_language --key core.document_output_language --key modules.bmm.planning_artifacts --key modules.bmm.implementation_artifacts
```

期望项目名为 `campus-event-signup`，对话和文档语言均为 `Chinese`。规划与实施路径含 `{project-root}` 占位符，由使用它的工作流替换为当前项目根目录。本次受限环境对相对路径的解析出现偏差，验证命令改为传入绝对项目路径，未修改官方脚本。

实际验证结果见下节，不把安装器退出成功等同于工作流全部完成。

## 验证记录

| 检查 | 实际结果 |
|---|---|
| 安装器退出与版本 | 成功退出；core、bmm 均为 6.12.1 |
| Codex 技能入口 | 29 个，全部存在 SKILL.md，与清单中的技能名称一致 |
| 配置解析 | 项目名正确，对话和文档均为中文 |
| PRD 技能配置 | 模板与输出目录配置可正常解析，未启动 PRD 创作 |
| 不带个人配置的克隆模拟 | 从 Git 暂存区导出的副本不含个人配置，仍可解析团队中文设置 |
| bmad-build 渲染检查 | 副本内官方渲染脚本成功生成 workflow.md；只验证加载，没有执行开发工作流 |
| 安装时的产品简报保留检查 | 配置完成时与安装前存档逐字节一致；后续两种报名模式的需求修订另见决策日志 D-003 |
| bmad-help 方向检查 | 已有产品简报仍为初稿；下一必经规划步骤是 bmad-prd，此前可用 bmad-product-brief 完善简报 |

受限环境下，渲染脚本曾返回 Windows 拒绝访问；同一脚本在标准权限环境通过，未修改官方代码。新开 Codex 项目聊天后的技能自动发现尚未验证。

安装器会在技能复制到 `.agents/skills/` 后清理 `_bmad/` 中的重复技能源目录。`skill-manifest.csv` 的 path 保留源布局，验证时按清单中的技能名检查实际工具入口，不把这些源路径当成运行入口。

原始安装输出见 [安装记录](evidence/bmad-install.txt)，检查摘要见 [验证记录](evidence/bmad-verification.json)。第三方工具许可保存在 [BMAD MIT License](third-party/BMAD-LICENSE.txt)。

## 在 Codex 中使用

在 Codex 中打开本仓库根目录 `campus-event-signup`，在该目录的聊天里使用项目技能。当前配置过程通过读取安装后的 `bmad-help` 技能、解析配置并检查目录进行验证；新开项目聊天后的技能菜单是否显示，需要在该项目上下文中确认。

可以给 Codex 以下指令：

> 使用 bmad-help，读取现有产品简报与作业核对表，说明当前阶段和下一步。不要把简报初稿当作已经完成真实用户验证。

本次实际安装中的主要入口为：

| 技能 | 用途 |
|---|---|
| `bmad-help` | 根据配置、已有产物和状态说明下一步 |
| `bmad-product-brief` | 通过引导完善产品简报；不同于课件中的旧入口拼写 |
| `bmad-prd` | 创建、更新或验证 PRD |
| `bmad-architecture` | PRD 或规格形成后比较方案并记录架构决策 |
| `bmad-create-epics-and-stories` | 在架构之后拆分史诗与故事 |
| `bmad-sprint-planning` | 检查就绪状态并形成冲刺跟踪 |
| `bmad-build`、`bmad-code-review` | 实施与评审 |
| `bmad-retrospective` | 结合实际过程证据复盘 |

产品简报仍为初稿，安装之后已使用 `bmad-product-brief` 的 Update 流程增加发起者审核模式，并记录对应的 memlog 与 D-003。随后已启动 `bmad-prd` 快速初稿，并补充 D-004 至 D-006；具体入口见 README。PRD 假设、真实使用场景和组员分工仍待审阅或补充；架构、故事和软件尚未实施。

课程特有要求继续以作业核对表为准。若 BMAD 默认输出路径或格式与课程命名不同，应在 README 中提供明确入口；课程要求的 `project-context.md`、根目录决策日志及复盘不能因工具默认行为不同而省略。
