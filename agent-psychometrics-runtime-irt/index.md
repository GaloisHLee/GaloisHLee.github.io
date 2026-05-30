# 从Agent Psychometrics到任务难度感知Agent：IRT、LLM-as-Judge与Planner/Audit模式


> Reading: aggregate pass rate告诉我们的，只是Agent大概会不会做；它没有告诉我们Agent正在面对什么任务。

`Agent Psychometrics`这篇论文最有价值的地方，不是给Agent排行榜再加一层精度，而是把问题重新表述成“Agent能力、任务难度与成功概率”的关系问题。这个问题一旦成立，很多原本看起来像工程技巧的东西，就会变成统计校准、预算控制和路径选择的问题。它让我们第一次比较清楚地看到：Agent不是“能不能做”，而是“面对某类任务时，值得不值得继续做、该怎么做、做多久”。

但这篇论文对运行时的启发，不能被机械搬运。Embedding、IRT、LLM-as-Judge都很强，但它们更适合放在后台经验层和难度校准层，而不是直接变成前台控制器。真正需要改造的，不是再造一个Planner，而是给现有Plan机制加一层Plan Control Layer。它负责把自然语言计划转成可审计对象，把审计转成执行约束，把重规划转成局部patch，而不是把整个计划推倒重写。

<!--more-->

## 1. 问题起点：Agent不是不会Plan，而是不会约束Plan

现有Agent系统其实已经会Plan了。Codex、Claude Code、各种带Update Plan接口的工作流，都已经在做计划生成、步骤拆分、状态推进、SubAgent编排、工具调用和回写记录。问题不是“没有Planner”，而是“计划缺少控制面”。也就是说，Plan存在，但Plan没有被静态分析、领域审计、预算约束和停止条件所约束。

这个差别很重要。一个能写出计划的Agent，不代表它知道什么时候该停、什么时候该重排、什么时候该分配子任务、什么时候该请求人类介入。很多所谓“死循环”，表面上是执行层反复失败，实际上是控制层没有识别失败类型，没有更新成功概率，也没有把这些信息转成新的执行约束。

所以这篇文章的核心不是“再造一个Planner”，而是“把已有Plan机制升级成Plan Control Layer”。它更像计划静态分析器、领域感知Plan Auditor、Stop Controller和预算器的组合。它不替代原生Planner，而是约束原生Planner；不替代Exec，而是给Exec加合同；不替代RePlan，而是让RePlan以patch的方式发生。

> Audit不是评论Plan，而是生成执行约束。

这句话基本定义了本文的系统边界。

## 2. Agent=LLM+Scaffold：Plan本身也是Scaffold行为

如果把Agent看成`LLM + Scaffold`，就更容易理解为什么Plan本身也应该属于Scaffold的一部分。LLM负责语义理解、推理和生成，Scaffold负责工具调用、上下文管理、文件系统、计划更新、SubAgent编排、预算控制和状态持久化。Plan不是模型脑内的一段独白，而是Scaffold把模型意图变成可执行流程的方式。

这意味着Plan不是“想法文本”，而是“控制信号”。在一个成熟Agent里，Plan至少承担三种角色。

第一，它是任务分解器。它把模糊目标拆成可执行步骤，让系统知道先做什么、后做什么、哪些步骤依赖前置证据。

第二，它是约束接口。它不是只说“要做什么”，还应当说“不能怎么做”“做到什么程度必须停”“哪些证据必须先有”“哪些路径可以并行”。

第三，它是状态切片。Plan不是一次性文档，而是持续更新的状态对象。不同阶段的Plan、Audit、Exec、RePlan，其实是同一条任务在不同状态下的不同表达。

因此，Plan控制层不是附加物，而是Agent Scaffold中最关键的系统软件之一。没有它，Plan只会停留在Markdown层面；有了它，Plan才会进入机器可检查、可持久化、可回放、可patch的状态流。

## 3. 从Benchmark到Runtime：IRT的启发与边界

`Agent Psychometrics`的贡献，在于它把静态benchmark评测改造成了任务级成功概率预测。它借用IRT，把考生、试题、能力、难度、答对/答错这些概念映射到Agent、任务、能力、难度和成功/失败上。最简化的直觉可以写成：

$$
P(\mathrm{success})=\sigma(\theta_{\mathrm{agent}}-\beta_{\mathrm{task}})
$$

