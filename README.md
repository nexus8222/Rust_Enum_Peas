# rust_peas

`rust_peas` is a next-generation, high-performance, concurrent Linux local security auditing, compliance verification, and privilege escalation risk-assessment engine written entirely in native Rust.

Designed as a modern, blindingly fast successor to legacy shell-based enumeration scripts, `rust_peas` leverages massive parallel thread pools and direct kernel/VFS interaction to deliver comprehensive host auditing without the operational overhead or detection signatures of spawning thousands of subshells.

---

## Key Architectural Advantages

| Capability            | Legacy Shell Scripts (e.g., LinPEAS)                | `rust_peas` (Rust Engine)                                      |
| --------------------- | --------------------------------------------------- | -------------------------------------------------------------- |
| **Execution Speed**   | Slow (Spawns thousands of heavy bash subshells)     | **Instantaneous** (Parallel processing via Rayon)              |
| **System Visibility** | High (Generates massive host process-logging noise) | **Minimal** (Direct VFS & system call interaction only)        |
| **Memory Inspection** | None (Limited by shell scripting capabilities)      | **Advanced** (Native runtime memory & descriptor scanning)     |
| **Portability**       | Dependent on host binaries (`grep`, `find`, `awk`)  | **Zero Dependencies** (Statically compiled via MUSL toolchain) |

---

## Current Subsystem Scan Matrix

The engine executes a multi-threaded auditing pipeline broken into distinct structural modules:

### Core Security & Hardening Baseline

* **[1] SUID/SGID GTFOBins Cross-Reference:** Asynchronously identifies elevated binaries and flags signatures matching known exploit primitives.
* **[21] Extended Kernel Sysctl Hardening:** Evaluates kernel execution parameters (e.g., `unprivileged_userfaultfd`, `unprivileged_bpf_disabled`, `perf_event_paranoid`) against secure configuration standards.
* **[33] Configuration Secrets Harvester:** Recursively combs through system configuration targets (`/etc`, `/opt`, `/var/www`) utilizing high-performance parallel regex matching to capture plaintext credentials.
* **[34] Chronological Task Analyzer:** Audits system-wide automation tables (`/etc/crontab` and `cron.*` drops) for unprivileged configuration anomalies.

### Infrastructure & Identity Control

* **[35] Container Boundary & Virtualization Isolation:** Inspects virtualization properties (`/proc/1/cgroup`) and micro-architectural capabilities (`CapEff` masks) to detect nested environments and potential escape vectors.
* **[36] Cryptographic Auth & SSH Hardening:** Parses home layouts via `/etc/passwd` to catch over-permissive files or exposed `.ssh/id_rsa` keys across every user profile.
* **[37] Kernel Patch Level Matcher:** Extracts release signatures natively via `libc::uname` to correlate operating system variants against critical missing patch thresholds (e.g., Dirty Pipe, OverlayFS vulnerabilities).
* **[38] Dynamic SUID Library Integrity:** Inspects dynamic linking paths of elevated components to eliminate shared object hijacking pathways from world-writable environments.

---

## Building the Binary

To ensure full compatibility across legacy and modern Linux deployments, compile the binary as a completely static standalone executable via the MUSL target framework:

```bash
# Add the MUSL compilation target architecture
rustup target add x86_64-unknown-linux-musl

# Build the production-hardened binary
cargo build --release --target x86_64-unknown-linux-musl
```

The resulting optimized asset can be found at:

```text
target/x86_64-unknown-linux-musl/release/rust_peas
```

---

## Master Engineering Roadmap

To establish `rust_peas` as a definitive open-source platform for Linux host auditing, the roadmap focuses on expanding deeper into the host operating system layers:

### Phase 1: Advanced In-Memory & Process Sniffing (Immediate Next Steps)

* **Live Process Memory Maps (`/proc/[pid]/maps`):** Implement native parallel memory page scrapers to inspect environment strings of running processes for orphaned API tokens, credentials, or session variables.
* **Interactive IPC Audit:** Map local UNIX sockets, unnamed pipes, and shared memory allocations (`shm`) to find hidden communications between high-privileged and low-privileged users.

### Phase 2: Systemd & Microservice Hardening Depth

* **Deep Systemd Dependency Mapping:** Inspect unit definition structures, drop-in overrides, and absolute service binary path permissions to pinpoint configurations where custom services execute group-writable code.
* **Dynamic Local Socket Mapping:** Audit hidden loopback-only sockets (`127.0.0.1` and `::1`) that bypass host firewalls to find unauthenticated internal HTTP endpoints or debuggers.

### Phase 3: Cryptographic & Active Directory Integration

* **Automated System Keyring (`keyctl`) Sniffing:** Native extraction of retained Linux kernel keyring profiles.
* **SSSD & Kerberos Target Enumeration:** Enumerate Linux systems integrated into Active Directory domains, checking `/etc/sssd/sssd.conf` permissions and automated ccache ticket locations for credential harvesting risks.

---

## License

This project is intended for authorized systems monitoring, configuration compliance checking, and defensive hardening validation. Please refer to the `LICENSE` file for more details.
