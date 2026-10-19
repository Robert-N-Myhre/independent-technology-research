---
title: "Project Boundary — Governing Consequential Automation"
description: "Architecture case study on using implementation to test how enterprise AI can consume governed context and influence consequential infrastructure under deterministic evidence, authority, and recovery controls."
---

# Project Boundary — Governing Consequential Automation

**Context:** Independent architecture research  
**Status:** Research prototype complete for its intended investigation; production concerns remain open  
**Architecture domains:** Agentic AI · Infrastructure Automation · Security & Context Integrity · Authority & Authorization · Evidence & Provenance · Recovery & Operational Safety · Policy & Governance

<p class="boundary-lede"><strong>I used implementation to test and evolve a Reference Architecture for a consequential form of enterprise AI: systems whose reasoning may influence production infrastructure.</strong></p>

<div class="boundary-question-grid" role="group" aria-label="The two architecture questions that shaped Boundary">
  <section class="boundary-question-card boundary-question-card--input">
    <p class="boundary-question-card__eyebrow">Input / Context integrity</p>
    <h3>What do I trust the model to consume?</h3>
    <p>Which information is safe, attributable, and sufficiently governed to influence probabilistic reasoning?</p>
    <p class="boundary-question-card__concepts">provenance · screening · field trust · acquisition</p>
  </section>
  <section class="boundary-question-card boundary-question-card--consequence">
    <p class="boundary-question-card__eyebrow">Consequence / Authority</p>
    <h3>What do I trust the model's reasoning to influence?</h3>
    <p>Where does probabilistic judgment end, and where must deterministic control and accountable authority begin?</p>
    <p class="boundary-question-card__concepts">verification · authorization · blast radius · recovery</p>
  </section>
</div>

## Two questions before implementation

I wanted to explore a different class of enterprise AI use: not assisting with documents, communications, or other knowledge work, but influencing changes to production infrastructure. Server patching became the proving ground.

I created a Reference Architecture before I wrote any code. As it evolved in the days before implementation, the two questions above came to dominate the design.

Those questions protected opposite sides of the reasoning path.

On the consequence side, the architecture separated reasoning, control, and authority. Models could interpret, analyze, propose, and record. Deterministic orchestration governed progression. Human decisions and policy governed what the system was allowed to cause. Agents did not own estate mutation.

On the input side, the architecture treated model context as a security boundary of its own. Free-form content could be malformed, misleading, or adversarial even when it arrived through a legitimate workflow. By the final pre-implementation version of the Reference Architecture, that concern had become an explicit **context-integrity plane** alongside the **patch-safety plane**.

The second question mattered enough that, before implementation began, I had already run a small supporting experiment to see whether a dedicated security/classification model and the reasoning model could coexist on the same GPU. I wanted to know semantic screening was a practical control before relying on it in the design.

The screener itself was narrow by design: bounded output, no tools, no memory, and no mutation authority. It contributed risk evidence about content entering reasoning and had no say in what the system did with it.

By the time implementation began, the architecture protected two things separately: the integrity of information entering reasoning, and the authority governing what that reasoning could cause.

Then I built enough of that architecture to try to break its assumptions.

<div class="boundary-research-strip" aria-label="Architecture research method">
  <div class="boundary-research-strip__flow">
    <span>Reference Architecture</span>
    <span aria-hidden="true">→</span>
    <span>Implementation</span>
    <span aria-hidden="true">→</span>
    <span>Evidence</span>
    <span aria-hidden="true">→</span>
    <span>Revise / Preserve / Stop</span>
  </div>
  <p><strong>Implementation was the research instrument, not the destination.</strong></p>
</div>

## When the implementation stopped expressing the architecture

The first implementation exercised the architecture one vertical slice at a time. It confirmed several principles early:

- models should produce artifacts rather than actions;
- screening results should contribute evidence without owning consequence;
- model output should not create affirmative safety;
- human approval needs explicit scope;
- ambiguous outcomes should reconcile before another mutation;
- validation must mean more than "the automation job completed."

But local fixes accumulated. Each one could be defended individually, while the system as a whole became harder to reason about, so I restarted the implementation.