这条式子当然不能直接拿来管运行时，但它提供了一个非常关键的视角：任务难度不是主观感觉，而是可以被估计、校准和更新的隐变量。Agent能力也不是单一模型参数，而是`LLM能力 + Scaffold能力`的组合。

论文进一步把Agent能力拆成LLM能力与Scaffold能力，并用加和形式建模：

$$
P(y_{msj}=1\mid \theta_m,\theta_s,\beta_j)=\sigma(\theta_m+\theta_s-\beta_j)
$$

这对工程的启发很直接。你不能只看底模强不强，还得看Scaffold是不是把模型组织好了。很多时候，真正决定成败的不是模型本体，而是它被怎样喂上下文、怎样做工具调用、怎样做失败恢复、怎样维护执行历史。

但这里必须保留边界。Embedding、IRT、LLM-as-Judge都不应该被机械搬到前台控制中。它们的角色更像后台经验层和难度校准层：

- Embedding负责召回相似任务、相似失败状态、相似Plan Patch、相似Stop Condition。
- IRT负责校准当前Agent/Scaffold组合面对某类任务的大致成功概率、任务难度和预算建议。
- LLM-as-Judge或AuditAgent负责结构化审计，但不能只输出自然语言评价，必须输出可执行约束。

如果把这三者误当作控制器本身，就会把“统计信号”误当成“执行真理”。那会让系统看起来更聪明，实际上更脆。

## 4. 不要做新的Planner，做Plan Control Layer

这里要先把一个常见误区切开：这个问题不是“再造一个Planner”，也不是“再加一个Audit SubAgent”。当前很多Agent已经原生带有Plan和Update Plan机制。真正缺的不是计划生成，而是计划控制。

所以更准确的说法不是新建Planner，而是给已有Plan机制加一层`Plan Control Layer`。它像刹车系统、仪表盘、预算器和领域检查器的组合。它不替代原生Planner，而是约束、校准和修正原生Planner的输出。

如果只是做一个AuditAgent，输出一句“这个Plan可行/不可行”，那只是多了一次LLM调用。如果只是生成`plan-audited.md`，那仍然是自然语言上下文传递。如果Audit不能约束Exec，那整个循环只是流程装饰。真正有价值的是把Agent运行从“对话流”改造成“状态契约流”。

这个控制层的职责可以概括成四个动词：读Plan、审Plan、约束Exec、patch RePlan。它不是另一个大脑，而是计划静态分析器与领域检查器。我们要的是`Plan Linter / Plan Static Analyzer / Domain-Aware Plan Auditor`，不是一个更会说话的Planner。

### 不要重复造轮子

这里需要明确批判一下：如果只是做一个AuditAgent评价Plan，那只是多一次LLM调用；如果只是生成`plan-audited.md`，那仍然是自然语言上下文传递；如果只是Plan->Audit->Exec循环，但Audit不能约束Exec，那只是流程装饰。

真正有价值的，不是再发明一个“会反思的Agent”，而是把现有Agent运行从对话流改成状态契约流。现有系统已经有Planner，我们要做的是`Plan Linter / Plan Static Analyzer / Domain-Aware Plan Auditor / Plan Control Layer`，不是重复造轮子。

这也是为什么本文一直强调文件契约、patch和contract：这些东西的存在，决定了系统到底是“说过了”还是“生效了”。

## 5. Plan IR：把自然语言计划变成可审计对象

`plan.md`适合人类阅读，但不适合机器控制。只要还停留在Markdown层面，Audit就容易滑回自然语言点评，Exec也只能“参考建议”，不能受明确约束。

因此第一步不是生成新Plan，而是把现有Plan解析成`plan.ir.json`。这里的IR不是额外负担，而是控制层的中间表示。它让计划从“段落”变成“对象”，从“描述”变成“字段”，从“建议”变成“可检查约束”。

一个Plan IR至少应该包含这些字段：

- `step id`
- `description`
- `domain`
- `dependencies`
- `inputs`
- `outputs`
- `status`
- `estimated cost`
- `required evidence`
- `assigned_to`
- `preconditions`
- `stop_conditions`

示意结构可以长这样：

```json
{
  "task_id": "crypto-analysis-001",
  "plan_version": 1,
  "steps": [
    {
      "id": "s1",
      "description": "analyze DLP feasibility",
      "domain": "ecc",
      "dependencies": [],
      "inputs": ["curve_params", "public_keys"],
      "outputs": ["dlp_feasibility_report"],
      "status": "planned",
      "estimated_cost": "unknown",
      "required_evidence": ["group_order", "cofactor", "curve_family"],
      "assigned_to": "main_agent",
      "preconditions": ["curve parameters are known"],
      "stop_conditions": ["no evidence of exploitable weakness after domain checks"]
    }
  ]
}
```

