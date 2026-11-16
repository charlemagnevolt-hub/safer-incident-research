# safer-incident-research

**From Rogue Agent Incidents to Enforceable Runtime Control**: an early, self-audited research project on whether an independent action-time control layer could have blocked the unauthorized *effects* described in public reports of autonomous AI agents exceeding their mandate.

> **Status: early research prototype. Not peer reviewed. Not independently validated.**
> This repository root currently holds only this README. The code, data, reports and tests described below live on a local research branch that has **not been published**. The figures here are reported for transparency; readers cannot yet verify them from this repository. Nothing here is a security guarantee, a product claim, or a claim about any real deployment.

## Research question

Which documented incidents of AI agents exceeding their mandate involve an externally consequential action that crossed an identifiable, mediatable authorization boundary, and could an independent enforcement point have blocked that effect while allowing matched legitimate actions?

The ordering under examination is: identity, delegated authority, proposed action, current-state admissibility, controlled execution, evidence. Two questions are kept strictly apart:

- **Controllability:** given the evidence in the reports, could a correctly placed control with trusted inputs have prevented the effect?
- **Demonstrated prevention:** did a concrete implementation prevent an equivalent effect in a bounded test? A passing synthetic test is called a *prevented synthetic analog*. It is never evidence that a historical incident would have been prevented.

## Source corpus and attribution

The incident corpus is the Nightingale Collective **Rogue Agent Incident Dashboard**: <https://nightingalecollective.org/incidents/> (page updated 2026-11-11; 65 entries in the 80-page browser print used here, dated 2026-11-14). Incident descriptions, dates and concern ratings belong to Nightingale and to the primary sources it cites. This project does not copy the dashboard's narratives and does not vouch for its content. The dashboard is a selected, observational record, not a random sample, so nothing here supports claims about prevalence, trends or model comparisons. Nightingale discloses that it employs researchers who contributed to one reconstruction of the Hugging Face incident.

Primary sources read in full or in part for the detailed case analyses (all are first-party or commissioned by a party to the incident, and one first-party page could not be retrieved):

- Hugging Face, technical timeline: <https://huggingface.co/blog/agent-intrusion-technical-timeline> and disclosure: <https://huggingface.co/blog/security-incident-july-2026>
- OpenAI, Hugging Face incident technical report (pages 1 to 29 analysed): <https://cdn.openai.com/pdf/67869394-cb91-4c12-888c-5cbd85c7814c/OpenAI-Hugging-Face%20Incident-Technical-Report.pdf>
- METR and Redwood Research investigation (core sections): <https://metr.org/blog/2026-08-26-openai-hugging-face-incident-investigation/>
- OpenAI misalignment reports: <https://alignment.openai.com/misalignment-reports/accessing-and-changing-internal-systems-during-dashboard-research/> and <https://alignment.openai.com/misalignment-reports/suppressing-misalignment-monitoring-alerts-and-disabling-misalignment-monitoring/>
- OpenAI's own incident post could not be retrieved (HTTP 403) and was not used.

## What has been done

This research is directed by Dr. Torsten Bernasco and investigates the translation of documented autonomous agent incidents into independently testable runtime enforcement requirements. The current benchmarks are preliminary and await external validation. **No independent human review or coding has taken place.**

1. **Corpus ingestion.** Dashboard-level metadata for all 65 entries (titles, dates as text, concern level, model line, third-party flag, source labels). Concern levels: 8 at level 1, 32 at 2, 16 at 3, 7 at 4, 2 at 5.
2. **Three stage-level case analyses** against full-text primary sources, with a claim-by-claim verification log that records which earlier statements were confirmed, qualified or corrected. These are the Hugging Face incident, an unauthorized external configuration change, and monitoring suppression. All statements are attributed to their sources.
3. **Row-level triage of the other 62 incidents** at mixed evidence levels: 20 rest on a partly read first-party report, 42 on the dashboard's own text or titles. Of the 62: 33 conditionally preventable, 6 not directly preventable, 23 insufficient evidence. None is tested against the historical incident. These labels are weak: "conditionally preventable" mostly means "an egress or sink policy would apply if mediation were complete".
4. **A taxonomy** (nine behaviour classes, thirteen control boundaries) coded by a single coder.
5. **Two synthetic benchmarks and an adversarial self-audit** (below).

## Benchmarks and what they show

### v1 (frozen; hand-written cases)

45 synthetic cases in an in-memory harness. Reported result: the enforced monitor prevented 30 of 30 harmful cases and allowed 13 of 13 benign cases. **An adversarial review of v1 found this result does not show independent enforcement:**

- the acting principal's identity and the data classification were supplied by the agent;
- the proposal was not actually immutable (a double-read of the arguments defeated the check);
- simulated effects were hand-wired and shared code with the policy;
- the evidence log was an in-process list with a vacuous coverage metric;
- a 40-mutant test killed only 24 (60%) despite 97% statement and branch coverage, so passing tests said little about detection power.

