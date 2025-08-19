https://github.com/M-c-tech-dev/mchammer/releases

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/M-c-tech-dev/mchammer/releases)

# mchammer — Machine-Check Exploitation Toolkit for Kernel Research 🔨🧩

![machine-check](https://images.unsplash.com/photo-1518770660439-4636190af475?auto=format&fit=crop&w=1400&q=80)

Table of contents
- Overview
- Goals
- Quick start
- Download and run release
- Build from source
- Core concepts
- Features and modules
- Example workflows
- Output and reports
- Architecture and internals
- Development notes
- Contributing
- License and contact

Overview
This repository holds mchammer, a toolkit for controlled research into machine-check events (MCE), hardware error handling, and fault-injection related to CPU and memory subsystems. The project targets kernel developers, platform engineers, and reliability researchers. It provides components that generate, analyze, and log MCE-style conditions and that help map hardware signals to kernel behavior.

The project operates at the interface of hardware error reporting, RAS (Reliability, Availability, Serviceability), and OS handling. It focuses on replicable test cases, deterministic inputs, and structured output that support root-cause analysis.

Goals
- Provide repeatable test vectors for MCE-style conditions.
- Offer modules that simulate or trigger RAS paths in a lab.
- Map hardware event traces to kernel state and stack dumps.
- Produce structured reports for post-mortem analysis.
- Allow safe bench testing inside isolated environments.

Quick start
The releases page contains packaged binaries and release artifacts. You must download the release file for your platform and run it inside an isolated test environment.

Download releases and run the file from the Releases page:
https://github.com/M-c-tech-dev/mchammer/releases

If you need a visual cue, click the badge above to go to the releases. Pick the artifact that matches your OS and architecture, unpack it in a VM or lab host, and follow the included README in the artifact.

Download and run release
- Visit the Releases page and pick the artifact for your platform.
- Download the binary or archive to an isolated VM or dedicated test host.
- Extract the package and read the packaged README or RELEASE_NOTES.
- Execute the provided launcher or test harness.

Example (abstract)
- Download the archive from the Releases page.
- Extract the archive.
- Run the test harness from a dedicated test host or VM.

Do not run these artifacts on production systems. Use a controlled lab with snapshots or disposable hardware.

Build from source
Clone the repository and build locally. The project uses a small toolchain that targets kernel and userland components.

git clone https://github.com/M-c-tech-dev/mchammer.git
cd mchammer

Prerequisites
- gcc or clang
- make and cmake
- kernel headers for your target
- Python 3 for the harness and reporting tools

Build steps (summary)
- Build the core library:
  cd core && mkdir build && cd build && cmake .. && make
- Build the userland harness:
  cd ../harness && make
- Run unit tests:
  cd ../tests && ./run_tests.sh

Core concepts
- Machine Check Exception (MCE): A CPU-level event raised when hardware detects a fatal or recoverable error. MCEs often include error codes, physical address info, and bank registers.
- Error bank: CPU internal registers that capture error state per domain.
- Poisoned memory: Data corrupted by a hardware fault and flagged by ECC or system firmware.
- RAS path: The end-to-end path of detection, logging, recovery, and reporting for hardware faults.
- Synthetic injection: Controlled creation of fault conditions in hardware or simulation to exercise error pathways.

Features and modules
mchammer divides functionality into modules that you can combine in tests.

- harness: A userland harness that drives tests and collects logs.
- injector: Modules that request firmware or hardware components to emulate error conditions in lab setups.
- tracer: Kernel and userland hooks that capture call traces and CPU state at the time of the event.
- parser: Tools that parse MCE logs and map codes to known root causes.
- reporter: Generates JSON and HTML reports for each test run.

Selected module list
- mce-sim: Simulates MCE bank records and posts them to the OS logging facility.
- pci-parity: Exercises PCI parity paths in a sandboxed device model.
- mem-poison: Runs memory patterns to stress ECC and logs corrected/uncorrected events.
- pagewalk: Triggers corner cases in page table handling under injected faults.
- uefi-fit: Interfaces with a UEFI test firmware to toggle hardware reporting features.

Example workflows
1) Reproducible MCE simulation
- Run the mce-sim module in the harness on a lab VM.
- The tracer collects kernel state at the simulated event.
- The parser creates an event mapping and the reporter produces a JSON file.

2) ECC stress and log correlation
- Run mem-poison against a target DIMM in an isolated test rig.
- Use tracer to correlate physical addresses with kernel page allocations.
- Use the parser to match ECC syndrome to DIMM topology.

