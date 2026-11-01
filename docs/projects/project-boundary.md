---
title: "Project Boundary: Where AI Reasoning Ends and Operational Authority Begins"
description: "An architecture research case study in context integrity, deterministic evidence, governed authority, and recovery for consequential AI infrastructure operations."
---

# Project Boundary: Where AI reasoning ends and operational authority begins

*Exploring how AI agents can participate in consequential infrastructure operations without inheriting uncontrolled authority over the systems they influence.*

Enterprise infrastructure automation is something we've been doing for decades. We have mature tools for deploying configurations, provisioning infrastructure, applying software updates, and performing operational tasks across thousands of systems. The questions change when we introduce AI into the decisions surrounding those operations.

An AI agent can interpret information from multiple sources, identify relationships, evaluate alternatives, and recommend a course of action. Those capabilities could be valuable in enterprise operations, particularly where engineers spend considerable time investigating conditions before deciding what automation should do. But recommending an action is different from having the authority to execute it.

That difference became the foundation for Project Boundary, an independent architecture research project exploring how probabilistic AI reasoning could participate in infrastructure operations while remaining subject to deterministic controls and enterprise governance.

The project began with a hypothetical enterprise Linux patching scenario, with no customer environment or production deployment involved. The scenario was realistic enough to investigate the architectural decisions that come into play once AI can influence consequential infrastructure changes. My objective was to determine what kind of architecture would make AI participation in those operations governable.

As the investigation developed, two questions came to define almost everything that followed:

**What do I trust the model to consume?**

**What do I trust the model's reasoning to influence?**

These sound like straightforward questions. In practice, they led to some of the project's more useful architectural findings.

## Starting with the architecture

I chose Linux patching because it illustrates a familiar operational problem. Tools such as Ansible Automation Platform can execute patching workflows reliably, package management systems can identify available updates, and monitoring systems can provide operational telemetry. None of those capabilities resolves the surrounding decisions. Which systems should be patched together? What application dependencies exist? Could taking a particular group of servers offline affect service availability? What evidence should establish that the operation succeeded? And if something goes wrong, how does the system decide what happens next?

These questions frequently require information from multiple enterprise systems, along with operational knowledge that may not be represented in a single authoritative source. This is where AI reasoning could help. Working alongside existing automation, an agent might interpret operational context, identify dependencies, recommend maintenance groups, or investigate unexpected outcomes. Allowing an agent to perform that reasoning does not mean it should also decide what actions are authorized.

I began by developing a Reference Architecture that separated three responsibilities. AI components capable of interpreting context and developing proposals would handle reasoning. Deterministic orchestration would handle control, verifying evidence, enforcing constraints, and governing workflow progression. Authority would remain with explicitly governed human decisions or previously approved enterprise policies. The execution systems would continue performing the actual infrastructure changes.

This separation established an architectural principle that remained consistent throughout the investigation:

**AI may make decisions within an authorized boundary, but it cannot independently define, expand, or waive that boundary.**

The principle carries particular weight with probabilistic systems. A model can produce a technically plausible recommendation that is incomplete, incorrect, or based on misleading information. Improving the model may reduce those risks, but it doesn't eliminate the need for independent operational controls. The architecture therefore needed to treat AI output as something to evaluate rather than something that automatically carried authority.

## The other side of the problem: what enters the model?

Much of the early architecture focused on what an AI agent should be allowed to do. As I worked through the design, I became increasingly interested in what the agent was being allowed to know, or more precisely, what information it was allowed to use as the basis for operational reasoning.

Enterprise AI systems rarely operate on carefully curated prompts alone. They consume tickets, inventory records, runbooks, monitoring results, application descriptions, configuration details, and information retrieved through tools. Some of that information is structured and governed. Some is free-form text. Some may be outdated or contradictory, and some could contain malicious instructions intended to change the behavior of the model.

An agent could therefore reach an unsafe conclusion without ever receiving direct authority to execute an action. The architecture might correctly prevent the model from calling an execution tool, yet still permit the model to influence the inputs used by the system that makes the final decision. I began treating the information entering AI reasoning as a separate security boundary.

