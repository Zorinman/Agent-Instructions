# HTML Report Format

The architectural review is rendered as a single self-contained HTML file in the OS temp directory. Tailwind and Mermaid both come from CDNs. Mermaid handles graph-shaped diagrams reliably; hand-built divs and inline SVG handle the more editorial visuals (mass diagrams, cross-sections). Mix the two — don't lean on Mermaid for everything, it'll start to look generic.

## Scaffold

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="utf-8" />
    <title>架构审查 — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* small custom layer for things Tailwind doesn't cover cleanly:
         dashed seam lines, hand-drawn-feeling arrow heads, etc. */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## Header

Repo name, date, and a compact legend in Chinese: 实线框 = 模块 (module), 虚线 = 接缝 (seam), 红色箭头 = 泄漏 (leakage), 深色粗框 = 深模块 (deep module). No introduction paragraph — straight into the candidates.

## Candidate card

The diagrams carry the weight. Prose is sparse and plain — written in Chinese, using the Chinese glossary terms from the Tone section below (not the raw English `/codebase-design` terms), each attached to a short inline gloss on first use per card.

Each candidate is one `<article>`:

- **Title** — short, names the deepening (e.g. "Collapse the Order intake pipeline").
- **Badge row** — recommendation strength (`Strong` = emerald, `Worth exploring` = amber, `Speculative` = slate), plus a tag for the dependency category (`in-process`, `local-substitutable`, `ports & adapters`, `mock`).
- **Files** — monospaced list, `font-mono text-sm`.
- **Before / After diagram** — the centrepiece. Two columns, side by side. See patterns below.
- **Problem** — one sentence. What hurts.
- **Solution** — one sentence. What changes.
- **Wins** — bullets, ≤6 words each. e.g. "Tests hit one interface", "Pricing logic stops leaking", "Delete 4 shallow wrappers".
- **ADR callout** (if applicable) — one line in an amber-tinted box.

No paragraphs of explanation. If the diagram needs a paragraph to be understood, redraw the diagram.

## Diagram patterns

Pick the pattern that fits the candidate. Mix them. Don't make every diagram look the same — variety is part of the point.

### Mermaid graph (the workhorse for dependencies / call flow)

Use a Mermaid `flowchart` or `graph` when the point is "X calls Y calls Z, and look at the mess." Wrap it in a Tailwind-styled card so it doesn't feel parachuted in. Style with classDef to colour leakage edges red and the deep module dark. Sequence diagrams work well for "before: 6 round-trips; after: 1."

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.泄漏.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

Node labels can stay as the actual code identifiers (file/class/function names) — those are facts about the codebase, not glossary terms, so they don't need translation. Edge/annotation labels that describe the *relationship* (leak, seam, deep module) should be in Chinese.

### Hand-built boxes-and-arrows (when Mermaid's layout fights you)

Modules as `<div>`s with borders and labels. Arrows as inline SVG `<line>` or `<path>` elements positioned absolutely over a relative container. Reach for this when you want the "after" diagram to feel like one thick-bordered deep module with greyed-out internals — Mermaid won't render that with the right weight.

### Cross-section (good for layered shallowness)

Stack horizontal bands (`h-12 border-l-4`) to show layers a call passes through. Before: 6 thin layers each doing nothing. After: 1 thick band labelled with the consolidated responsibility.

### Mass diagram (good for "interface as wide as implementation")

Two rectangles per module — one for interface surface area, one for implementation. Before: interface rectangle is nearly as tall as the implementation rectangle (shallow). After: interface rectangle is short, implementation rectangle is tall (deep).

### Call-graph collapse

Before: a tree of function calls rendered as nested boxes. After: the same tree collapsed into one box, with the now-internal calls shown faded inside it.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. Serif optional for headings (`font-serif` works well with stone/slate).
- Colour sparingly: one accent (emerald or indigo) plus red for leakage and amber for warnings.
- Keep diagrams ~320px tall so before/after sits comfortably side by side without scrolling.
- Use `text-xs uppercase tracking-wider` for module labels inside diagrams — they should read as schematic, not as UI.
- The only scripts are the Tailwind CDN and the Mermaid ESM import. The report is otherwise static — no app code, no interactivity beyond Mermaid's own rendering.

## Top recommendation section

One larger card. Candidate name, one sentence on why, anchor link to its card. That's it.

## Tone

Plain, colloquial Chinese — this report is read by someone who has NOT necessarily read the `/codebase-design` glossary. Concision is not an excuse to leave a term unexplained.

**Chinese glossary** — use these exact Chinese terms (not the raw English words, not invented alternatives), each with a short parenthetical gloss the *first time it appears in a given card* (later uses in the same card can drop the gloss):

- **模块**（module，一段有对外接口和内部实现的代码，大到一个子系统、小到一个函数都算）
- **接口**（interface，调用者需要知道的一切：怎么调用、有什么限制、会不会出错——不只是函数签名）
- **实现**（implementation，接口背后真正做事的代码）
- **深/浅**（depth，浅模块的接口几乎和实现一样复杂，等于没帮调用者省事；深模块用一个简单接口藏住大量实现）
- **接缝**（seam，一个可以在不改动当前代码的情况下替换掉的位置，比如换成测试用的假实现）
- **适配器**（adapter，插在接缝上的某个具体实现，比如生产用的真实版本或测试用的假版本）
- **杠杆**（leverage，调用者只需学一个简单接口，就能用到背后大量能力，一次投入多次收益）
- **局部性**（locality，改动、bug 和排查集中在一个模块内，不会散落到调用它的各处）

**Do not** leave terms in English, and do not substitute with vaguer everyday words like "组件"、"模块"（当你指别的东西时）、"接口"（当你只想说函数签名时）、"边界". Precision matters — the gloss is what makes precision readable, not a reason to drop the term.

**Phrasings that fit the style** (Chinese, term + gloss, then plain consequence):

- "Order 接收模块是浅模块（接口几乎和实现一样复杂，没有把复杂度藏起来）。"
- "计价逻辑跨接缝泄漏了，改一处要改两边。"
- "深化后：一个接口，一个测试的地方。"
- "两个适配器才值得留这个接缝：生产环境用真实版本，测试用内存假版本。"

**Wins bullets** name the gain using the Chinese glossary terms, in plain language, ≤6 Chinese words worth of content: *"局部性：bug 只会出现在一个模块里"*、*"杠杆：一个接口服务 N 个调用点"*、*"接口变小，实现吸收了原来的包装层"*. Don't write *"更好维护"* or *"代码更干净"* — too vague to verify.

No hedging, no throat-clearing, no "值得注意的是…". If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it. Every glossary term must carry its gloss on first use in a card — a term without a gloss is the failure mode this section exists to prevent.