I kept the code, tests, findings, and failed assumptions as evidence, and dropped any obligation to preserve an implementation because effort had already gone into it. The second build carried forward the architectural invariants that had survived and restarted from a thin end-to-end walking skeleton.

The question behind that restart recurred throughout Boundary:

> **I preserved the evidence and restarted the implementation. The question was: is the problem in the architecture, or only in the way I realized it?**

## Architecture at a glance

The evolved architecture retained the two-sided protection model, but both sides became more explicit.

<div class="boundary-architecture" role="img" aria-label="Flow showing operator and enterprise information entering through governed preparation or acquisition, including a deterministic evidence floor, then provenance, field classification, and screening before probabilistic reasoning. A proposal is deterministically verified before an independent authority boundary. Work without mutation authority becomes Advisory; transaction or standing authorization can proceed to controlled consequence, conditional governed recovery, and validation.">
  <section class="boundary-architecture__zone boundary-architecture__zone--context">
    <p class="boundary-architecture__zone-label">Context</p>
    <div class="boundary-architecture__node boundary-architecture__node--source">
      <strong>Operator Input · Enterprise Sources · External Content</strong>
    </div>
    <div class="boundary-architecture__arrow" aria-hidden="true">↓</div>
    <div class="boundary-architecture__node boundary-architecture__node--control">
      <span class="boundary-architecture__eyebrow">Governed context entry / acquisition</span>
      <strong>Prepared Input · Deterministic Source Access</strong>
      <span>includes the gate-required acquisition floor</span>
    </div>
    <div class="boundary-architecture__arrow" aria-hidden="true">↓</div>
    <div class="boundary-architecture__node boundary-architecture__node--boundary">
      <span class="boundary-architecture__eyebrow">Context integrity boundary</span>
      <strong>Provenance · Field Classification · Screening</strong>
    </div>
  </section>

  <div class="boundary-architecture__connector" aria-hidden="true">↓</div>

  <section class="boundary-architecture__zone boundary-architecture__zone--reasoning">
    <p class="boundary-architecture__zone-label">Reasoning &amp; deterministic control</p>
    <div class="boundary-architecture__node boundary-architecture__node--reasoning">
      <span class="boundary-architecture__eyebrow">Probabilistic</span>
      <strong>Reasoning</strong>
    </div>
    <div class="boundary-architecture__arrow" aria-hidden="true">↓</div>
    <div class="boundary-architecture__node">
      <strong>Plan / Proposal</strong>
    </div>
    <div class="boundary-architecture__arrow" aria-hidden="true">↓</div>
    <div class="boundary-architecture__node boundary-architecture__node--control">
      <span class="boundary-architecture__eyebrow">Deterministic</span>
      <strong>Deterministic Verification</strong>
    </div>
  </section>

  <div class="boundary-architecture__connector" aria-hidden="true">↓</div>

  <section class="boundary-architecture__zone boundary-architecture__zone--authority">
    <p class="boundary-architecture__zone-label">Authority &amp; consequence</p>
    <div class="boundary-architecture__node boundary-architecture__node--authority-boundary">
      <span class="boundary-architecture__eyebrow">Independent authority boundary</span>
      <strong>What is the system allowed to cause?</strong>
      <span>Human act or prior governed policy supplies authority; model judgment does not.</span>
    </div>

    <div class="boundary-authority-split">
      <div class="boundary-authority-stop">
        <span class="boundary-authority-branch-label">No applicable mutation authority</span>
        <div class="boundary-authority-lane boundary-authority-lane--advisory">
          <strong>Advisory</strong>
          <span>Human-usable outcome</span>
          <span>No automated consequence</span>
        </div>
      </div>

      <div class="boundary-authority-go">
        <span class="boundary-authority-branch-label">Governed mutation authority exists</span>
        <div class="boundary-authority-lanes">
          <div class="boundary-authority-lane boundary-authority-lane--authorized">
            <span class="boundary-architecture__eyebrow">Exact human act</span>
            <strong>Transaction Authorization</strong>
          </div>
          <div class="boundary-authority-lane boundary-authority-lane--authorized">
            <span class="boundary-architecture__eyebrow">Prior governed act</span>
            <strong>Standing Policy Authorization</strong>
          </div>
        </div>

        <div class="boundary-authorized-flow">
          <div class="boundary-architecture__arrow" aria-hidden="true">↓</div>
          <div class="boundary-architecture__node boundary-architecture__node--consequence">
            <strong>Controlled Consequence</strong>
          </div>
          <div class="boundary-architecture__arrow" aria-hidden="true">↓</div>
          <div class="boundary-post-consequence">
            <div class="boundary-post-consequence__path">
              <span class="boundary-architecture__eyebrow">Normal completion</span>
              <strong>Continue to validation</strong>
            </div>
            <div class="boundary-post-consequence__path boundary-post-consequence__path--recovery">
              <span class="boundary-architecture__eyebrow">Stop / recovery path</span>
              <strong>Governed Stop / Recovery</strong>
              <span>separate authority when recovery mutates state</span>
            </div>
          </div>
          <div class="boundary-architecture__arrow" aria-hidden="true">↓</div>
          <div class="boundary-architecture__node boundary-architecture__node--validation">
            <strong>Deterministic Validation</strong>
            <span>built / tested · not operator-accepted</span>
          </div>
        </div>
      </div>
    </div>
  </section>
