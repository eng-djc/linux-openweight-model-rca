# linux-openweight-model-rca
Linux root-cause analysis of open-weight model performance degradation and crashes under CPU, memory, and I/O pressure.

# Linux Perspective on an Open-Weight Model Crash

**Proposed session for Open Source AI Summit 2026**  
**Selected Area:** Inference & Compute

## Session idea

Linux support engineers often troubleshoot applications as black boxes: we may not own the application code, but we still need to explain why they slowed down, stopped responding, or crashed.

In this session, the black box is an open-weight AI model running on Linux in the cloud.

The session starts by establishing a healthy operating baseline for the inference workload. CPU, memory, I/O, process behavior, system logs, and relevant runtime signals are recorded so that expected behavior is understood before introducing a failure.

The workload is then deliberately placed under CPU, memory, and I/O pressure, following it from normal operation through performance degradation and, where technically possible, to an actual crash or process termination.

## Questions the session will explore

- What does a healthy open-weight inference workload look like from Linux?
- Which Linux signals change first as CPU, memory, or I/O resources become constrained?
- How can we distinguish expected inference behavior from an operating-system resource problem?
- What evidence should be collected while the workload is still running?
- What evidence survives after the process has crashed or been killed?
- Can standard Linux troubleshooting tools provide enough evidence to explain the root cause?
- Did Linux resource controls contain the failure as expected, or did the incident affect the wider system?

## Linux RCA approach

1. Establish the expected baseline.
2. Introduce a controlled failure condition.
3. Observe the customer-visible symptom.
4. Collect Linux evidence.
5. Isolate the responsible resource or failure mechanism.
6. Reconstruct the incident timeline.
7. Provide a defensible root-cause analysis.
8. Identify containment and corrective actions.

## Toolbox

The focus is on tools that Linux engineers already know and use in production environments, including:

- `top`
- `/proc`
- `vmstat`
- `strace`
- `journalctl`
- kernel logs
- process and resource accounting
- cgroups and resource limits
- historical system telemetry
- core dumps and postmortem artifacts when applicable

Deeper tools such as `perf`, stacks, tracing, or eBPF can be introduced only when the standard evidence is not sufficient.

## Why this matters

Open-weight models are becoming another class of production workloads running on Linux. Once these workloads have availability and performance objectives, restarting a failed process is not enough.

When inference falls outside its expected operating envelope, engineers need to determine whether the cause is CPU starvation, memory pressure, I/O contention, a runtime failure, or another system-level condition.

This session applies the same evidence-driven troubleshooting discipline used for traditional enterprise Linux workloads to open-weight AI inference.

## Intended audience

Linux engineers, SREs, platform engineers, AI infrastructure engineers, support engineers, and builders operating open-weight models in production or cloud environments.

## Speaker background

I am a Senior Linux Escalation Engineer with 15+ years of experience supporting Linux workloads in enterprise environments, with a focus on advanced troubleshooting, cloud reliability, Linux security, and black-box root-cause analysis.

At Oracle, I contributed to isolating and documenting 36 Oracle Linux and Oracle VM product bugs, with lessons from those investigations contributing to 74 technical documents. I currently deliver Linux technical training and mentoring to 60+ engineers and have presented internal technical brownbag sessions at Microsoft Azure on topics including CVE Management and Public Key Infrastructure (PKI).

## Related work

- GitHub portfolio: https://github.com/eng-djc
- LinkedIn: https://www.linkedin.com/in/eng-djc/

---

**Status:** Proposal / experimental work in progress.  
The final session will include reproducible failure scenarios, collected Linux evidence, and RCA findings from the completed experiments.
