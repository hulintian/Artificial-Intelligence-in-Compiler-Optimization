# README 旧论文分析与近两年 AI Compile 顶会论文补充

更新时间：2026-03-26

筛选原则：

- 优先保留 2024-2025 已正式发表在 `PLDI`、`CGO`、`CC`、`ICML` 等强相关顶会的论文。
- 重点放在 `AI for compilation / compiler optimization`，也就是用 AI/ML/LLM 改进编译分析、优化搜索、调优和代码生成。
- 少量补充 1-2 篇“紧贴 AI 编译/codegen 边界”的 benchmark 论文，用来判断这条线当前真实到了什么程度。

说明：

- 每篇里的 `做什么 / 核心方法 / 结果` 主要根据论文官方页面、摘要和公开介绍整理。
- 每篇里的 `为什么值得看` 是我基于这个仓库主题做的归纳判断。

## 1. README 现有论文脉络

### 1.1 这份 README 的主线其实很清楚

- `2004-2013`：以 `phase ordering`、`iterative compilation`、`heuristic learning` 为主，是“机器学习替代人工启发式”的早期阶段。
- `2014-2020`：开始出现更系统的 `autotuning` 框架、性能模型、迁移学习与表示学习，关注“如何在巨大优化空间里更快找到可行点”。
- `2021-2024`：一方面延伸到 `tensor compiler / auto-scheduling / cost model`，另一方面开始出现 `LLM for compiler optimization` 这类新方向。

### 1.2 这份 README 现在最明显的缺口

- `2024` 以后更新很稀疏。README 里最新代表作基本只到 `BaCO (ASPLOS 2024)`，而 `2025` 这一波论文已经明显转向了 `LLM + compiler` 和 `compiler-native ML architecture`。
- 旧清单更偏“调 exposed flags / pass ordering / auto-scheduling”，但近两年热点已经扩展到：
  - `foundation model for compiler IR / assembly`
  - `LLM + compiler 协作式分析`
  - `隐藏 heuristics / latent knobs` 的自动发现
  - `compiler-native neural architecture`，不再满足于通用 `GNN`
  - `低层 GPU schedule / SASS` 的学习优化
  - `LLM 是否真的理解 IR` 这类基准化评测

### 1.3 我对 README 的判断

- 它适合拿来补历史脉络，尤其是 `phase ordering -> autotuning -> learned cost model` 的演化。
- 如果你的目标是继续跟进“现在 AI 能在编译器里做什么”，那阅读重心应该从旧的 `flag tuning` 进一步转向：
  - `compiler + foundation model`
  - `compiler-guided reasoning`
  - `search space abstraction`
  - `benchmarking / robustness / generalization`

## 2. 新的 top paper / conference 论文清单

说明：截至 `2026-03-26`，公开可稳定引用、且与这个仓库主题最贴近的“近期顶会代表作”，主要集中在 `2024-2025`。

### A. 基础设施、搜索空间与自动调优

#### 1. [The Next 700 ML-Enabled Compiler Optimizations](https://deepmind.google/research/publications/57746/)

- 会议：`CC 2024`
- `做什么`：提出 `ML-Compiler-Bridge`，试图解决“ML 模型很难真正嵌入编译器内部并长期维护”的工程问题。
- `核心方法`：把模型开发放在常规 Python/ML 栈里，但给编译器提供一条高效、端到端、可部署的桥接路径，而不是为每个优化任务手工打补丁。
- `结果`：在 4 类优化任务上、跨多个编译器版本与训练/推理场景验证这套桥接方案。
- `为什么值得看`：README 里很多论文是在研究“某个模型怎样调某个优化”，这篇更像是把“做一个可复用的 ML 编译优化平台”这件事正式工程化。

#### 2. [Revealing Compiler Heuristics through Automated Discovery and Optimization](https://conf.researchr.org/details/cgo-2024/cgo-2024-main-conference/18/Revealing-Compiler-Heuristics-through-Automated-Discovery-and-Optimization)