3) Firmware-RAS verification
- Coordinate a UEFI fixture with uefi-fit to toggle error reporting.
- Confirm that firmware buffers propagate to kernel MCE events.
- Capture timestamps and stack traces for end-to-end validation.

Output and reports
mchammer produces structured output to support automation.

- Raw logs: syslog or kernel ring buffer captures for each event.
- JSON events: Each test run produces a JSON file with fields such as timestamp, cpu, bank, syndrome, paddr, module, and trace.
- HTML report: The reporter generates a human-readable HTML document with charts and timelines.
- CSV summary: Aggregated statistics for stress runs.

Fields in JSON output (example)
- run_id: UUID
- start_time: ISO8601
- events: array of objects
  - cpu: integer
  - bank: integer
  - syndrome: hex string
  - paddr: hex string
  - description: mapped cause
  - trace: array of frames

Architecture and internals
mchammer uses a layered design to minimize risk and to let researchers isolate components.

- Userland harness
  - Orchestrates tests and collects signals.
  - Runs modules as subprocesses and captures stdout/stderr.
- Tracer
  - Kernel probe or eBPF-based collector that timestamps events and stores state.
  - Minimal footprint to avoid altering system behavior.
- Injector
  - Interfaces to firmware, lab controllers, or SoC debug interfaces.
  - Abstracts hardware-specific commands behind a driver API.
- Parser and reporter
  - Map raw codes to human-readable root causes.
  - Produce structured artifacts for CI and dashboards.

Design notes
- Keep the tracer code small and auditable.
- Use JSON for machine-parsable reports.
- Support offline analysis by keeping timestamps and offsets.

Integration points
- CI: Test harness supports headless runs and artifact upload.
- Lab controllers: Use a simple REST interface to control power and firmware in lab racks.
- Telemetry: Export a summary CSV for integration with dashboard tools.

Development notes
Coding style
- C for core modules that touch kernel interfaces.
- Python 3 for harness and reports.
- Use linters and static analysis for safety.

Testing
- Unit tests cover parsers and reporters.
- Integration tests run in QEMU or isolated VMs for deterministic events.
- Use snapshot restore to ensure clean test starts.

Logging
- The harness logs to both console and per-run directories.
- Each run directory includes: raw.log, events.json, report.html.

Contributing
- Fork the repo and open a PR.
- Follow the coding style for the target language.
- Write tests for new features and regression cases.
- Tag modules with clear metadata: name, description, impact, required-hardware.

Issue reporting
- File issues with reproducible steps and attachments for events.json.
- Provide platform details: CPU vendor, kernel version, firmware version.

Security model
- The project targets research in controlled settings.
- Modules that require hardware control are gated and clearly labeled.
- Use the releases page to obtain signed packages when available.

Contact and references
- For questions and patches, open an issue or a PR on the main repository.
- See the Releases page for packaged artifacts and versioned builds:
  https://github.com/M-c-tech-dev/mchammer/releases

References and background
- Intel and AMD MCE documentation
- Kernel MCE subsystem docs (Documentation/x86/mce.txt or equivalent)
- RAS whitepapers and ECC literature
- eBPF tracing techniques for low-impact capture

Images and diagrams
- Use the images inside docs/diagrams for testbed layouts.
- External image: a CPU lab photo for visual context
  ![lab-photo](https://images.unsplash.com/photo-1518779578993-ec3579fee39f?auto=format&fit=crop&w=1400&q=80)

Release artifacts and execution
The Releases page hosts artifacts that target different platforms. Download the release file that matches your platform. The release contains a launcher and a README that lists the exact steps to execute the test harness and the modules it contains. Use the supplied launcher inside a controlled environment.

Releases link (again) and guidance:
- Visit and download the release artifact here: https://github.com/M-c-tech-dev/mchammer/releases
- Extract the package and follow the packaged README for the artifact.
- Run the harness inside an isolated test VM or lab host.

Common troubleshooting
- If logs show missing kernel symbols, install the kernel headers that match your running kernel.
- If the tracer fails to attach, check kernel config for debug and tracing options.
- For hardware interface failures, verify out-of-band access and firmware permissions.

Tags and metadata
- Repository: mchammer
- Topic: machine check exploitation (research)
- Maintainer tags: RAS, MCE, kernel, ECC, tracer

License
- See LICENSE in the repository root for the project license and contributor terms.

End of file