</div>

The prototype demonstrated that path through controlled execution and governed recovery in a simulated estate. Deterministic validation was built and tested but did not complete its operator acceptance walk. Later anomaly validation, final outcome/learning, and the real execution adapter remained open.

<p class="boundary-evidence-key" aria-label="Architecture evidence-depth summary">
  <strong>Evidence depth:</strong>
  demonstrated through governed recovery · deterministic validation built/tested but not operator-accepted · later anomaly/final-outcome/learning stages remained open
</p>

That distinction between architectural intent and demonstrated depth is part of the result.

## Four architecture turns

<p class="boundary-turn__label">Architecture turn 01</p>

### A deterministic gate can still be blind

The original architecture was right that consequential decisions should be owned by deterministic controls. What it missed was that a deterministic decision is only as good as the evidence it is guaranteed to receive.

In one implementation path, a reasoning agent decided which operational evidence to retrieve. A deterministic verifier then evaluated the proposal using the evidence that happened to be available. On paper, the final decision was deterministic.

In practice, one specimen exposed the flaw. A governed source and current observed state disagreed, but the agent retrieved only one side of the comparison. The conflict gate worked exactly as designed and still missed the conflict because the evidence required to trigger it never arrived.

A second case showed the same weakness from another direction. The agent could spend its retrieval attempts on useful but nonessential context and still fail to collect the minimum facts the control required.

So I changed the architecture. For every gate-required input, the deterministic side became responsible for obtaining a minimum evidence set before probabilistic reasoning could proceed. The model could still investigate beyond that floor, but it no longer decided whether the control received the evidence necessary to enforce its own rule.

I called that minimum the **Deterministic Acquisition Floor**.

> **A control must own reliable access to the evidence required to enforce itself.**

The project exposed a broader lesson: authority can leak through evidence dependencies even when the final decision point is deterministic.

<p class="boundary-turn__label">Architecture turn 02</p>

### A trusted system does not make every field trustworthy

The second question, whether I trust the information going into the model, did not disappear once a screener existed. Implementation made it harder.

The pre-implementation design treated unstructured natural language as an injection surface and gave the screener several points where hostile content could enter reasoning. As new context paths appeared, I kept asking the same question: could content entering here be malicious or malformed, and so unsafe to treat as instruction?

The first implementation reinforced one boundary quickly. A conservative screening result could not be allowed to become its own authority path. The screener could classify risk, but deterministic policy still had to own the consequence.

The deeper problem was the architecture's original source-level trust model. Structured enterprise systems were treated as trusted and typed, while free-form material was screened. That distinction was useful, but too coarse. A governed system may tightly control an identifier, lifecycle state, or policy value while also carrying operator-entered notes, imported descriptions, or script-generated prose.

Calling the whole source "trusted" collapses two different questions: who is authorized to write into the system, and whether a specific field is safe to admit into a reasoning context.

The architecture moved from source-level trust to field/content-level trust. Typed, enumerable fields could follow the governed structured path. Free-form prose from any source, including an otherwise governed enterprise system, followed a screened, provenance-preserving path.