- 会议：`CGO 2024`
- `做什么`：指出真正有价值的优化空间不只在 `-O` 级别或显式 flags 上，还藏在编译器内部大量未暴露、未标注的 heuristics 中。
- `核心方法`：自动发现并暴露这些内部 heuristic/parameter，把过去“根本没法调”的部分变成可搜索、可优化的空间。
- `结果`：论文的关键贡献不是某一个单点指标，而是把 auto-tuning 的对象从“外部 flags”扩展到了“编译器内部决策逻辑”。
- `为什么值得看`：这篇和 README 旧线最大的差别在于，它不再默认搜索空间是现成给你的，而是先把搜索空间本身挖出来。

#### 3. [Exponentially Expanding the Phase-Ordering Search Space via Dormant Information](https://conf.researchr.org/details/CC-2024/CC-2024-papers/21/Exponentially-Expanding-the-Phase-Ordering-Search-Space-via-Dormant-Information)

- 会议：`CC 2024`
- `做什么`：继续解决经典 `phase ordering`，但反对以往依赖人工经验的激进剪枝。
- `核心方法`：利用 `dormant information` 对优化 pass 进行更保守的剪枝，只排除当前“休眠”的变换，让搜索空间大得多但不至于失控；系统名为 `FlexPO`。
- `结果`：`FlexPO` 能探索比已有方案大得多的空间，实验中生成的程序相较现代编译器可达到 `12%` 更快或 `17.6%` 更小。
- `为什么值得看`：如果你想接着 README 里那条 `phase ordering + RL` 线往后看，这篇是一个很自然的新起点。

#### 4. [Towards Efficient Compiler Auto-tuning: Leveraging Synergistic Search Spaces](https://2025.cgo.org/details/cgo-2025-papers/48/Towards-Efficient-Compiler-Auto-tuning-Leveraging-Synergistic-Search-Spaces)

- 会议：`CGO 2025`
- `做什么`：继续做 pass sequence auto-tuning，但重点放在“pass 之间的协同关系”而不是单个 pass 的独立作用。
- `核心方法`：先发现能共同降低 IR 指令数的 `synergistic pass pairs`，再用 `K-means` 聚类形成 `coreset`，最后训练监督模型为新程序预测最有用的 coreset。
- `结果`：在 `MiBench`、`CBench`、`NPB`、`CHStone` 等 10 个 benchmark 数据集上，相比 `Oz` 平均能再降低 `7.5%` IR 指令数；在 code size 上，相比 `Oz` 平均降低 `13.6%`。
- `为什么值得看`：这篇比“暴力搜索 pass 序列”更工程化，也比纯 RL 更容易解释，适合做落地型 pass ordering baseline。

#### 5. [CuAsmRL: optimizing GPU SASS schedules via deep reinforcement learning](https://2025.cgo.org/details/cgo-2025-papers/29/CuAsmRL-optimizing-GPU-SASS-schedules-via-deep-reinforcement-learning)

- 会议：`CGO 2025`
- `做什么`：把优化对象继续下探到了 `GPU SASS` 指令调度层，目标是进一步压榨 CUDA kernel 的吞吐。
- `核心方法`：把 SASS 调度建模成一个 `assembly game`，从 `-O3` 生成的初始 schedule 出发，让 RL agent 通过动作变异当前 schedule，并根据真实 GPU throughput 获取奖励。
- `结果`：对已有 specialized CUDA kernels 可以透明地继续优化，实验报告 `最高 26%`、`平均 9%` 的性能提升。
- `为什么值得看`：README 里的 auto-tuning 大多停在编译 flag、schedule 参数或 tensor IR；这篇说明“AI for compilation”已经走到更底层的机器级调度了。

### B. 编译分析、性能建模与泛化

#### 6. [Fast and Accurate Context-Aware Basic Block Timing Prediction using Transformers](https://conf.researchr.org/details/CC-2024/CC-2024-papers/19/Fast-and-Accurate-Context-Aware-Basic-Block-Timing-Prediction-using-Transformers-)

