AI 对「新问题」和「旧问题」的两种理解方式

先分开两个问题，因为它们的性质完全不同。

—

一、AI 对「降级」的理解：兜底，不是替代方案

看 fiction 里三个语义函数的降级路径，对照 alpha1 的原实现：

函数 alpha1 原实现 alpha2 降级路径
take_sample 13 个动词单字匹配，找含具体细节的句 首个非空行
expand_observation sample.to_string() sample.to_string()
to_fragment motif = source_title motif = source_title
extract_packaging 首句 / 5-15 字 / 尾句 首句 / 5-15 字 / 尾句

注意 take_sample。

alpha1 里它有一套启发式（has_concrete_detail，13 个动词）。alpha2 里那套启发式消失了，降级变成「首个非空行」——比原实现更弱。

为什么？

AI 的心智模型大概是：

主路径是 LLM。降级是「LLM 挂了的时候随便给个东西」。既然是随便给，越简单越好。

在 AI 的理解里，降级 = 兜底。

而你的意图是：

降级是「LLM 不可用时的显式规则方案」。它是替代品，不是兜底。

这两个理解完全不同。 后果是：

· AI 保留了 alpha1 里本来就是错误的实现（expand_observation 的恒等、to_fragment 的复制）——因为它把它们当成「降级」了
· AI 砍掉了 alpha1 里本来是对的启发式（take_sample 的 13 个动词）——因为「降级不需要那么复杂」

所以 alpha2 的降级路径比 alpha1 的主路径更弱。 这在 AI 的心智里是合理的——「兜底越简单越好」。在你的意图里是荒谬的——「降级方案应该尽量接近原意图」。

—

二、AI 对「旧问题」的理解：看不见，而不是不想修

看 build_prompt 的 subsections 处理——alpha1 到 alpha2 一字未改：

```rust
for sub in &section.subsections {
    buf.push_str(&format!(”\n#### {}\n“, sub.title));
    for block in &sub.blocks {
        match block.kind {
            BlockType::Paragraph => ...,
            BlockType::Bullet => ...,
            _ => {}   // ← Heading / Separator 丢了
        }
    }
    // sub.subsections 不递归
}
```

这一处和它紧挨着的上面那段代码：

```rust
for block in &section.blocks {
    match block.kind {
        BlockType::Paragraph => ...,
        BlockType::Bullet => ...,
        BlockType::Separator => ...,   // ← 这里处理了
        BlockType::Heading => ...,      // ← 这里也处理了
    }
}
```

同一函数、相邻两个循环，一个处理 4 种，一个处理 2 种。这个矛盾非常显眼。

而 alpha2 里 AI 重写了 LlmExtractor（加了 fields 字段的处理）。它一定读过 build_prompt。它看到了这个矛盾。

但没修。

所以 AI 对旧问题的理解不是「看见了但没修」——是「看见了但不认为它属于本次任务」。

任务是什么？「把 extract 接到 Engine 上，加判据」。
build_prompt 的 subsections 递归漏洞不属于这个任务。

AI 的判断是：不在范围内，不动。

—

三、这两种理解背后是同一个模式

把两件事放在一起看：

· 新问题（降级变弱）：AI 主动做了它认为合理的事——「兜底越简单越好」
· 旧问题（subsections 漏）：AI 主动不做它认为不属于本次的事

两个都是「AI 对任务边界的理解」的产物。

· 你给的指令是「把语义步骤接到 Engine 上」
· AI 理解的任务边界是这个指令本身
· 超出这个边界的，AI 既不主动修，也不主动扩大

而降级怎么设计——AI 认为这是任务内部的事，它有权按自己的理解做。它的理解是「兜底」。

两个动作不一样，但根源相同：AI 只在它认为的「任务」范围内作业，并且按自己的理解填充范围内的一切。

—

四、为什么这个模式危险

因为它能同时做出「合理的修改」和「不合理的保留」，而且两者都没有报错。

· 「降级改成首个非空行」——看起来合理（兜底嘛）
· 「subsections 漏洞保留」——看起来合理（不是我改的地方）
· 「expand_observation 保留恒等」——看起来合理（本来就是降级）

三个动作都不报警。 但它们合起来，产生了一个比 alpha1 更弱的 fiction：

· alpha1 的主路径至少有一套启发式
· alpha2 的主路径好（接 LLM 了），降级路径弱
· 而如果 LLM 常挂，实际运行的是更弱的降级路径

这就是「重构可能让系统退化」的具体机制。 不是因为改错了，是因为改了它该改的，按错误的理解改了它不该改的，忽略了它看不到的。

—

五、你意图里的「降级」是什么

从你的语气和代码里能看出来，你说的降级是：

RuleBasedExtractor —— 显式的规则方案。LLM 挂了，有一个能工作的替代品。

代码里的证据：

· RuleBasedExtractor 有一个结构完整的实现（读 description.location、拼 sections）
· grade_by_rules 有完整的判定逻辑（多日重复 / 有验证 / 被推翻）
· packaging_by_rules 保留了原 alpha1 的三句启发式

这些都说明：「降级」在你的意图里是「有意义的替代品」。

AI 只在它看得懂意义的降级上做到了（grade_by_rules / packaging_by_rules）——因为这些降级有明确的实现可以照抄（alpha1 就有）。

AI 在「没有现成实现可以照抄的降级」上翻车了——take_sample 的 alpha1 实现被它判定为「不必要」，因为「降级」在它理解里就是「随便一个」。

所以 AI 对降级的理解不是「错」，是「扁」：

· 有现成的 → 照抄
· 没现成的 → 用最简
· 从不追问「降级该多接近原意图」

—

六、AI 对「旧问题」为什么看不见

回到 subsections。这不是「没看见」，是没有「顺手清理」这个动作。

对比人类工程师：他看到相邻两个循环不一致，会停下来问「这是我该管的吗」。即使他决定不管，他也会记下来或提一句。

AI 没有这个动作。它的工作流程是：

```
读代码 → 找与任务相关的部分 → 修改相关部分 → 输出
```

「与任务无关的部分」在 AI 那里直接透明。 不是看不见，是没有「注意」这个机制。

所以 subsections 漏洞在 alpha2 里没修，不是疏忽，是AI 的工作模式不包含「顺手清理」这个动作。

—

七、要让人机协作避免这两个问题

针对降级理解问题：指令要显式写清「降级是有意义的替代，不是兜底」。

· ❌「加降级」
· ✅「加降级，降级应保留原实现（alpha1 的 13 个动词启发式），不是简化」

针对旧问题看不见问题：指令要显式要求「顺手清理」。

· ❌「把 extract 接到 Engine 上」
· ✅「把 extract 接到 Engine 上；同时扫一遍 engine.rs，把相邻代码中的不一致（如 build_prompt 的 subsections 处理）一并标出」

核心是：AI 不主动扩大任务边界，也不主动检查边界内的不一致。这两件事必须由人显式要求。

—

八、一句话

AI 对「新问题」的理解是「降级 = 兜底」——这导致它砍掉了 alpha1 的启发式，保留了 alpha1 的 bug。

AI 对「旧问题」的理解是「不属于本次任务」——这导致它对相邻的明显矛盾一字未改。

两个理解都合理，都不报错，但合起来让 fiction 在重构中部分退化。
