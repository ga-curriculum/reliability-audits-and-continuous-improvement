# SME Feedback: Reliability Audits and Continuous Improvement
**Module:** Week 8, Class 15 — Reliability Audits and Continuous Improvement  
**Reviewer:** Sumit Rahman  
**Documents Reviewed:** README.md (student-facing), Instructor Guide

---

## Overall Assessment

This is the strongest lesson in Unit 4. The conceptual framing — proactive chaos
engineering as a "fire drill" rather than reactive incident response — is exactly right,
and the continuous improvement loop (audit → RCA → fix → retest) is the cleanest
three-step cycle in the course. The Chaos Mesh toolchain is production-realistic, the
YAML examples are accurate, and the lesson correctly emphasises blameless culture as a
prerequisite for finding vulnerabilities rather than hiding them.

The recurring lab placeholder is the primary issue. The live Chaos Mesh demo faces the
same setup constraints as the Kubernetes labs (requires a running Minikube cluster with
Chaos Mesh installed — a 15-minute setup), but the lesson correctly frames this as a
worth-it investment for the final module of the course.

---

## 1. Structural Issues

**Issue 1.1 — The Audit Process Steps Are Listed But Not Numbered Consistently**

Section 1 lists the audit process as four numbered steps (Define Scope, Establish
Baseline, Inject Failure, Observe and Document). This is the clearest procedural list
in the lesson. However, the audit process diagram (Mermaid graph) does not use the
same step numbering, making it hard for students to map the diagram to the text.
The diagram nodes should reference the step numbers.

---

## 2. Alignment with Syllabus and Competency Map

| Learning Objective | In README? | In Instructor Guide? | Notes |
|---|---|---|---|
| Explain the purpose and process of reliability audits | ✅ Section 1 | ✅ Yes | Four-step audit process is well-defined |
| Conduct performance reviews using key metrics and historical data | ✅ Section 2 | ✅ Yes | SLI reference is accurate |
| Design continuous improvement plans aligned with reliability goals | ✅ Section 3 | ✅ Yes | RCA → action items → validate loop is correct |

All three LOs are addressed. The lesson is the first in the course to explicitly
connect all three lab types (YAML infrastructure, postmortem writing, and data
analysis) into a single workflow.

---

## 3. Technical Content

**Issue 3.1 — Chaos Mesh Installation Not Addressed**

Section 1 mentions Chaos Mesh running on Minikube but does not provide the installation
command. Chaos Mesh installation requires Helm and a specific `--set` configuration
depending on whether Minikube uses Docker or VM drivers:

```bash
helm repo add chaos-mesh https://charts.chaos-mesh.org
helm install chaos-mesh chaos-mesh/chaos-mesh \
  --namespace=chaos-mesh --create-namespace \
  --set chaosDaemon.runtime=containerd \
  --set chaosDaemon.socketPath=/run/containerd/containerd.sock
```

Without these flags, the Chaos Mesh pods start but `PodChaos` experiments silently
fail. The installation command belongs in the lesson's prerequisites or the lab README.

**Issue 3.2 — Selector Mismatch is the #1 Chaos Mesh Failure Mode (Undersurfaced)**

The instructor guide's troubleshooting table correctly identifies "Failure Injection has
no effect" as caused by a selector mismatch — but buries it in the table. This is the
most common reason chaos experiments produce no observable effect in a student lab
environment. Moving this to the lesson body ("your experiment's `labelSelectors` must
exactly match the labels on your target pods") would prevent the most common
frustration point.

**Issue 3.3 — `NetworkChaos` Example Could Show `correlation` Field**

The lesson's practical example in Section 1 shows a `NetworkChaos` rule with a `2s`
delay but no `correlation` field. In practice, injecting a delay without correlation
affects all packets independently, which does not simulate the bursty network
degradation that real infrastructure failures produce. Adding `correlation: "25"`
(25% of packets experience correlated delay) would make the example more realistic
and match EXP-002 in the lab data.

---

## 4. Instructor Guide

**Issue 4.1 — Demo Setup Time Not Accounted For**

The pacing guide allocates 40 minutes to a "Live demonstration: Deploy a simple pod
to Minikube. Use Chaos Mesh to inject a network delay or pod kill." The setup required:
- Minikube start (if not already running): ~2 minutes
- Chaos Mesh installation (Helm): ~5 minutes
- Pod startup and metrics-server warm-up: ~3 minutes

Total setup: ~10 minutes before the first chaos command can be run. The instructor
guide should include a "before session" checklist identical to Lab 11's approach:
*"Start Minikube and install Chaos Mesh at least 15 minutes before the session."*

**Issue 4.2 — The "Fire Drill" Analogy Is the Strongest Teaching Point**

The instructor guide mentions the fire drill analogy in delivery tips but does not
position it as the opening hook for the session. Starting the session with: *"How
many of you have run a fire drill? That is what we are doing today. We are setting
the building on fire — in a controlled way — to verify the escape routes work."*
is more memorable than the pacing guide's current framing. This is worth making
explicit as the opening 60 seconds of the session.

---

## 5. Theory-to-Lab Alignment Recommendations

**Recommendation 5.1 — Add Chaos Mesh Installation Command to Section 1 or Lab README**  
The Helm command with `chaosDaemon.runtime` flags prevents the most common installation
failure. Without it, students see Chaos Mesh pods running but experiments silently
do nothing.

**Recommendation 5.2 — Connect Chaos Experiments to Prior Lab Incidents**  
The lesson introduces chaos engineering as a new concept but does not connect it to the
incidents students have already analysed. EXP-001 (pod kill) is the same failure as
Lab 11's Failover A. EXP-002 (network delay) is the same as the timeout pattern that
caused INC-002's long detection gap. Making this connection explicit ("we are now
testing whether the fixes from Labs 11 and 13 actually work") gives the lesson a
payoff rather than starting fresh.

**Recommendation 5.3 — Add Selector Mismatch Note to Lesson Body**  
One sentence in Section 1: *"The `labelSelectors` in your chaos experiment must exactly
match the pod labels applied by your Deployment — verify with `kubectl get pods
--show-labels` before running an experiment."*

---

## Summary of Action Items

| Priority | Issue | Type | Owner |
|---|---|---|---|
| High | Add Chaos Mesh installation command (Helm + runtime flags) | Fix | Content Author |
| High | Add selector mismatch warning to lesson body | Fix | Content Author |
| Medium | Connect chaos experiments to prior lab incidents (Labs 11, 13) | Recommendation | Content Author |
| Medium | Add Chaos Mesh setup to instructor pre-session checklist | Fix | Content Author |
| Medium | Position "fire drill" analogy as opening session hook | Recommendation | Instructor |
| Low | Align audit process diagram step numbering with text list | Fix | Content Author |
| Low | Add `correlation` field to NetworkChaos example in Section 1 | Recommendation | Content Author |