<div class="boundary-trust-evolution" role="img" aria-label="Trust evolution from pre-implementation screening of unstructured content, through the discovery that governed sources contain mixed-trust fields, to field-level trust enforced by governed acquisition, provenance, screening, withholding, and explicit evidence gaps.">
  <div class="boundary-trust-step">
    <span class="boundary-trust-step__label">Pre-implementation</span>
    <strong>Unstructured content is an injection surface</strong>
    <span>Screen potentially hostile natural-language context before it influences reasoning.</span>
  </div>
  <div class="boundary-trust-arrow" aria-hidden="true">→</div>
  <div class="boundary-trust-step boundary-trust-step--pressure">
    <span class="boundary-trust-step__label">Implementation pressure</span>
    <strong>A governed source can still contain untrusted prose</strong>
    <span>Source authority and semantic trust are different properties.</span>
  </div>
  <div class="boundary-trust-arrow" aria-hidden="true">→</div>
  <div class="boundary-trust-step boundary-trust-step--revision">
    <span class="boundary-trust-step__label">Architecture revision</span>
    <strong>Trust follows the field / content</strong>
    <span>Typed fields and free-form prose take different governed paths.</span>
  </div>
  <div class="boundary-trust-arrow" aria-hidden="true">→</div>
  <div class="boundary-trust-step boundary-trust-step--operationalized">
    <span class="boundary-trust-step__label">Operationalized</span>
    <strong>Context Acquisition Gateway</strong>
    <span>Classify · preserve provenance · screen prose · withhold flagged content · record the gap.</span>
  </div>
  <p class="boundary-trust-evolution__caption"><strong>Screening remained a control; the architecture around trust became broader.</strong></p>
</div>

The change also moved acquisition behind a governed boundary. The **Context Acquisition Gateway** controlled the tool surface, classified returned fields, preserved provenance, screened prose, and withheld flagged content. When it withheld something, it recorded the gap explicitly.

> **Governance of a system establishes write authority. Semantic trust in each field it contains has to be established separately.**

Screening remained important, now as one mechanism inside a broader context-integrity architecture.

I later generalized that work into a separate Reasoning Context Admission architecture. The abstraction came after Boundary, using the project's evidence as the basis for that generalization.

<p class="boundary-turn__label">Architecture turn 03</p>

### Stopping safely is not enough if the operator is stranded

Boundary was designed to fail closed. If evidence was insufficient or authority was missing, the system should stop. That remained correct.

Operator use of the live prototype exposed that a safe stop could still be operationally incomplete. A workflow could refuse to proceed for the right reason and leave the human with no clear, governed next step.

That changed the design in two ways.

First, Advisory had been a terminal label and became an operating outcome. When automation lacked authority, the system needed to hand a human enough verified context to understand the proposed work, why automation stopped, what evidence existed, and what legitimate action could follow.

Second, once the prototype could change simulated state, recovery exposed the same issue on the other side of consequence. The original approval covered the planned action only. Calling a new mutation "recovery" did not bring it under that approval, so I treated recovery as a separate governed authority event. A human could authorize a recovery strategy already bound to the verified plan, but could not invent a new recovery action at the stop.

The prototype exercised three different outcomes: recovery refused because mutation could not be proven, recovery completed under separate authorization, and recovery itself failed and stopped without further mutation.

> **If recovery can change consequential state, it needs its own scope, evidence, identity, and accountable authorization.**

Fail-closed behavior was necessary, and governed recovery made it usable.

<p class="boundary-turn__label">Architecture turn 04</p>

### The enterprise grants the AI its autonomy

The original architecture treated autonomy as tiered and earned. Implementation forced that idea into a more explicit authority model.

Boundary separated four things:

- **workload tier:** the maximum authority a class of work may receive;
- **current autonomy lane:** its present operating posture;
- **transaction authorization:** a human authorizes this exact plan;
- **standing authorization:** a prior governed act authorizes a bounded class of operations.

The model owns none of those.

A later test exposed another weakness. One standing-policy envelope could govern one class correctly, but it could not represent several workload classes operating under different authority postures at the same time. The design moved to a non-overlapping policy portfolio.