- 会议：`CC 2024`
- `做什么`：做基本块执行时间预测，用于性能建模/时序分析，但明确引入 `context-aware` 视角。
- `核心方法`：提出 `ORXESTRA`，使用 `Transformers XL` 在考虑执行上下文的前提下预测 timing，而不是像传统模型那样孤立地看单个 basic block。
- `结果`：在 `ARM Cortex M4/M7/A53/A72` 等目标上，预测精度和预测速度都优于已有的 ML timing 模型。
- `为什么值得看`：README 里性能模型主要偏 `tensor tuning`；这篇更偏传统编译分析与嵌入式时序建模，能补齐另一个很重要的 cost model 分支。

#### 7. [DFA-Net: A Compiler-Specific Neural Architecture for Robust Generalization in Data Flow Analyses](https://conf.researchr.org/details/CC-2025/CC-2025-main-conference/9/DFA-Net-A-Compiler-Specific-Neural-Architecture-for-Robust-Generalization-in-Data-Fl)

- 会议：`CC 2025`
- `做什么`：面向编译器最核心的数据流分析任务，回答“神经网络能不能像编译器一样做稳定泛化”的问题。
- `核心方法`：不直接套通用 `GNN`，而是把数据流分析拆成 `initialization / transfer / meet` 三类网络模块，把编译器知识直接写进网络结构。
- `结果`：在复杂程序上，对 `data dependencies` 的 `F1` 达到 `0.761`，而 GNN baseline 只有 `0.009`；对 `dominators`，`0.989` 对 `0.196`。`liveness` 和 `reachability` 上也保持稳定高分。
- `为什么值得看`：这篇是“compiler-native inductive bias”路线的代表作，和 README 里偏通用表示学习的论文相比，它更强调“模型结构要像编译器思考”。

### C. LLM 与编译器的结合

#### 8. [Reductive Analysis with Compiler-Guided Large Language Models for Input-Centric Code Optimizations](https://pldi25.sigplan.org/details/pldi-2025-papers/33/Reductive-Analysis-with-Compiler-Guided-Large-Language-Models-for-Input-Centric-Code-)

- 会议：`PLDI 2025`
- `做什么`：面向 `input-centric optimization`，自动找出哪些输入特征真正驱动程序行为，从而做自适应优化。
- `核心方法`：不是直接把整个程序丢给 LLM，而是用编译器先做 `compiler-guided reductive analysis`，把分析对象缩减到 LLM 能处理的规模，再让 LLM推断关键输入特征。
- `结果`：相较基于 profiling 的传统方法，关键输入识别时间平均缩短 `44x`；若用本地 LLM 可达 `450x`。在自适应 OpenMP 并行决策上达到 `92.6%` 准确率，并在 serverless 场景带来 `20-30%` 性能提升和 `50-60%` 资源节省。
- `为什么值得看`：这篇很重要，因为它代表了一种更现实的范式：`compiler 负责结构化裁剪，LLM 负责高层推断`，而不是幻想 LLM 单独取代编译器。

#### 9. [Finding Missed Code Size Optimizations in Compilers using Large Language Models](https://conf.researchr.org/details/CC-2025/CC-2025-main-conference/8/Finding-Missed-Code-Size-Optimizations-in-Compilers-using-Large-Language-Models)

- 会议：`CC 2025`
- `做什么`：把 LLM 用在编译器“性能测试”而不是“优化决策”上，专门找编译器遗漏的 code-size optimization。
- `核心方法`：把 `LLM` 与 `differential testing` 结合，让 LLM 负责生成测试程序，再用启发式和分析手段识别异常优化行为。
- `结果`：实现非常轻量，论文称主体只需要 `150` 行以内代码；不仅能用于 `C/C++`，改 prompt 后还能迁移到 `Rust` 和 `Swift`；总共报告了 `24` 个生产编译器 bug。
- `为什么值得看`：它提醒我们，AI 不只是“帮编译器做优化”，也可以“帮我们检查编译器没把优化做好”。

#### 10. [LLM Compiler: Foundation Language Models for Compiler Optimization](https://conf.researchr.org/details/CC-2025/CC-2025-main-conference/13/LLM-Compiler-Foundation-Language-Models-for-Compiler-Optimization)

