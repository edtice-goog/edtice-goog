# Ed Tice

**I apply frontier models to the secure development lifecycle so that security is neither compromised nor the phase that limits delivery.**

For most of my career the scarce thing in software security was detection — finding defects efficiently enough that the finding happened before release instead of after. I spent nineteen years on that problem: static analysis internals, compiler dialects nobody else wanted to touch, fuzzing, and the unglamorous work of getting a real build through an analyzer honestly.

Detection is no longer the scarce thing. That changes where the bottleneck sits, and it changes what is worth building.

Six projects below. They look unrelated — analyzer internals, a supply-chain monitor, a CISO strategy program, a CRM — but they are the same move applied to different problems. Each one starts by refusing the question as asked, and ends by producing the artifact that settles it rather than an argument that it is settled.

---

## [CoveritySkills](https://github.com/edtice-goog/CoveritySkills)

Eleven skills that teach an AI coding assistant to use Coverity Static Analysis properly. 7,700 lines of standard-library Python, developed against real 2026.6.0 and 2026.3.0 installations.

**The problem.** A deep analyzer is built to do something sensible on any codebase out of the box *and* to be tuned hard for a particular one — a real toolchain, a specific defect class, a team's own conventions. The distance between those two is where the value is. Most teams never cross it, because the reference is enormous and the actual skill is knowing which twenty lines of it bear on the situation in front of you.

**What it does.** Closes that distance deliberately rather than by trial and error: model every compiler invocation the build actually makes, confirm how much of the project reached the analyzer before interpreting a result, and tune toward a specific question so you turn on what earns its keep instead of everything.

**The interesting part.** When a developer says they don't believe a finding, that is fair diligence and it deserves better than reassurance. `coverity-fuzz-triage` generates a C stub for each callee from the analyzer's *own derived model* — a `.dot` automaton whose edge labels are the behaviours the analyzer assumes — and enumerates root-to-final paths as behaviour sets. A libFuzzer and AddressSanitizer harness selects one per call from the fuzz input.

The consequence is the whole point: **a crash is one the analyzer would have accepted.** The question stops being a matter of reading and becomes a matter of running.

`coverity-function-slice` makes that affordable, pulling one function out of the intermediate directory as a file that compiles alone — the body plus every typedef, struct, enum, global and callee prototype it transitively needs, reconstructed from the emit's AST in dependency order. Re-emits and re-analyzes in seconds instead of a rebuild.

**The proof.** A factual claim in a skill was established by running the command, not remembered. Each skill's `CALIBRATION.md` records the runs and the boundary of what they cover. When a run contradicts the text, the text changes and the correction is kept — rule 9 was rewritten after a run showed the mechanism behaving differently from the write-up, and a recommended reconciliation was withdrawn when the link units turned out never to be produced.

---

## [pathout-shapes](https://github.com/edtice-goog/pathout-shapes)

Twelve path-insensitive CodeXM checkers, one per defect shape.

**The problem — and why it is not the one people assume.** An analysis that finishes and finds 99% of the defects is worth more than one that finds 100% and never terminates. That bargain is why heuristic analyzers prevailed over sound formal-methods tools on large codebases, and for twenty years it was the right call. The path limit is a deliberate scalability decision — the reason the technology scales at all, not an oversight in it.

What changed is the economics on the other side of that trade. The residual defects the bound left behind used to require expert attention to find and chain together, and expert attention was scarce. It no longer is. So the corners it was rational to leave unlit are now worth lighting.

**What it does.** Reasons about structure only — deliberately noisy — running across the whole intermediate directory and filtering back to exactly the functions where the path-sensitive checker reached its budget. Each checker stands in for a specific set of Coverity checkers, so the filter knows which findings bear on which shape. Covers null-test-then-dereference, division by an unchecked zero, double free, unbounded copy into a fixed buffer, allocation never released, unchecked null return, unchecked array index, integer overflow before allocation, free of non-heap memory, non-literal format strings, sizeof-of-pointer as a size, and the buffer shape behind CVE-2025-0282.

**Why it matters commercially.** The goal is keeping these an early-stage find. Pushing a defect class to a late-stage tool does not only raise cost per finding — it converts a cheap lifecycle stage into an expensive one that no budget accommodates. Higher detection rate, without surrendering the efficiency that made the technology adoptable in the first place.

The repository includes a section on what the checkers cannot do, on purpose.

---

## [ssdlc-mythos](https://github.com/edtice-goog/ssdlc-mythos)

A thesis and eight-deck program on secure development practice under frontier-model threat capability. Presented at OWASP, August 2026. This is the argument the project above is an instance of.

**The problem.** The question a CISO is usually handed is *"where do we put AI in our pipeline."* The more useful one is whether the process downstream of detection is rigorous enough to absorb whatever tools arrive — because tools turn over in eighteen months and process does not.

**The argument.** Attacker attention used to be a scarce resource, and that scarcity quietly subsidized the lifecycle of every organization that was not safety-critical. Deferring low-severity findings was never negligence; it was a rational response to a real constraint, because chaining unremarkable findings into a critical path took expert attention that rarely arrived.