为什么要IR？因为只有IR才能做结构化审计。Markdown可以给人看，但不能直接表达依赖关系、成本、前置条件、停止条件和证据要求。没有这些字段，Audit就只能做情绪化点评，不能做控制。

## 6. Audit Patch：审计不是评论，而是补丁和约束

Plan Control Layer的第二步，不是输出一篇审计意见，而是输出一个`audit.patch.json`。因为审计的价值不在于评论，而在于改变后续执行条件。

RePlan也不应该重新写一份新Plan。那样会制造版本混乱，也会让历史证据断裂。更合理的做法是：在`plan.ir.json`基础上应用patch。这个过程更像编译器pass或代码审查diff，而不是新建一个文档。

示意的patch结构可以是这样：

```json
{
  "patch_type": "plan_audit",
  "blocked_steps": [
    {
      "step_id": "try_dlp_bruteforce",
      "reason": "group order size unknown; brute force should not be attempted before order analysis"
    }
  ],
  "inserted_prechecks": [
    {
      "before_step": "try_dlp_attack",
      "new_step": "check_group_order_factorization",
      "knowledge_pack": "ecc_dlp_feasibility"
    }
  ],
  "budget_limits": [
    {
      "step_id": "lattice_scaling_sweep",
      "max_trials": 8,
      "stop_if": "no norm improvement in 3 consecutive trials"
    }
  ]
}
```

这里的关键点有三个。

第一，Audit不是只写结论，而是生成执行约束。`execution.contract.json`比自然语言点评更重要，因为它告诉Exec什么能做、什么不能做、做多少次、做到什么程度必须停。

第二，Audit输出应当能够插入新的前置检查，而不是只标记“失败”。一个好审计不会只说“不要尝试DLP暴力求解”，而是会把前置条件显式化：先检查群阶、cofactor、小子群、曲线族、是否存在MOV/Frey-Rück可用性，再决定是否有攻击面。

第三，Audit必须能生成停止条件，而不是只生成建议。没有停止条件的Plan，本质上只是更长的蛮干。

从这个角度看，AuditPatch更像代码审查diff，而不是复盘报告。它的作用是改变执行空间，而不是描述执行空间。

## 7. 领域知识激活：从“尝试DLP”到具体前置条件

这里是和普通Agent区别最大的地方。通用Agent往往只会说“检查DLP攻击是否可行”；Domain-Aware Audit则必须把模糊任务拆成具体检查项。

知识激活不是让模型泛泛地“考虑密码学知识”，而是把任务映射到一组可检查的领域前置条件、攻击路径、证据需求和停止条件。只有这样，Audit才从泛化反思变成具体约束。

以ECDH/DLP任务为例，激活一个`ecc_dlp_feasibility`知识包后，Audit至少要检查：

- group order factorization
- cofactor check
- small subgroup attack surface
- invalid curve assumptions
- MOV/Frey-Rück可用性
- Smart attack前提
- Pohlig-Hellman可分解性
- Pollard rho的实际成本
- transcript binding
- replay freshness
- MITM authentication assumptions
- SCA/oracle assumptions

如果这些前提都没有问题，那就不应该继续把资源押在DLP分支上。相反，应该转向协议层、实现层或认证层的其他风险面。

类似地，RSA任务不应该一上来就“尝试分解2048-bit N”，而应先激活`rsa_factorization`知识包，检查是否存在共享因子、低指数、padding问题、oracle、泄露、边信道、CRT实现缺陷等。如果这些前提都没有出现，继续蛮力分解只是浪费预算。

对lattice任务也是一样。`lattice_scaling`不该是无穷尝试，而应该先检查basis质量、norm bounds、scale策略、是否已经进入饱和、是否存在三轮无改进。如果没有新信息，就应该回到建模而不是继续扫尺度。

这也说明了为什么“通用反思”不够。通用反思只会说“再看看有没有别的可能”；领域知识激活会把可能性分解成具体的检查列表、证据需求和停止条件。两者不是一个等级。

## 8. SubAgent分配：不是多开Agent，而是成本收益决策

很多人一看到复杂任务，就本能地想“再开一个SubAgent”。但SubAgent不是越多越好，分配与否本身应该由Audit判断。

适合分配SubAgent的情况通常有四个条件：

