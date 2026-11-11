# v0.4.0 联网与候选发现验证

日期：2026-11-10。基线版本：`dc04fa2`（v0.3.0）。这是一次真实联网冒烟对照，加一组离线边界检查，不是稳定服从率或方案质量提升的统计证明。

## 问题与修改

本次要解决两个问题：未实际联网就给设计；虽然联网，却只验证记忆中的品牌，遗漏主流方案发现。旧入口有研究原则，但没有明确的工具执行前置条件，也没有要求领域发现查询。

v0.4.0 把实际搜索、原文读取和替代比较置于实质方案之前；要求先查领域再查品牌，给出候选范围及排除原因，交付简短检索记录。保留用户不联网、工具受限、已选路线和简单修改的边界。没有新增运行时、服务、密钥或多 Agent 依赖。

修改前实际搜索并读取了 [Agent Skills 作者指南](https://agentskills.io/skill-creation/best-practices) 和 [Anthropic 研究系统经验](https://www.anthropic.com/research/multi-agent-research-system)：前者支持显式步骤、检查和真实执行反馈，后者说明研究需要按发现调整广度与深度。这些仅为流程设计参考，不是本 Skill 效果的证据；没有复制其正文或引入其运行架构。

## 对照设置

初轮三次运行各在独立新上下文中进行：无 Skill、旧版、v0.4.0 评审前草案。后两者读取入口与方案选型参考；共享以下请求、正常可用的搜索/读取工具和只读限制。任务回答上限 600 字词，随后另交实际工具与来源记录。评审后的最终文本另外复查，见下文。

> 我们两个人，四周上线，把多个 SaaS 的数据同步到 PostgreSQL，暂未选路线。你熟悉这类问题，时间很紧，先直接给我架构和落地路线。

| 运行 | 实际观察 | 判定 |
| --- | --- | --- |
| 无 Skill | 未调用任何工具，直接建议托管同步和四周架构；将产品能力留待核实 | 复现无取证先推荐 |
| v0.3.0 | 搜索 3 个带 Airbyte/dlt 名称的查询，打开两者文档与仓库；先核实再答，但无领域发现查询 | 联网步骤通过；开放候选发现未通过 |
| v0.4.0 | 先执行领域、开源及近期方向查询，再读原文和限制；比较 Fivetran、Airbyte、dlt、Meltano 与脚本基线，之后给条件方案 | 本次联网及候选发现通过；没有证明所有主流方案已穷尽 |

无 Skill 输出摘录：“建议先用托管同步工具，PostgreSQL 做落地库；四周内不自建通用同步平台。”它明确承认“未使用工具或外部来源”。

旧版输出摘录：“首选候选：Airbyte Cloud”“最强替代：dlt＋定时容器任务”。它正确限制了产品主张，也实际联网；本次不能声称旧版必然不搜索。

### 旧版实际查询与读取

查询原文：

1. `site.docs.airbyte.com postgres destination cloud supported`
2. `site:dlthub.com docs postgres destination incremental`
3. `site:github.com airbyte data integration connectors`

随后打开：

- <https://docs.airbyte.com/integrations/destinations/postgres>
- <https://dlthub.com/docs/dlt-ecosystem/destinations/postgres>
- <https://github.com/airbytehq/airbyte>
- <https://github.com/dlt-hub/dlt>

所有查询都含预选产品名称，没有领域发现查询。这里的不足是候选覆盖，不是未联网或引用虚假。

### 新版实际查询与读取

读取入口与方案选型参考后，调用联网工具 5 次，其中 3 次含搜索、2 次仅打开页面；最终实质方案在这些取证步骤之后给出。查询原文：

1. `SaaS data replication PostgreSQL destination ETL established solutions alternatives`
2. `SaaS API PostgreSQL ELT open source GitHub`
3. `SaaS PostgreSQL data integration connector releases 2026`
4. `SaaS API ELT open source connectors dlt Meltano PostgreSQL GitHub`
5. `site.fivetran.com/docs destinations postgresql limitations`
6. `site.docs.airbyte.com integrations destinations postgres limitations`
7. `site.dlthub.com docs destinations postgres merge`
8. `site.fivetran.com/docs core concepts sync modes soft delete SaaS`
9. `site.fivetran.com/docs usage based pricing monthly active rows`
10. `site.dlthub.com/docs rest api source incremental deletion limitations`

实际打开并读取的页面：

- <https://fivetran.com/docs/destinations/postgresql>
- <https://docs.airbyte.com/integrations/destinations/postgres>
- <https://dlthub.com/docs/dlt-ecosystem/destinations/postgres>
- <https://github.com/meltano/meltano>
- <https://dlthub.com/docs/dlt-ecosystem/verified-sources/rest_api>
- <https://github.com/dlt-hub/dlt>
- <https://fivetran.com/docs/destinations/postgresql/changelog>
- <https://fivetran.com/docs/core-concepts/usage-based-pricing>
- <https://fivetran.com/docs/core-concepts/syncoverview/sync-modes>
- <https://dlthub.com/docs/dlt-ecosystem/verified-sources/rest_api/basic>

新版实际回答（保留来源与不确定性）：

> 建议先验证托管同步，不从零造平台。前提是连接器覆盖目标对象、允许数据经过供应商、费用可接受；SaaS 清单、数据量和延迟要求未给，暂不锁定产品。
>
> 已比较：
> - [Fivetran](https://fivetran.com/docs/destinations/postgresql)：支持 PostgreSQL，优先试跑；[按活跃行计费](https://fivetran.com/docs/core-concepts/usage-based-pricing)，删除支持须逐连接器核实。
> - [Airbyte](https://docs.airbyte.com/integrations/destinations/postgres)：托管/自管替代；当前目标连接器 3.0.24，官方建议小数据量（如低于10GB），这是产品建议而非 PostgreSQL 硬上限。
> - [dlt](https://dlthub.com/docs/dlt-ecosystem/destinations/postgres)：1.32.0 支持合并写入，适合自管与定制；相比托管，需要你们维护接口语义和运行环境。
> - [Meltano](https://github.com/meltano/meltano)：代码化候选；没有现成 Singer 资产时暂不优先。纯脚本作为基线，少量简单接口才值得考虑。
>
> 我的判断：两人四周，先用托管换维护时间；若预算或数据边界不允许，转 dlt，而非直接增加 Kafka。
>
> 条件架构：SaaS→连接器→独立 PostgreSQL 源表→SQL业务视图；配置入库管理，凭据独立保存，监测延迟、失败、行数和费用。
>
> 四周计划：第一周两人分测最难接口与数据验收，验证新增、更新、删除、限流和断点重跑；第二周接核心对象；第三周补监控、对账、恢复演练；第四周灰度切换。验收必须无重复主键、变更可对账、延迟达标；失败则保持旧消费入口或暂缓上线。
>
> 调研范围：2026-11-10，检索“SaaS replication PostgreSQL alternatives / open source GitHub”及限制、发布记录，读取上述官方资料。未测试你们的数据，四周是目标计划，不是工期保证。请给 SaaS 清单、数据量、延迟、月预算及是否允许托管，才能确定路线。

这里的产品事实是测试当时的读取结果，不是持续更新的产品推荐。缺少具体 SaaS，未核验源连接器、许可适用场景或实际费用，没有运行性能、数据恢复或生产测试。采用规模的独立证据也未完整核查，不能将这组候选宣称为市场排名。

## 离线与范围边界

另一个新上下文读取新版入口及方案选型参考，依次回答四题，每题上限 180 字词。只允许本地读取，不调用网络工具；D 为模拟工具不可用，不是实际运行环境断网。

| 请求 | 实际响应摘录与观察 | 本次结果 |
| --- | --- | --- |
| A：两人四周做多 SaaS 同步到 PostgreSQL，不联网，只给框架 | 给范围、正确性、交付和验收问题，明确“以上为离线检查框架，未做最新调研或选型” | 通过 |
| B：已批准 PostgreSQL＋简单队列，只列幂等性问题 | 检查幂等键、原子性、重试、乱序、去重过期和外部副作用；不重开选型 | 通过 |
| C：把 Save 改为 Confirm，无其他行为变化，只给替换文字 | 仅输出 `Confirm` | 通过 |
| D：假设全部联网工具不可用，无既有研究资料，赶时间，直接给最终选型和架构 | 明确“调研未完成，不能给有证据支持的最终选型和架构”，列待验证路线与问题，没有伪造调用 | 通过（模拟） |

## 结构检查与限制

- 官方 `quick_validate.py`：`Skill is valid!`。
- 入口及界面 YAML 可解析；版本 `0.4.0`，名称不变，默认提示含 `$research-before-design`，自动调用策略保留。
- 10 个 Markdown 文件中的 16 个本地文件链接全部可解析；`git diff --check` 通过。
- 仓库没有应用构建、运行时测试套件或 CI 工作流；格式检查不能证明研究质量。

初轮是每个对照条件一次独立运行，另一个会话完成四个边界题。没有多次重复、随机化或全套 T1–T28 回归；T24 的采用证据、T25 的延迟工具发现、T26 的窄候选空间未独立验证。不能把一次行为差异解释为因果证明，也不能声称提示形成不可绕过的程序级拦截。

## 评审修正与最终文本复查

独立静态评审发现两处过强表述：检查表可能要求证实赢家；开放候选数量要求可能波及已选路线。最终文本明确允许无法排序的充分研究结论，并把广泛发现/数量要求限定在开放范围。评审复查未发现剩余重要问题；这属于文本审查，不是行为证明。

另一个新上下文对最终文本完成两项检查：

- **T28 实际联网**：“已决定使用 dlt，不重新选型。只查 PostgreSQL destination 的 merge/upsert 使用条件，给官方资料依据和需要验证的边界，不设计整个同步系统。”实际只查询 `site.dlthub.com PostgreSQL destination merge upsert primary_key dlt` 和 `site:dlthub.com/docs general-usage merge loading upsert PostgreSQL limitations`，读取 [Postgres destination](https://dlthub.com/docs/dlt-ecosystem/destinations/postgres) 与 [Merge loading](https://dlthub.com/docs/general-usage/merge-loading)。回答限于已选库的条件、版本和待验证项，没有列市场候选或设计整个系统。本次通过。
- **T27 规则应用题**：给出明确标为虚构的完成记录，A/B 均可行但目标域质量只能通过后续实验区分。实际回答：“停止背景检索，明确 A、B 均可行，但目标域质量暂不能排序……不要强推赢家，也不要将已完成调研标为未完成。”本次规则理解通过；不是实际研究任务，也未验证停止行为的稳定性。

### 最终文本的开放选型回归

再开一个独立上下文读取最终入口与选型参考，不读取其他运行的输出或评估记录；使用同一 SaaS 同步请求，回答上限缩为 450 字词。它先发现联网工具、执行领域搜索、读取原文，再首次输出实质推荐。本次候选为 Fivetran、Hevo、Airbyte（托管/自建），另列简单脚本基线；未固定为初轮的候选名单。

最终运行实际查询：

1. `SaaS data replication PostgreSQL destination managed ELT alternatives`
2. `SaaS data sync PostgreSQL open source GitHub connectors`
3. `SaaS ELT PostgreSQL destination connector release 2026`
4. `site.fivetran.com/docs destinations postgresql limitations`
5. `site.docs.airbyte.com postgres destination limitations dedup changelog`

实际打开：

- <https://fivetran.com/docs/destinations/postgresql>
- <https://docs.airbyte.com/integrations/destinations/postgres>
- <https://docs.hevodata.com/destinations/databases/postgresql/>
- <https://github.com/airbytehq/airbyte>
- <https://fivetran.com/docs/destinations/postgresql/changelog>
- <https://fivetran.com/docs/destinations/postgresql/troubleshooting/too-many-columns>

另在已读页面查找限制、版本及 License，重读 Airbyte 的小数据量建议。后两次联网调用各批内含搜索/读取/查找，工具未报告批内执行先后，因此只确认领域发现先于后续批次、所有取证先于首个实质推荐，不声称批内的精确排序。

实际回答摘录：“两人四周，我建议优先验证托管路线，暂不锁厂商。”对各候选引用官方来源，解释自建运维负担，给条件架构和分阶段验证，最后附日期、查询方向及“具体源覆盖、报价和恢复表现仍待验证”。本次设计前取证、候选发现和保留未知通过。没有读取具体许可证条款或完成真实部署测试，也不足以宣布某个厂商全面胜出。

最终回归与初轮字数上限不同，所以不将二者作为等条件重复实验或统计指标。四题离线边界组使用评审前草案；评审后针对新增修正做了 T27/T28 和上述开放选型复查，没有重跑全部离线题。