With the portfolio, one orchestrator could demonstrate three different outcomes at once:

- a workload operating under prior standing authority;
- a workload producing a human-usable Advisory outcome;
- a workload with no applicable authority refusing before execution.

A simple automatic/manual switch cannot express that.

The test also exposed a limitation. If lower-authority operation is supposed to help a workload earn greater autonomy later, those lower-authority paths must produce the evidence future promotion policy will consume. Boundary did not complete that evidence-production loop.

The project demonstrated governed autonomy lanes and explicit promotion authority, but not a complete earned-autonomy lifecycle.

> **Autonomy is a governed operating posture over bounded work. Promotion is an enterprise authorization decision made on evidence.**

## What the implementation demonstrated

Boundary exercised evidence on both sides of reasoning.

<div class="boundary-question-grid boundary-question-grid--compact" role="group" aria-label="Evidence questions exercised by the implementation">
  <section class="boundary-question-card boundary-question-card--input">
    <p class="boundary-question-card__eyebrow">Before consequence</p>
    <p><strong>What information is safe enough, well-enough classified, and sufficiently evidenced to influence reasoning?</strong></p>
  </section>
  <section class="boundary-question-card boundary-question-card--consequence">
    <p class="boundary-question-card__eyebrow">Across / after consequence</p>
    <p><strong>What evidence must exist before that reasoning is allowed to influence action, and what proves what happened afterward?</strong></p>
  </section>
</div>

The semantic screener was the first real model path implemented in the original build, with adversarial canaries used to test the bounded contract. Later work showed both its value and its limits: screening could contribute useful risk evidence, but it did not make source trust simple, it could miss hostile content, and its verdict still had to remain subordinate to deterministic policy.

Execution exposed a parallel evidence problem after action. I ran real execution probes before defining the execution contract. Those probes showed that automation-tool status fields could not be treated as self-interpreting evidence. In one case, a result reported that change had occurred even though the attempted rollback had failed before changing the host. Other probes exposed similar ambiguity around reachability, transaction identity, and rollback behavior.

So I built the execution contract around the semantics I had measured.

<div class="boundary-evidence-key" aria-label="Evidence status key">
  <span class="boundary-evidence-chip"><strong>Demonstrated</strong> — exercised through the working prototype / accepted path</span>
  <span class="boundary-evidence-chip"><strong>Probed</strong> — semantics derived from operator-run real probes</span>
  <span class="boundary-evidence-chip"><strong>Built / Tested</strong> — implemented and tested, but not fully operator-accepted</span>
  <span class="boundary-evidence-chip"><strong>Open</strong> — target or production obligation, not demonstrated</span>
</div>

<div class="boundary-evidence-table" role="region" aria-label="Boundary implementation evidence status" tabindex="0">
<table>
  <thead>
    <tr>
      <th scope="col">Area</th>
      <th scope="col">Status</th>
      <th scope="col">Evidence</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>semantic screening / governed context controls</td><td><strong>Demonstrated</strong></td><td>exercised with real-model canaries; refined through field-level trust and governed acquisition</td></tr>
    <tr><td>deterministic evidence floor</td><td><strong>Demonstrated</strong></td><td>executable and mutation-tested guards</td></tr>
    <tr><td>transaction and standing authorization</td><td><strong>Demonstrated</strong></td><td>exercised end to end</td></tr>
    <tr><td>multi-lane autonomy + Advisory</td><td><strong>Demonstrated</strong></td><td>exercised end to end</td></tr>
    <tr><td>execution-result semantics</td><td><strong>Probed</strong></td><td>derived from operator-run real probes</td></tr>
    <tr><td>controlled execution + governed recovery</td><td><strong>Demonstrated</strong></td><td>exercised end to end in the simulated estate</td></tr>
    <tr><td>deterministic validation</td><td><strong>Built / Tested</strong></td><td>implemented and tested; not operator-accepted</td></tr>
    <tr><td>anomaly validation / final outcome / learning</td><td><strong>Open</strong></td><td>not built</td></tr>
    <tr><td>production HA, policy lifecycle, evidence custody, segregation of duties</td><td><strong>Open</strong></td><td>production obligations not demonstrated</td></tr>
  </tbody>