1. 领域知识高度专门。
2. 输入输出边界清楚。
3. 任务可以并行。
4. 失败不会污染主上下文。

例如，检查RSA的`N`位数、`e`值、`gcd`共享因子，这种任务太轻，不需要SubAgent。直接在主线里做更便宜。

但判断MOV/Frey-Rück攻击是否适用，就可以交给一个`ECC Pairing Specialist`。因为这里需要专门的曲线知识、配对结构、嵌入度、目标群和成本判断，且输出边界很清楚：`yes/no`加上理由和证据。

诊断lattice scaling失败到底是basis问题、bounds问题还是scale问题，也可以分给一个`Lattice Diagnostic SubAgent`。但前提是它的输入契约清楚、输出契约清楚、预算清楚。否则分出去的不是能力，而是混乱。

所以SubAgent分配不是“让另一个Agent看看”，而是一个严格的成本收益决策。它必须有：

- input contract
- output contract
- budget
- stop condition
- handoff rule

如果这些都没有，就不该分配。

换句话说，SubAgent不是控制层的中心，而是控制层根据领域知识和预算策略临时调用的外包单元。它是工具，不是治理者。

## 9. Embedding与IRT：经验召回和难度校准

Embedding和IRT在这里都不应该被放到前台控制器的位置。它们更适合作为后台经验层和难度校准层。

Embedding负责经验召回。它可以回答的问题不是“当前任务真不真”，而是：

- 当前任务像过去哪些任务？
- 当前失败像过去哪些失败？
- 过去有哪些相似的Plan Patch？
- 过去有哪些Stop Condition真正有效？
- 哪些Knowledge Pack被证明能减少无效重试？

IRT负责能力/难度校准。它可以给出更粗粒度但更稳定的信号：

- 当前Agent+Scaffold组合面对这类任务大致有多大成功概率？
- 当前任务是否已经超过预算？
- 当前路径是否值得继续尝试？

但必须强调，Embedding不判断可行性，IRT不判断真理。最终控制仍然来自Domain Rules、Exec Evidence和Audit Contract。

也就是说，Embedding和IRT是后台经验与预算校准，不是前台决策者。它们可以帮助我们判断“像不像以前失败过的任务”“这条路大概值不值得继续”，但不能替代对当前证据的解析。

从工程顺序上看，第一版MVP甚至不需要强行实现IRT。更合理的顺序是：

1. V1：`Plan IR + Domain Knowledge Pack + Audit Patch`
2. V2：Embedding召回历史案例、失败模式和Plan Patch
3. V3：IRT校准任务难度、Agent能力和预算策略
4. V4：自动调整SubAgent分配、Stop Condition和RePlan策略

这个顺序很重要，因为它避免我们把统计建模当成地基。地基应该先是结构化计划和执行契约，统计模型应当在数据积累之后再进来。

## 10. Exec Result与RePlan Patch：重规划是状态更新，不是推倒重来

Exec不是“参考Audit建议”这么简单。Exec必须受`execution.contract.json`约束，并且在完成后返回`exec-state.json`和`evidence.jsonl`。这样RePlan才能基于证据做局部更新，而不是推倒重写。

`replan.patch.json`应该只做状态更新：

- 哪些步骤被证明无效
- 哪些前置条件需要补充
- 哪些证据不足
- 哪些新分支值得新增
- 哪些SubAgent应该重新分配

如果每次都重写一份新Plan，就会失去版本连续性。反过来，如果只在脑子里“记得上次失败了”，那么后续步骤就很容易把历史错误重新带回来。Patch式RePlan的好处就在于它既保留了历史，又避免了计划爆炸。

RePlan因此不是重新写计划，而是对计划做局部语义修补。它的输入不是空白页，而是`exec-state.json + evidence.jsonl + audit.patch.json`，它的输出也不是新世界，而是`replan.patch.json`。

## 11. Stop Controller：让Agent知道什么时候不该继续蛮干

Agent最常见的蛮干，不是因为没有Plan，而是因为没有停止条件、预算感知和失败类型判断。

所以Stop Controller不是简单的`max loop count`。它应该根据以下信息判断是否继续：

- 是否重复同类失败
- 是否缺少关键前置条件
- 是否没有新信息增量
- 是否已经超过预算
- 是否违反Audit Contract
- 是否需要RePlan或请求人类介入

可以把继续条件写得更形式化一点：