Frontier capability supplies exactly that reasoning layer. Not detection — fuzzing and sanitizers had already industrialized that — but the judgment on top of it. Which is why the change reads as discontinuous even though the underlying curve is continuous, and why the deferral heuristic will not return if model capability plateaus: the bottleneck it depended on was never about model quality.

**What follows.** A two-loop model — a per-finding defect loop owned by engineering, and an aggregate portfolio loop for control calibration — plus classification, stage placement, tool interface properties, selection, AI roles, economics and agentic oversight.

**The proof.** The argument is built to survive being wrong about its own most dramatic claim. If frontier capability plateaued tomorrow the underlying curve keeps rising, because the forces driving it are independent of any vendor's roadmap — and the thesis says so explicitly rather than resting on a single model generation.

---

## [ssdlc-onboarding](https://github.com/edtice-goog/ssdlc-onboarding)

A system of record and measurement for what actually ships.

**The problem.** Asking whether every application an organization is responsible for is covered by its security program is a reasonable question and a genuinely hard one. The evidence is real but distributed — it lives in an asset inventory, a source control system, a build server and a scanner dashboard, each owned by different people and none of them reconciled against the others.

**What it does.** Records three things for every application: what we have, what lifecycle controls are supposed to run against it, and what evidence exists that they actually ran — including the part everyone skips, whether anyone is acting on what the tools find. 24 tables, 7 views, 32 API paths, and a working slice running end to end over simulated inventory, source-control, build-server and spreadsheet sources. Design document with 60+ recorded decisions.

**The discipline.** It renders no verdicts and makes no decisions — those belong to expert practitioners. It records what exists, what was decided, who decided it, and what has been observed. The first slice covers inventory, mapping and coverage only, on simulated data, so the problem and the proposed solution can be argued over by people who will never read the design document, while changing course is still cheap.

---

## [RepoMonitoring](https://github.com/edtice-goog/RepoMonitoring)

A patch-gap monitor for embedded and native builds. Four services, roughly 8,000 lines of Python, designed and shipped in one week.

**The problem.** A bill of materials answers what is in a product. That is the right question for inventory, licensing and advisory matching. For an embedded or native build a second question follows it: of the code that actually compiled into this binary, has anything upstream changed in a way that matters?

That one is harder, because the useful signal often arrives without an advisory. Upstream maintainers repair memory bugs as ordinary maintenance and ship them without filing a CVE — normal practice, and not something an advisory feed is designed to catch.

**The definition that makes it work.** A repository is *monitored* only if it owns at least one primary translation unit that actually compiled. Owning only `#include`d headers makes it *reference-only* — shown for transparency, never watched. That single distinction separates a precision tool from an inventory with notifications turned on.

Three independent signals are unioned so the watch set is never under-approximated: the SCA bill of materials, an inference over compiled file paths, and `.git` discovery as ground truth. Forked and vendored copies resolve to the canonical upstream through the fork-parent link, because that is where patches land.

**The proof.** Every triage verdict is cross-checked against OSV.dev by exact commit SHA — with no model anywhere in that path. So the grader is independent of the thing it grades: a `response_required` on a known CVE fix is corroborated, a `not_meaningful` is a flagged miss, and a `suppressed` one shows the scope filter correctly excluding a patch to files this build never compiles.

The architecture document names its own weakest seam before a reviewer can.

---

## [CustomerRelationshipManagement](https://github.com/edtice-goog/CustomerRelationshipManagement)

An observation-based CRM whose input layer is agents reading existing communications. Included here because it demonstrates that the method is not specific to security.

**The problem.** CRM fields go stale because the system asks people to type in what they have already said in email. The information exists — it is in the thread. That is a design problem, not a discipline problem.

**The boundary the design rests on.** *Detection is code, judgment is the model.* Case tokens, dedup hashing and name normalization are regex and SQL. Who is mentioned, how a case is going, whether a reply fulfills an outstanding promise — those are model calls. Getting that line right is the whole design, and it is the part that does not transfer from a conventional CRM.

**What follows from it.** There are no wide profile tables. Every fact is an observation carrying a confidence, a verbatim evidence quote and a link to its source, and a profile is a projection over active observations. New attribute types need no schema change, and because raw content is kept, extraction can be re-run as models improve.

---

## Method

Four habits, visible in all six.

**Solve the problem, not the one the tool is shaped to answer.** Every project here began by rejecting the stated question. "Configure the analyzer" was really "make me believe the findings." "Scan my dependencies" was really "tell me about code I actually compile." "Where does AI go" was really "can our process absorb any tool."

**Build the check before the thing being checked.** The OSV cross-check exists so the AI triage can be wrong in public. The obfuscation comparison exists so a redacted twin cannot quietly differ. Capture fidelity exists because every downstream conclusion assumes the code reached the analyzer.

**Credit what exists before extending it.** The path limit is not a flaw, it is the bargain that made the technology scale. A bill of materials is not noise, it answers a different question. Deferring low findings was not negligence, it was rational under a constraint that has since changed. Work that begins by finding fault usually misunderstands why the thing was built.

**Advance underspecified work before asking for more.** Partial execution is discovery: it establishes which missing input actually blocks progress rather than which was presumed to, and it replaces an open request with a specific one supported by documented state.

---

*Coverity and Black Duck are trademarks of Black Duck Software. These are independent projects, not affiliated with or endorsed by any vendor.*