</table>
</div>

> **Related artifacts:** [Inspect the evidence](https://github.com/Robert-N-Myhre/project-boundary-evidence){ target="_blank" rel="noopener noreferrer" } supporting the claims above, or [read the distilled Project Boundary Reference Architecture](../reference-architectures/governed-agentic-infrastructure-operations.md) that captures the reusable architecture resulting from the investigation.

A successful run proves that a path occurred. A mutation test can show that removing a guard changes the result. A canary can show that a screening path reacts to a known specimen. None of those establishes production reliability, operational rates, model accuracy, or production safety.

## Why I stopped

I continued building while the next increment still had an architecture question worth answering. Execution and recovery met that test. They exposed new questions about evidence, consequence, and recovery authority, so I built them.

By October 8, the remaining work was still legitimate engineering: complete operator acceptance of deterministic validation, build the later validation/learning stages, integrate a real execution adapter, and address production concerns such as HA, policy lifecycle, evidence custody, and segregation of duties. But those increments were about completing the product, and they no longer challenged the architecture.

> **I stopped when the next increments were increasingly about completing the product rather than challenging the architecture.**

I ended the project when further implementation was no longer likely to falsify or change the design. Boundary is unfinished as a production automation platform. As an architecture investigation, it had reached diminishing returns.

## AI collaboration and architecture decision accountability

AI models and coding agents were involved throughout Boundary. They helped surface defects, challenge assumptions, develop alternatives, implement experiments, and generate tests.

I treated AI participation and architecture decision accountability as separate things. The project distinguished discovering a problem, developing alternatives, selecting the governing direction, implementing the response, and validating the result.

Where the surviving record supports explicit ownership, I use it. Where it does not, I do not reconstruct certainty after the fact.

Boundary's own architecture makes the same separation. Being able to propose or implement something is different from being accountable for deciding what should govern consequential behavior.

## Claims and limits

<div class="boundary-claims-grid">
  <section class="boundary-claim-panel boundary-claim-panel--demonstrates">
    <h3>What Boundary demonstrates</h3>
    <ul>
      <li>probabilistic reasoning can be separated from consequence-bearing authority;</li>
      <li>untrusted natural-language context can be screened through a bounded function without granting that function decision authority;</li>
      <li>trust can be governed at field/content level;</li>
      <li>deterministic controls can own the evidence required to enforce themselves;</li>
      <li>autonomy can be represented as an enterprise authority posture;</li>
      <li>safe stops can remain operable without widening authority;</li>
      <li>recovery can be governed as a separate consequential act;</li>
      <li>implementation can be used to falsify an architecture as well as realize it.</li>
    </ul>
  </section>
  <section class="boundary-claim-panel boundary-claim-panel--limits">
    <h3>What Boundary does not establish</h3>
    <ul>
      <li>reliable detection of all hostile or misleading content;</li>
      <li>production-grade HA;</li>
      <li>enterprise policy lifecycle, evidence custody, or segregation of duties;</li>
      <li>statistically justified autonomy thresholds;</li>
      <li>production model accuracy;</li>
      <li>complete validation / outcome / learning depth;</li>
      <li>a universal enterprise operating model for agentic automation.</li>
    </ul>
  </section>
</div>

## What applies beyond Boundary

<div class="boundary-closing-questions" role="group" aria-label="The two questions Boundary carries forward">
  <p><strong>What do I trust the model to consume?</strong></p>
  <p><strong>What do I trust the model's reasoning to influence?</strong></p>
</div>

The implementation context was agentic infrastructure automation, but the architectural conclusions are broader:

1. Capability and authority are different properties.
2. Source authority and content trust are different properties.
3. A control must own the evidence required to enforce itself.
4. Autonomy is a governed operating posture over bounded work.
5. Safe failure is incomplete without governed recovery.

The project also reinforced a working method I use beyond AI systems:

> **Implementation is useful architecture work when it is built deeply enough to make assumptions falsifiable.**

Boundary did not become valuable because I kept making the automation platform more complete. It became valuable because implementation forced me to get more precise about both sides of the reasoning boundary: what I was willing to let shape the model's judgment, and what I was willing to let that judgment cause.