- 会议：`CC 2025`
- `做什么`：构建面向编译任务的基础模型，而不是每个优化点各训一个小模型。
- `核心方法`：基于 `Code Llama`，在 `546B tokens` 的 `LLVM-IR + assembly` 语料上继续训练，再做 instruction fine-tuning，让模型理解 compiler behavior、优化效果和反汇编/回译任务。
- `结果`：发布 `7B` 与 `13B` 两个模型；其 fine-tuned 版本在 code-size optimization 上达到 autotuning 搜索潜力的 `77%`，在 `x86_64/ARM assembly -> LLVM-IR` 回译上达到 `45%` round trip（`14%` exact match）。
- `为什么值得看`：这是近两年最值得优先跟的“compiler foundation model”工作之一，也是 README 里 `Large Language Models for Compiler Optimization (CC 2023)` 的直接后继。

#### 11. [Can Large Language Models Understand Intermediate Representations in Compilers?](https://proceedings.mlr.press/v267/jiang25p.html)

- 会议：`ICML 2025`
- `做什么`：系统评测当前 LLM 到底能不能理解编译器 IR。
- `核心方法`：从 `CFG reconstruction`、`decompilation`、`code summarization`、`execution reasoning` 四类任务评估 `GPT-4`、`DeepSeek`、`Llama 3`、`Code Llama` 等模型。
- `结果`：结论非常明确：LLM 对 IR 的语法和高层结构有一定理解，但在 `instruction-level reasoning`、控制流推理、循环处理和动态执行推理上表现持续偏弱。
- `为什么值得看`：如果你打算做 “LLM + compiler”，这篇几乎是必读 sanity check。它告诉你哪些任务 LLM 现在能做，哪些还不能放心交给它。

### D. 边界扩展：AI codegen / benchmark

#### 12. [KernelBench: Can LLMs Write Efficient GPU Kernels?](https://proceedings.mlr.press/v267/ouyang25a.html)

- 会议：`ICML 2025`
- `做什么`：建立一个更接近真实工程环境的 benchmark，评估 LLM 能否写出既正确又快的 GPU kernels。
- `核心方法`：构建 `250` 个 PyTorch workload，并提出 `fast_p` 指标，要求生成的 kernel 不仅正确，还得相对 baseline 达到足够速度提升。
- `结果`：前沿 reasoning model 虽然是当前最强，但总体仍然偏弱，`匹配 PyTorch baseline 的案例不到 20%`；加入执行反馈和 profiling 反馈后，性能会提升，但 benchmark 依然很难。
- `为什么值得看`：它不完全是 compiler paper，但和 AI 编译/codegen 的边界非常近，而且给“LLM 直接写高性能内核”这件事做了非常硬的现实校验。

## 3. 我建议的阅读顺序

如果你的目标是“尽快理解这条方向现在的主流问题”，我建议：

1. `LLM Compiler`：先建立“compiler foundation model”这条线的整体图景。
2. `Reductive Analysis with Compiler-Guided LLMs`：看清楚编译器与 LLM 如何分工协作。
3. `DFA-Net`：理解为什么“通用图网络”不够，为什么需要 compiler-specific architecture。
4. `Revealing Compiler Heuristics...` 和 `Synergistic Search Spaces`：把注意力从“给定搜索空间怎么找”推进到“搜索空间本身怎么设计/抽象”。
5. `Can LLMs Understand IR?`：给前面所有 LLM 论文补一层现实边界。
6. `CuAsmRL` 和 `KernelBench`：如果你更关心 AI workload / GPU codegen，再继续往低层走。

## 4. 对这个仓库后续维护的建议

- `README` 的旧论文可以保留，但建议新增一个 `Recent Trends (2024-2025)` 小节，不然读者会误以为这个领域还停在 `pass ordering + autotuning`。
- 未来可以把论文按下面四条线重组，而不是只按年份排：
  - `pass ordering / auto-tuning`
  - `cost model / performance prediction / program analysis`
  - `LLM / foundation model for compiler tasks`
  - `AI compiler / tensor compiler / GPU codegen`
- 如果后面继续记笔记，我建议优先补这几篇：`LLM Compiler`、`DFA-Net`、`Reductive Analysis`、`Synergistic Search Spaces`。