$$
\text{continue} \Leftrightarrow \mathbb{E}[U(P_i)\mid \mathcal{D}_{1:t}] > 0 \land \neg \mathrm{violates}(\mathrm{AuditContract})
$$

其中

$$
U(P_i)=\alpha \hat p_i-\beta \hat c_i-\gamma \hat r_i
$$

这意味着只有当预期收益仍然为正，而且没有违反审计合同，才值得继续。这不是把Agent变成经济学模型，而是把“蛮干阈值”显式化。

对lattice scaling的例子来说，不能无限尝试scale。如果连续几次没有norm improvement，就应该停止scale sweep，回到basis/bounds/modeling诊断，而不是继续盲扫。

对RSA解密的例子来说，如果是2048-bit `N`，又没有泄露、oracle、低指数、共享因子或padding问题，那就不能蛮力分解。此时Stop Controller应该输出“当前信息不足”或转向前提检查，而不是继续制造算力幻觉。

因此Stop Controller的职责不是禁止执行，而是防止错误的执行意图在没有新证据的情况下被无限重复。它是预算、证据和失败类型共同驱动的止损器。

### 失败类型不是一个标签，而是一组可区分的状态

Stop Controller要真正起作用，首先得把“失败”拆开。很多Agent把失败都当成同一件事：这一步没成，于是再试一次。问题在于，失败有不同的语义，而不同语义对应不同动作。

可以粗略分成四类：

1. **证据不足型失败**：方向可能对，但现有证据不够。此时应该补预检，而不是盲试。
2. **路径失配型失败**：证据已经说明这条路不对。此时应该RePlan，而不是再加预算。
3. **实现局部型失败**：任务方向没错，但实现细节有误。此时应该修补，而不是重写。
4. **预算耗尽型失败**：方向未必错，但收益已经低于成本。此时应该停止或请求人类介入。

把失败类型区分开，Stop Controller才有意义。否则它只是一个更聪明的循环计数器。

### Stop Controller与Audit Contract的关系

Stop Controller不是独立的判断器，它应该依附于Audit Contract工作。Audit Contract定义了什么算有效进展、什么算重复失败、什么算证据增量、什么算违反约束。Stop Controller再根据这些合同判断是否继续。

这意味着停止条件不应该是一个孤立的`max loop count`，而应该是一组和任务、证据、预算绑定的判定式。例如：

$$
\text{stop} \Leftrightarrow \mathrm{repeat\_failure} \lor \mathrm{budget\_exceeded} \lor \neg \mathrm{evidence\_gain} \lor \mathrm{contract\_violation}
$$

这比“最多尝试五次”强得多，因为它绑定的是语义，不是次数。

## 12. 工程落点：状态契约流，而不是对话流

要把上面的东西落地，最好把工作区设计成一个最小的状态契约目录，而不是在聊天上下文里东一段西一段地传。

```text
.agent-workspace/
  task.json
  plan.md
  plan.ir.json
  activated-knowledge.json
  audit.patch.json
  execution.contract.json
  exec-state.json
  evidence.jsonl
  replan.patch.json
  final.md
```

每个文件的职责应该非常明确：

- `task.json`：任务边界、成功标准、预算、工具权限。
- `plan.md`：现有Agent原生Plan，人类可读。
- `plan.ir.json`：结构化Plan，中间表示。
- `activated-knowledge.json`：被激活的领域知识包。
- `audit.patch.json`：审计补丁。
- `execution.contract.json`：执行阶段必须遵守的约束。
- `exec-state.json`：当前执行状态。
- `evidence.jsonl`：工具调用与证据日志。
- `replan.patch.json`：基于Exec Result的局部Plan更新。
- `final.md`：最终交付结果。

这个目录的核心思想其实很简单：Plan/Audit/Exec/RePlan不是几个Agent聊天，而是几个可持久化状态之间的转换。状态之间不应该只靠上下文窗口传递，而应该靠文件契约传递。

如果再说得更直白一点，`plan.md`是人类的语言，`plan.ir.json`是机器的语言，`audit.patch.json`和`execution.contract.json`是机器之间的契约，`evidence.jsonl`是历史，`replan.patch.json`是增量修补。这样系统才不会在每一次对话轮转里遗忘自己到底做到了哪一步。

### 一个端到端例子

把上面的目录真的串起来，大致会像这样：