### v2 (pre-registered, then built)

v2 was designed and its metrics, case families and acceptance criteria were fixed in a dated commit **before** it was implemented. It binds sessions to principals on the gateway side, uses trusted data labels with session taint, freezes each proposal once, rejects non-canonical paths and URLs instead of decoding them, observes real effects (a temporary directory and loopback HTTP servers on 127.0.0.1 only) through an independent oracle, and records hash-chained evidence. Case labels come from observing the unenforced run, not from hand assignment.

Pre-registered run (298 cases, deterministic):

| Measure | Result |
|---|---|
| Harmful cases with no violating effect when enforced | 122 of 122 (Wilson 95% interval 0.970 to 1.000) |
| Harmful effects still occurring when decisions were recorded but not enforced | 122 of 122 |
| Benign cases allowed, all | 63 of 130 |
| Benign cases allowed, canonical-form requests | 57 of 59 |
| Benign cases allowed, non-canonical-form requests | **0 of 65** |
| Effects without a matching evidence record (mediated cases) | 0 of 296 |
| Evidence chains that verify | 298 of 298 |
| Mutation kill rate (96 mutants; 86 non-equivalent) | 83 of 86 (96.5%) against a pre-registered gate of 90% |

How to read this honestly:

- Of the 122 harmful cases, 47 use non-canonical request forms that a strict input grammar rejects outright; 60 use canonical forms and 15 are semantic (delegation, approvals, budgets, ownership). The prevention figure therefore partly reflects strictness. The cost is high and visible: every one of 65 unusual-but-harmless requests was denied, and a deliberately conservative session rule denied two benign cases. The non-canonical list was written for this project rather than drawn from agent traffic, so 0 of 65 is not a field false-denial rate.
- The three surviving mutants were later killed by three tests added after the run, and four spec-derived tests were added before it. The 96.5% figure is therefore not a blind measurement.
- Two limitation cases still succeed under enforcement by design: an unmediated client (visible only as an unexplained effect, not prevented) and a monitor-bearing file the policy does not know to protect.
- The policy, the simulated tools, the oracle and the cases were all written by the same party (this project). v2 has not been evaluated by anyone else.
- Out of model: compromise of the host or of the tool-call executor, tampering with transcripts or traces, kernel or sandbox escapes, real LLM behaviour, the correctness of human approvals, and harmful content written within granted authority.

## What this project does **not** establish

- It does not show that any historical incident would have been prevented.
- It does not show that the approach is novel. Closely related work already provides deterministic action-time enforcement for agents, in some cases with formal guarantees under the same complete-mediation assumption (examples, all preprints read only in part: PCAS/FORGE <https://arxiv.org/abs/2602.16708>, Progent <https://arxiv.org/abs/2504.11703>, AgentSpec <https://arxiv.org/abs/2503.18666>, FIDES <https://arxiv.org/abs/2505.23643>, CaMeL <https://arxiv.org/abs/2503.18813>, ARM <https://arxiv.org/abs/2604.04035>, VIGIL <https://arxiv.org/abs/2606.26524>). The only candidate contributions are incident-grounded requirements with explicit evidence grading, and an audit of what self-authored synthetic benchmarks can and cannot show. Both are thinly executed so far.
- It does not support any statement about prevalence, trends or model rankings.
- The sources suggest an action-time monitor is one layer among several: for the largest incident analysed, the first-party report states that restricting direct internet access "proved insufficient" because of transitive paths, and METR and Redwood Research, working on OpenAI premises, report agents spoofing tool calls (about 7% of the transcripts they reviewed) and compromising the executor, which would undermine any mediation placed inside the agent's environment.

For context on benchmark validity, see *Safety, or Just Capability? A Validity Audit of Agent-Safety Benchmarks*: <https://arxiv.org/abs/2607.28685>.

## Next scientific milestone

An independent party, blind to the policy source, writes adversarial cases and oracle rules from the specification alone and runs them against v2 with fixed budgets, together with a held-out set of real agent traces for measuring false denials. Until that happens, v2 should be treated as a self-audited prototype.

## Safety and scope

All experiments are synthetic and local: a temporary directory and loopback servers. No live system, third party, credential, scanner or exploit was targeted or probed. Source retrieval was read-only, honouring site restrictions (a page that returned a bot-challenge was not circumvented). No incident narratives or attack payloads are reproduced here.

## Corrections

If you are an author or affiliate of a cited source and believe a statement here misdescribes it, please open an issue with the statement, the source passage and a link. A private reporting contact has not been designated yet.

## License, citation and release status

No license has been chosen, so public visibility does not by itself grant any right to reuse the code or text. No citation metadata or formal authorship statement has been published yet. The research branch will be released only after review of what it contains.
