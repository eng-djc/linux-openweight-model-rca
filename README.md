# Linux Perspective on an Open-Weight Model Crash

**Proposed session for Open Source AI Summit 2026**  
**Selected Area:** Inference & Compute

## Session idea

Linux support engineers often troubleshoot applications as black boxes: we may not own the application code, but we still need to explain why they slowed down, stopped responding, or crashed.

In this session, the black box is an open-weight AI model running on Linux in the cloud.

The session starts by establishing a healthy operating baseline for the inference workload. CPU, memory, I/O, process behavior, system logs, and relevant runtime signals are recorded so that expected behavior is understood before introducing a failure.

The workload is then deliberately placed under CPU, memory, and I/O pressure, following it from normal operation through performance degradation and, where technically possible, to an actual crash or process termination.

The objective is not to benchmark the model or compare model quality. The objective is to perform a root-cause analysis from the Linux operating-system perspective.

## Open-Weight Model

The primary workload selected for this study is Microsoft Phi-4, a 14-billion-parameter open-weight language model.

Phi-4 was selected because it provides a useful middle ground for this experiment. It is large enough to create a meaningful Linux workload for observing CPU, memory, and I/O pressure, while remaining practical for controlled and reproducible experiments in a cloud-based Linux environment.

Using a model that is too small could make resource exhaustion trivial and provide a limited degradation window before failure. At the same time, using a very large model would introduce unnecessary infrastructure complexity and could shift the focus of the study toward GPU capacity or large-scale inference rather than Linux root-cause analysis.

Phi-4 allows the study to focus on the transition between:

**normal operation → resource pressure → performance degradation → failure → RCA**

The model will be treated as a black-box workload. The study does not evaluate its intelligence, accuracy, benchmark performance, or model architecture.

### Model format

The experiment will use the official Microsoft Phi-4 ONNX release published for ONNX Runtime inference.

Microsoft publishes optimized Phi-4 ONNX variants for CPU and GPU execution across platforms including Linux. The initial laboratory configuration will use the official CPU INT4 variant:

`cpu_and_mobile/cpu-int4-rtn-block-32-acc-level-4`

The exact model revision, files, and hashes used in the experiments will be recorded in LAB 00 so the environment can be reproduced.

Official ONNX model page:

https://huggingface.co/microsoft/phi-4-onnx

### Runtime

The inference runtime selected for this study is Microsoft ONNX Runtime GenAI, an open-source runtime for generative AI models built on ONNX Runtime.

ONNX Runtime GenAI was selected because it provides an official Microsoft open-source path for running Phi-family models while keeping the laboratory aligned with the objective of studying an AI workload from the Linux operating-system perspective.

The model and runtime together will be treated as the application black box. The RCA will focus on what Linux can observe and explain when the workload experiences CPU, memory, or I/O pressure rather than debugging model internals.

The exact ONNX Runtime GenAI version used in the experiments will be frozen and documented in LAB 00.

Official runtime repository:

https://github.com/microsoft/onnxruntime-genai

### License

The Microsoft Phi-4 ONNX model is released under the MIT License.

Microsoft ONNX Runtime GenAI is also released under the MIT License.

This is important for the project because the experiment is intended to remain based on publicly available and openly licensed technology. The model and runtime are used only as components of controlled Linux experiments and root-cause analysis.

The use of Microsoft Phi-4, ONNX Runtime GenAI, or Azure infrastructure in this independent project does not imply endorsement, sponsorship, or official representation by Microsoft.

## Experimental stack

| Component | Selection |
|---|---|
| Cloud environment | Azure VM |
| Operating system | Linux; exact distribution, version, and kernel will be frozen in LAB 00 |
| Open-weight model | Microsoft Phi-4, 14B |
| Model format | Official Microsoft Phi-4 ONNX release |
| Initial model configuration | `cpu_and_mobile/cpu-int4-rtn-block-32-acc-level-4` |
| Inference runtime | Microsoft ONNX Runtime GenAI |
| Runtime version | Frozen and documented in LAB 00 |
| Initial execution | CPU-only |
| Resource control | Linux cgroup v2 / systemd resource controls |
| Objective | Linux root-cause analysis, not benchmarking |

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

## Failure scenarios

The initial study will focus on controlled pressure in three main areas:

### CPU pressure

CPU availability will be restricted or contention will be introduced to observe how the inference workload behaves as CPU resources become constrained.

The RCA will investigate whether degradation is caused by CPU starvation, scheduling contention, throttling, or another system-level condition.

### Memory pressure and exhaustion

Available memory will be progressively constrained to observe the workload before and during memory pressure.

The study will examine reclaim activity, page faults, process memory, Linux pressure indicators, cgroup events, and eventual OOM behavior where applicable.

### I/O pressure

Controlled storage contention will be introduced to observe how model loading and inference behavior change under I/O pressure.

The RCA will investigate device latency, process I/O, I/O wait, task state, and other Linux evidence to distinguish an I/O bottleneck from CPU or memory problems.

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

The investigation begins with standard Linux tools and escalates to deeper diagnostics only when the available evidence requires it.

Additional tools may include:

- `perf`
- process and thread stacks
- Pressure Stall Information (PSI)
- tracing
- eBPF / `bpftrace`
- `coredumpctl`
- historical telemetry such as `sar` or Performance Co-Pilot
- support collection tools such as `sos report`

## Real-time and postmortem RCA

The study considers two different troubleshooting situations.

### Real-time analysis

When the workload is still running, Linux can expose live evidence such as:

- CPU and memory utilization
- process and thread state
- page faults
- resource pressure
- scheduler behavior
- system calls
- I/O activity
- cgroup resource accounting
- runtime and kernel logs

### Postmortem analysis

When the workload has already crashed or been killed, some live evidence no longer exists.

The investigation must then rely on persisted artifacts such as:

- system and kernel journals
- historical performance telemetry
- cgroup event counters
- process exit information
- core dumps when applicable
- OOM evidence
- support collection reports

A central goal of the project is to determine which evidence must be collected proactively so that a failure can still be explained after the process no longer exists.

## Why this matters

Open-weight models are becoming another class of production workloads running on Linux.

Once these workloads have availability and performance objectives, restarting a failed process is not enough. Engineers need to understand what normal behavior looks like, where the workload's operating boundaries are, how degradation appears before failure, and what evidence can explain the incident afterward.

When inference falls outside its expected operating envelope, engineers need to determine whether the cause is CPU starvation, memory pressure, I/O contention, a runtime failure, or another system-level condition.

This session applies the same evidence-driven troubleshooting discipline used for traditional enterprise Linux workloads to open-weight AI inference.

## Intended audience

Linux engineers, SREs, platform engineers, AI infrastructure engineers, support engineers, and builders operating open-weight models in production or cloud environments.

## Speaker background

I am a Senior Linux Escalation Engineer with 15+ years of experience supporting Linux workloads in enterprise environments, with a focus on advanced troubleshooting, cloud reliability, Linux security, and black-box root-cause analysis.

At Oracle, I contributed to isolating and documenting 36 Oracle Linux and Oracle VM product bugs, with lessons from those investigations contributing to 74 technical documents.

I currently deliver Linux technical training and mentoring to 60+ engineers and have presented internal technical brownbag sessions at Microsoft Azure on topics including CVE Management and Public Key Infrastructure (PKI).

## Related work

- GitHub portfolio: https://github.com/eng-djc
- LinkedIn: https://www.linkedin.com/in/eng-djc/

---

**Status:** Proposal / experimental work in progress.

The final session will include reproducible failure scenarios, collected Linux evidence, and RCA findings from the completed experiments.