The initial design included a dedicated semantic screening function to evaluate potentially hostile natural-language content before admitting it into the reasoning context. Before implementing the larger architecture, I also conducted a small supporting experiment to investigate whether a dedicated screening model and a reasoning model could coexist on the same GPU. The purpose was to explore the practicality of model-based screening as an architectural component.

The results were encouraging, with some qualifications. On the tested NVIDIA RTX 4000 Ada, the two models occupied 16,559 MiB of the GPU's measured 20,475 MiB total, leaving 3,916 MiB free on that basis. Co-residency was practical, but it didn't provide useful parallel inference in the tested configuration. Across a 17-case comparison, the pipelined arrangement was about 1.2% slower per case than sequential execution. That gave me a workable basis for local semantic screening without requiring a separate GPU for each model. The [single-GPU guardrail research](https://github.com/Robert-N-Myhre/prompt-guardrail-single-gpu) includes the measurements and their limitations.

The screening function was intentionally constrained, with bounded outputs, no operational tools, no persistent memory, and no authority to modify infrastructure. It could contribute evidence about potentially unsafe content, but it could not decide what operational consequences should follow. That limit mattered from the beginning, and implementation would make it matter more.

## Using implementation to challenge the architecture

After developing the Reference Architecture, I began implementing it through a series of vertical slices. I wanted enough working functionality to test whether the architectural boundaries behaved the way I expected them to, and a complete enterprise automation platform was never the intention.

That approach produced useful findings early. The implementation reinforced the need for AI models to produce proposals and evidence rather than executable commands. It also demonstrated why safety decisions could not depend solely on affirmative judgments from an AI model. Human authorization needed to bind to a specific operation, ambiguous execution outcomes needed to be reconciled before another mutation, and validation needed to establish more than whether an automation job returned a successful status. Each of these findings influenced the implementation.

Eventually I encountered a different problem. The first implementation had accumulated a series of local changes. Each change addressed a legitimate issue, but the combined result was becoming increasingly difficult to reason about. I could have continued fixing it, but I was no longer confident that doing so was the best way to evaluate the architecture, so I restarted.

I preserved the code, tests, findings, and failed assumptions from the first implementation as research evidence. The second implementation began with a thinner end-to-end path, carrying forward the architectural principles that had survived the first investigation.

This became one of the project's clearer methodological lessons. An implementation can fail to express an architecture without necessarily proving the architecture itself is wrong. Conversely, successfully executing a workflow does not establish that the intended architectural controls are actually present. The responsibility is to tell those conditions apart and revise the appropriate part of the system. For Boundary, implementation was a research instrument, and preserving the architecture's testability mattered more than preserving every line of code.

## When a deterministic control doesn't have enough evidence

One of the project's main findings involved evidence acquisition. The architecture already required deterministic verification before an operation could proceed. The reasoning agents could propose actions, but a separate control component would determine whether the necessary conditions had been satisfied. That seemed like a reasonable separation of responsibilities. The problem appeared when I examined how the evidence reached the verifier.

In one implementation path, the reasoning agent was responsible for deciding which operational information to retrieve. The deterministic verifier then evaluated the proposal against the information that had been collected. The decision itself was deterministic, but the evidence available to make that decision depended on the agent's earlier choices.

A test involving a simulated server, app-db-02, exposed the weakness. The governed Source of Truth reported RHEL 9.2, while the current observed state reported RHEL 9.4. The architecture contained a deterministic control intended to identify precisely that kind of conflict, but the agent retrieved only the Source-of-Truth record. The control accepted the scope because it never received the live facts needed to identify the disagreement.

The problem went beyond retrieval quality. It was architectural:

**A deterministic decision is not necessarily a trustworthy decision if a probabilistic component controls whether the required evidence is available.**

I revised the architecture to introduce what I called the *Deterministic Acquisition Floor*. Under this model, the deterministic side of the system becomes responsible for obtaining a minimum required set of evidence before consequential reasoning can proceed. Agents may still investigate additional information. They can retrieve context, explore relationships, and develop more informed recommendations. What they cannot do is determine whether the mandatory evidence required by a governing control will exist. This creates a clearer separation between discretionary investigation and required verification.

After introducing the floor, I repeated the conflict scenario. This time, the orchestrator acquired both records before the agent began reasoning, and the control correctly referred the disagreement for human review. Additional runs showed that a missing host produced an insufficient-evidence referral, while a clean, corroborated record could still proceed normally. The [before-and-after evidence](https://github.com/Robert-N-Myhre/project-boundary-evidence/tree/main/evidence/acquisition-floor) is published with the project's supporting research.

The broader lesson extends beyond infrastructure automation. Whenever an AI agent participates in a governed decision, we need to examine who makes the final decision and who controls the information required to make it correctly. A control that doesn't own its evidence dependencies may have less authority than its architecture suggests.

## A trusted enterprise system can contain untrusted information

The next finding developed from the other governing question: what information should the model be allowed to consume? The early context-integrity architecture distinguished between structured information from governed enterprise systems and unstructured natural-language content. Structured information could follow a trusted, typed path, while free-form content would pass through additional screening. This was a useful starting point, but implementation exposed a limitation: a governed enterprise system does not necessarily contain uniformly trustworthy information.

Consider a configuration management database. Its identifiers, lifecycle states, or controlled classification fields might be governed through established enterprise processes, and those fields can have meaningful authority within the systems that use them. The same record might also contain an operator-entered description, imported notes, or text populated by an external integration. Those fields do not automatically inherit the same semantic trust as the structured data around them.

**The authority to write information into a system is not the same as establishing that the information is safe for an AI model to interpret as context.**

This changed the context-integrity architecture. Rather than assigning trust primarily at the source level, Boundary moved toward classifying information at the field and content level. Governed typed fields could follow an appropriate structured path. Free-form natural-language content, even when retrieved from a governed source, required additional handling.

The Context Acquisition Gateway served as the governed entry point for information retrieved from enterprise systems and external sources. It constrained available acquisition mechanisms and preserved provenance. The architecture distinguished governed typed fields from free-form prose, with the latter requiring additional handling and screening where appropriate.

The gateway was implemented and exercised in selected prototype paths, including the deterministic acquisition floor. The intake path also distinguished structured operator declarations from model-inferred fields and recorded their provenance. These results establish specific acquisition and provenance behaviors. The published evidence does not independently establish comprehensive field-level classification across every gateway source and content type.

The semantic screener remained useful, but its role became more narrowly defined. It was one mechanism within a broader context-integrity architecture, not a general-purpose authority capable of declaring content safe.

The first real-model screening evaluation used four test inputs, or canaries, consisting of three deliberately constructed malicious examples and one clean control. These inputs had known expected outcomes, allowing me to examine how the screening model and its surrounding controls responded to suspicious and legitimate content.

Using Cisco's Foundation-Sec 1.1 8B security-focused language model, all three malicious examples were identified as suspicious through model judgment. However, the clean example produced no model verdict and was conservatively classified as suspicious by the deterministic fail-safe.

Subsequent implementation work exposed a different problem. An integration defect caused the web-intake screening path to evaluate a hardcoded test fixture rather than the operator-submitted input. That defect was corrected, and a deliberately suspicious input was used to verify the wiring. The earlier four-canary evaluation had exercised the model directly, before this later intake defect was encountered; its results did not depend on the faulty web path. A screener can produce a plausible verdict while evaluating the wrong information.

These findings reinforced why screening couldn't serve as an independent authority boundary. The model's judgment, the integrity of the path delivering its inputs, and the deterministic handling of its results all required separate consideration. They also shaped subsequent research into how information should be evaluated and admitted into an AI system's reasoning context.

## When failing safely leaves the operator stranded

Another architectural lesson appeared when the prototype began exercising stop conditions and recovery behavior. Boundary was designed to fail closed. If a required condition was not satisfied, evidence was missing, or appropriate authority could not be established, the system should not proceed with an infrastructure change. That principle remained sound. But working with the prototype exposed an operational problem: stopping safely did not always leave the operator with a useful next step. A workflow could correctly refuse to proceed while providing little help in understanding how the situation should be resolved. From the perspective of preventing unauthorized changes, the control was working. From the perspective of operating an enterprise system, it was incomplete.

This led to a more explicit treatment of the *Advisory* outcome. Advisory had been simply a label indicating that automation would not proceed. It became a legitimate operating outcome that needed to produce useful information for a human. When automated mutation was not authorized, the system could still provide a verified description of the proposed operation, the evidence available, the reasons automation could not continue, and the legitimate next actions. The human remained responsible for any subsequent operational decision. This preserved the security boundary while making the system more usable.

Recovery exposed a related issue. Once Boundary could change state within its simulated infrastructure environment, I needed to determine what should happen when a controlled operation failed or produced an uncertain result. The original authorization covered the planned action. It did not automatically authorize a second action simply because that action was described as rollback or recovery. Recovery may be intended to restore a system to a safer state, but it can still involve consequential infrastructure mutation, so it requires its own scope, evidence, identity, and authorization.

I revised the design so recovery could occur under a separate governed authority event. A human could authorize an eligible recovery strategy already bound to the verified operation, but could not invent an unrestricted new recovery action at the point of failure.

The prototype exercised several recovery conditions. In one case, recovery was refused because the system could not prove that the original operation had changed anything. In another, a separately authorized recovery operation completed. A third case exercised recovery failure, where the system stopped without submitting the later undo step. These were simulated-estate tests, not evidence of production recovery reliability, but they were sufficient to exercise the difference between a safe stop and an authorized recovery action.

The resulting principle was straightforward:

**A safe stop prevents an unauthorized consequence. Governed recovery provides a controlled way forward without weakening that protection.**

Enterprise AI systems need both.

## Autonomy is something the enterprise grants

One of the original goals of Boundary was to explore whether agents could eventually operate with greater autonomy as the system accumulated evidence of reliable behavior. It is tempting to think of autonomy as a property of the model. A more capable model might appear to deserve greater freedom to act, while a less capable model might require more human oversight. But model capability and operational authority are different properties. An enterprise might permit a particular class of routine, low-risk maintenance to proceed under standing authorization while requiring explicit approval for changes involving critical systems. That difference may have little to do with the model itself. It depends on the operation, its risk, its governing policy, and the evidence available to support a decision.

Boundary therefore modeled autonomy as a governed operating posture associated with bounded classes of work, and several concepts needed to remain separate. A workload tier established the maximum level of authority permitted for a class of operations. An autonomy lane represented the current operating posture. Transaction authorization allowed a human to approve a specific verified operation. Standing authorization represented a prior governed decision allowing a defined class of operations to proceed without a new approval for every qualifying transaction. The AI model could propose work within those arrangements, but could not independently assign itself a different tier, promote its authority, or create a new policy.

Implementation also demonstrated why one global autonomy setting was insufficient. An early design could apply a standing-policy envelope to a single class of work, but it was less effective when several workload classes needed to operate under different authority conditions simultaneously. I revised the architecture around a non-overlapping policy portfolio.

The resulting prototype exercised multiple authority outcomes through the same orchestrator. One workload could operate under valid standing authorization. Another could produce an Advisory outcome for human consideration. A third, lacking applicable authority, would refuse to execute. Each outcome followed its own governing conditions, which offered a more expressive model than treating automation as simply manual or automatic.

There was still a limitation. The architecture could represent different autonomy postures and enforce their authority boundaries, but it did not complete the full evidence-production and promotion lifecycle required for a workload to earn greater autonomy over time. That would require representative operational evidence, explicit promotion criteria, and accountable enterprise governance. The intended direction remained clear: autonomy should increase when evidence justifies a governed decision, and an agent's assertion that it is ready is no such evidence.

## What did the implementation actually demonstrate?

As Boundary developed, I tried to keep architectural ideas, implemented mechanisms, observed behavior, and unfinished capabilities clearly separate. This became particularly important because different parts of the architecture reached different levels of validation.

The semantic screening path was exercised using Foundation-Sec and the canary inputs described earlier. Those tests helped evaluate the bounded screening contract, but did not establish complete protection against prompt injection. The Deterministic Acquisition Floor was exercised in before-and-after prototype runs and published executable negative-path tests, including cases where required evidence was missing or conflicting. Transaction authorization, standing authorization, multi-lane operation, and Advisory behavior were exercised through the working prototype. The simulated estate also supported controlled execution and governed recovery.

Execution semantics required a separate investigation. Before defining the execution contract, I ran isolated probes of Ansible execution and rollback behavior to understand what its returned results actually represented. These were distinct from the integrated Boundary prototype, which used a simulated infrastructure estate to exercise orchestration, authorization, execution, and recovery.

One probe showed that a result could report that a change had occurred even though an attempted rollback failed before making the corresponding change to the host. Other probes exposed ambiguity involving reachability, transaction identity, and rollback behavior. Those observations reinforced why an orchestration system should not treat a tool's returned status as authoritative evidence of the actual infrastructure outcome without understanding its semantics. The execution contract was subsequently designed around those observed behaviors, but the production execution adapter was not built.

The last operator-accepted integrated prototype milestone was M6.3, the controlled-execution and governed-recovery checkpoint, which exercised those paths in the simulated estate. In this context, operator acceptance means that I personally conducted an integrated prototype acceptance walk, submitting operations and reviewing their behavior against the expected results. It does not imply acceptance by an independent operator or production organization.

The next checkpoint, M6.4, implemented and tested deterministic post-operation validation, but its operator acceptance walk was never completed. The remaining validation and learning stages were not built. The following table summarizes where the research stopped.

| Architectural area | Evidence status |

|---|---|

| Semantic screening | Exercised using Foundation-Sec and four canary inputs; full screening corpus not public |

| Context Acquisition Gateway | Selected acquisition and provenance paths exercised; complete field-classification coverage not publicly established |

| Deterministic Acquisition Floor | Integrated before-and-after runs and published executable negative-path tests |

| Transaction and standing authorization | Both authorization paths exercised within the integrated prototype |

| Multiple autonomy lanes and Advisory | Three policy outcomes demonstrated: standing authorization, Advisory, and refusal when no policy applied |

| Execution-result semantics | Investigated through real execution probes |

| Controlled execution and governed recovery | Simulated-estate acceptance walk; recovery-decision tests and 14 mutation probes reported in the M6.3 record |

| Deterministic post-operation validation | Implemented and tested, but not operator-accepted |

| Anomaly validation, final outcome recording, and learning | Not started |

| Production HA, policy lifecycle, evidence custody, and segregation of duties | Open architectural and engineering obligations |

Each kind of evidence supports a limited claim. A successful prototype execution establishes that a particular path was exercised under particular conditions. A mutation test can demonstrate that a control changes the outcome when removed or altered. An adversarial canary can show that a screening mechanism responds to a particular class of malicious input. None of those independently establishes production reliability, statistically meaningful detection rates, or operational safety across an enterprise environment. Boundary demonstrated several architectural mechanisms, but it did not establish a production-ready autonomous infrastructure platform.

## Why I stopped building

As the project developed, I periodically considered what additional implementation would contribute to the architectural investigation. Some additions were clearly worthwhile. Execution and recovery, for example, exposed questions that could not be answered adequately through diagrams or design discussions alone, and I needed to exercise those paths to understand the relationships among evidence, authorization, actual mutation, and recovery. Other work would have improved the completeness of the implementation without necessarily providing substantial new architectural insight.

By October 8, 2026, the remaining work was still meaningful engineering. Deterministic validation needed to complete operator acceptance. The later anomaly validation, final outcome recording, and learning stages had not been started. A real execution adapter, high availability, enterprise policy management, evidence custody, and segregation of duties would all require additional work before contemplating production use. These were legitimate requirements, but they increasingly represented the work of completing a product rather than testing the assumptions that had motivated the research. I had reached a point where further implementation was unlikely to materially change my understanding of the architectural boundaries, so I stopped.

This reflects how I approach independent architecture research. A research prototype does not need to become a finished product to accomplish its purpose. Sometimes the useful result is an implementation that demonstrates an architectural principle. Sometimes it is a test that exposes a flawed assumption. Sometimes it is the discovery that an apparently reasonable design does not enforce the boundary it claims to enforce. Boundary had produced all three kinds of evidence, and continuing to build simply because more functionality could be added would not necessarily have improved the research.

## The role of AI in the research itself

The research involved AI both as the subject of investigation and as a collaborator in developing the implementation. I used AI models and coding agents to help examine assumptions, develop alternatives, implement experiments, generate tests, and identify potential defects. That collaboration contributed to the project's progress, but it also raised a familiar question about accountability.

Identifying a potential architectural problem is different from deciding how the architecture should change. Generating an implementation does not establish that it conforms to the intended design, and a successful test result does not automatically validate the claim that motivated the test. Throughout the project, I attempted to keep those activities separate.

The surviving research record does not always establish precise authorship for every idea or intermediate decision, and I don't think it would be appropriate to reconstruct certainty where that evidence doesn't exist. What mattered was maintaining accountability for the architectural direction and using implementation evidence to evaluate whether the resulting system behaved as intended.

This closely mirrors the problem Boundary was investigating. In both cases, AI can contribute analysis, identify possibilities, and generate technically useful artifacts. The ability to contribute does not, by itself, establish the authority to make consequential decisions.

## What Boundary taught me about enterprise AI

The most useful results of Boundary were not specific to Linux patching. The patching scenario provided the operational context needed to investigate a broader architectural problem: how to introduce probabilistic reasoning into environments where authority, evidence, and operational consequences must remain accountable.

Capability and authority must remain separate, because a model's ability to recommend an action does not establish that the action is permitted. Source authority and content trust are also different properties, and a governed enterprise system may contain information that requires additional validation before entering an AI reasoning context.

Deterministic controls need dependable evidence. A control cannot reliably enforce its requirements if another component can decide whether the evidence needed to trigger it is acquired.

Boundary demonstrated how different workloads could operate under distinct, governed authority conditions. It also established an architectural direction for earned autonomy, where future promotion would depend on representative operational evidence and explicit enterprise authorization. That promotion lifecycle remains unvalidated.

Recovery was another tested authority boundary. The simulated-estate runs exercised refusal, separately authorized recovery, and recovery failure without permitting uncontrolled additional mutation. Those results support the narrower recovery-control claim, not a claim of production recoverability.

The conclusion that interests me most concerns the relationship between architecture and implementation. Drawing a diagram that shows AI reasoning separated from deterministic controls is easy. Establishing that the separation holds is considerably harder when information is incomplete, sources disagree, tools return ambiguous results, or an operation fails halfway through. Those conditions reveal dependencies and assumptions that are not always visible in a conceptual design. Boundary reinforced the value of implementing enough of an architecture to challenge its assumptions and to determine whether it actually expresses the properties we expect it to have, even if the result never becomes a finished application.

## Where the investigation leaves us

I expect organizations to keep exploring ways to give AI systems more responsibility for operational activities. Some of those applications will be low consequence. Others will involve infrastructure, security controls, financial operations, or systems where an incorrect action can have serious effects. As those use cases develop, I suspect the architectural conversation will need to extend beyond how capable individual AI models have become. We will need to consider what information those models consume, how their conclusions are verified, which authority boundaries govern their actions, and what happens when observations differ from expectations.

Boundary is no universal solution to those problems. It is a proposed architectural approach, supported by bounded prototype evidence, for separating reasoning from authority and keeping consequential actions subject to independently governed controls.

Looking back at the investigation, I keep returning to the two questions that came to define the work.

**What do I trust the model to consume?**

**What do I trust the model's reasoning to influence?**

For consequential enterprise AI, I think those are two of the most useful questions an architect can ask before deciding what to build.

---

## Supporting research

The following links provide selected supporting evidence and the reusable Reference Architecture. The evidence repository is a curated subset, not the complete engineering archive: the full Boundary screening corpus, isolated Ansible probe captures, and complete mutation-test records remain unpublished. Measurements from the earlier GPU experiment are public in its separate repository. Some archived artifacts retain the project's former working name, Project Sentinel.

- [Project Boundary: Selected Supporting Evidence](https://github.com/Robert-N-Myhre/project-boundary-evidence)

- [Reference Architecture for Governed Agentic Infrastructure Operations](https://github.com/Robert-N-Myhre/independent-technology-research/blob/main/docs/reference-architectures/governed-agentic-infrastructure-operations.md)
- [Single-GPU Prompt Security Guardrails Research](https://github.com/Robert-N-Myhre/prompt-guardrail-single-gpu)

*Project Boundary is independent architecture research. Its prototype findings should not be interpreted as evidence of production deployment, complete operational validation, or generalized AI safety.*