1. `task.json`写明：目标是什么、预算多少、允许哪些工具、成功标准是什么。
2. 原生Agent写出`plan.md`。
3. 控制层把`plan.md`解析成`plan.ir.json`。
4. `activated-knowledge.json`激活对应知识包，比如`ecc_dlp_feasibility`或`rsa_factorization`。
5. `audit.patch.json`插入前置检查、预算限制和停止条件。
6. `execution.contract.json`约束Exec只能在合同范围内行动。
7. Exec执行后写出`exec-state.json`和`evidence.jsonl`。
8. RePlan读取这些文件，生成`replan.patch.json`，只补局部步骤，不重写整张Plan。
9. 最终结果进入`final.md`。

这样一来，Plan/Audit/Exec/RePlan就不再是对话轮次，而是文件状态机。文件就是接口，patch就是更新，证据就是记忆，合同就是边界。

### 这比自然语言循环更稳的原因

自然语言循环的问题在于，它把所有状态都塞进上下文窗口里。一旦上下文长了，重要信息和噪声就会混在一起，模型会逐渐忘记哪些是证据、哪些是猜测、哪些是合同、哪些是历史。

状态契约流解决的是这个问题：每个阶段只读自己需要的文件，只写自己负责的文件，只通过明确的patch和contract传递状态。这种方式更接近编译器流水线，也更接近工程上的可靠系统。

## 13. 结论：有价值的不是多一个SubAgent，而是让已有Agent受到领域感知的计划控制

如果把全文压缩成一句话，就是：`Agent Psychometrics`给了我们一个关于任务难度与成功概率的校准视角，但真正的工程价值不在于再做一个Planner，而在于给已有的Plan机制加上一层领域感知的计划控制层。

这层控制不是为了替代现有Agent，而是为了让现有Agent更知道什么时候该继续，什么时候该停，什么时候该分派，什么时候该回滚，什么时候该请求人类介入。它把“计划”从自然语言文档变成结构化对象，把“审计”从评论变成约束，把“重规划”从重写变成patch，把“上下文管理”从堆积变成契约。

所以，本文真正想推进的不是一个新框架名词，而是一种更成熟的系统观：

- 不是再造Planner，而是控制Plan。
- 不是增加聊天式SubAgent，而是增加状态契约。
- 不是用反思替代执行，而是用领域知识、预算和停止条件去约束执行。

这才是Agent Psychometrics、IRT和Embedding对长跨度Agent最实际的启发。

## References

- Agent Psychometrics: Task-level Performance Prediction in Agentic Coding Benchmarks. arXiv:2604.00594.
- OpenReview workshop version: Agents in the Wild, 2026.
- SWE-bench: Can Language Models Resolve Real-World GitHub Issues? arXiv:2310.06770.
- AgentBench: Evaluating LLMs as Agents. arXiv:2308.03688.
- G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment. arXiv:2303.16634.
- Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena. arXiv:2306.05685.

[^agent-psychometrics-arxiv]: 论文摘要指出，agentic coding正在从静态单步生成转向多步工具交互，而aggregate pass rate会掩盖benchmark内部任务差异。来源：<https://arxiv.org/abs/2604.00594>
[^agent-psychometrics-openreview]: 论文方法部分将Agent能力拆成LLM能力与scaffold能力，并采用加和形式建模成功概率。来源：<https://openreview.net/forum?id=T4ZFDwXaIS>
[^agent-psychometrics-judge]: 论文把LLM-as-a-judge特征定义为沿固定rubric对任务打分，例如验证容易度、领域知识需求、修复复杂度等。来源：<https://openreview.net/attachment?id=T4ZFDwXaIS&name=pdf>
[^agent-psychometrics-artifacts]: 论文实验显示，加入repository state、tests、solution等agentic task artifacts，相比只看problem statement能更好预测任务难度。来源：<https://openreview.net/attachment?id=T4ZFDwXaIS&name=pdf>
[^swebench]: SWE-bench将真实GitHub issues与真实修复补丁作为评测对象，强调软件工程任务的难度来自仓库状态、测试与补丁验证的组合。来源：<https://arxiv.org/abs/2310.06770>
[^agentbench]: AgentBench把模型放入多种环境任务中评估，核心思想是Agent能力必须在交互与环境反馈中被测量。来源：<https://arxiv.org/abs/2308.03688>
[^geval]: G-Eval强调使用结构化rubric和GPT-4式评审来逼近人类评价。来源：<https://arxiv.org/abs/2303.16634>
[^mtbench]: MT-Bench与Chatbot Arena相关工作展示了pairwise judging与偏好比较在LLM评测中的作用。来源：<https://arxiv.org/abs/2306.05685>

