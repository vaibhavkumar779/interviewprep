# Go, Yocto & Embedded DevOps - COMPREHENSIVE ANSWERS (All 70 Questions)

---

## Go (Golang) Basics

**1. What is Go? Who created it and why?**
Go (Golang) was created at Google in 2007 by Robert Griesemer, Rob Pike, and Ken Thompson. Designed for: simplicity, fast compilation, built-in concurrency, and efficient systems programming. Released open-source in 2009.

**2. Key features? Name 5.**
1. Statically typed, compiled to single binary
2. Built-in concurrency (goroutines + channels)
3. Garbage collected
4. Fast compilation
5. Cross-compilation out of the box
6. Simple syntax (no inheritance, no exceptions, no generics until Go 1.18)

**3. Basic structure of Go program?**
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```
- `package main`: Entry point package
- `import`: Dependencies
- `func main()`: Entry function

**4. Compile and run?**
```bash
go run main.go         # Compile and run immediately
go build -o myapp      # Compile to binary
./myapp                # Run binary
```

**5. `go mod init`? `go.mod`?**
```bash
go mod init github.com/user/project
```
Creates `go.mod` — the module file tracking dependencies (like package.json or requirements.txt):
```
module github.com/user/project
go 1.21
require github.com/gorilla/mux v1.8.0
```

**6. `go get`? `go mod tidy`?**
- `go get github.com/pkg/errors`: Download and add dependency
- `go mod tidy`: Remove unused deps, add missing ones. Run after changing imports.

**7. Goroutines? vs threads?**

Lightweight concurrent functions. `go myFunction()` starts a goroutine.

```
OS Threads:                         Go Goroutines:
┌────────────────────────┐      ┌────────────────────────┐
│ ~1MB stack per thread   │      │ ~2KB stack per goroutine │
│ OS-managed              │      │ Go runtime-managed       │
│ Expensive context switch│      │ Cheap context switch     │
│ Thousands max           │      │ Millions possible!       │
│                          │      │                          │
│ ┌───┐ ┌───┐ ┌───┐       │      │ ┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐  │
│ │ T1│ │ T2│ │ T3│       │      │ │G││G││G││G││G││G││G│  │
│ └───┘ └───┘ └───┘       │      │ └─┘└─┘└─┘└─┘└─┘└─┘└─┘  │
│ 3 threads = 3MB          │      │ 7 goroutines = ~14KB     │
│                          │      │ Multiplexed onto OS      │
│                          │      │ threads by Go scheduler  │
└────────────────────────┘      └────────────────────────┘
```

- **Goroutines**: ~2KB stack, managed by Go runtime, can run millions
- **OS threads**: ~1MB stack, managed by OS, thousands max
- Go runtime multiplexes goroutines onto OS threads.

**8. Channels?**
Communication mechanism between goroutines:
```go
ch := make(chan string)
go func() {
    ch <- "hello"    // Send to channel
}()
msg := <-ch          // Receive from channel
```
Buffered: `make(chan string, 10)` — non-blocking until full.

**9. Go vs Python for DevOps?**
| Go | Python |
|---|---|
| Compiled single binary | Interpreted, needs runtime |
| Fast execution | Slower |
| Static typing | Dynamic typing |
| Better for: CLI tools, agents, high-perf services | Better for: scripts, automation, quick prototypes |
| Harder to learn | Easier to learn |

**10. Why DevOps tools in Go?**
Docker, K8s, Terraform, Prometheus, Helm — all in Go because:
- Single binary deployment (no runtime dependency)
- Cross-compilation (build Linux binary on Mac)
- High performance and low memory
- Great concurrency (handle many connections)
- Strong standard library (net/http, os, etc.)

**11. Cross-compile?**
```bash
GOOS=linux GOARCH=amd64 go build -o myapp-linux
GOOS=linux GOARCH=arm64 go build -o myapp-arm
GOOS=windows GOARCH=amd64 go build -o myapp.exe
```

**12. Why cross-compilation important for embedded?**
Embedded devices (ARM, MIPS) can't compile locally — too slow, limited resources. You compile on a powerful x86 machine targeting the embedded architecture.

**13. `go test`?**
```go
// math_test.go
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Expected 5, got %d", result)
    }
}
```
```bash
go test ./...          # Run all tests
go test -v -cover      # Verbose + coverage
```

**14. Go interface?**
Implicit interface — any type implementing the methods satisfies it:
```go
type Logger interface {
    Log(message string)
}
type ConsoleLogger struct{}
func (c ConsoleLogger) Log(message string) {
    fmt.Println(message)
}
// ConsoleLogger automatically satisfies Logger
```

**15. Error handling?**
No try/catch. Functions return errors explicitly:
```go
result, err := doSomething()
if err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}
```

---

## Yocto Project (CRITICAL for Ciena)

**16. What is Yocto?**
Open-source project for building custom Linux distributions for embedded systems. Produces bootable images tailored to specific hardware. Not a Linux distro itself — a tool to create one.

**17. What problem does Yocto solve?**
Embedded devices need stripped-down, customized Linux. Yocto lets you: select exactly which packages to include, support custom hardware, produce minimal images (MB not GB), ensure reproducible builds.

**18. Embedded Linux vs desktop/server Linux?**
| Embedded | Desktop/Server |
|---|---|
| Minimal footprint (MBs) | Full OS (GBs) |
| Specific hardware (ARM, MIPS) | x86/x64 |
| Custom kernel | General kernel |
| Flash/ROM storage | HDD/SSD |
| Real-time requirements | General purpose |
| No GUI typically | Full desktop |
| Long lifecycle (10+ years) | Frequent updates |

**19. Why would Ciena use Yocto?**
Ciena builds optical networking hardware (transponders, routers). Each device needs: custom Linux tailored to specific networking chips, minimal attack surface, optimized for embedded processors, specific kernel drivers, long-term support. Yocto enables all of this.

**20. What is BitBake?**
The build engine/task scheduler of Yocto. Like `make` but for embedded Linux. Reads recipes (.bb files), resolves dependencies, and executes tasks (fetch, compile, package, image creation).

**21. Recipe (.bb file)?**
Instructions to build one software package:
```bitbake
SUMMARY = "My Application"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://LICENSE;md5=xxx"
SRC_URI = "git://github.com/user/myapp.git;branch=main"
SRCREV = "abc123"
S = "${WORKDIR}/git"

do_compile() {
    oe_runmake
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 myapp ${D}${bindir}
}
```

**22. Layer? How to create?**
Organized collection of recipes, configurations, and classes:

```
Yocto Layer Structure:

  meta-ciena/                         ← Custom layer for Ciena
  ├── conf/
  │   ├── layer.conf                ← Layer configuration (priority, etc.)
  │   ├── machine/
  │   │   └── ciena-ncs.conf        ← Machine-specific config
  │   └── distro/
  │       └── ciena-distro.conf     ← Distribution config
  ├── recipes-core/
  │   ├── systemd/
  │   │   └── systemd_%.bbappend    ← Modify existing recipe
  │   └── base-files/
  │       └── base-files_%.bbappend
  ├── recipes-apps/
  │   └── my-app/
  │       ├── my-app_1.0.bb         ← Custom recipe
  │       └── files/
  │           └── my-app.service    ← Supporting files
  ├── recipes-images/
  │   └── ciena-image.bb           ← Custom image recipe
  └── classes/
      └── ciena-qa.bbclass         ← Custom QA class

Layer stacking (priority order):
  meta-ciena (9)         ← highest priority (overrides)
  meta-openembedded (7)
  meta-poky (5)
  meta (OE-Core) (1)    ← base layer
```

```bash
bitbake-layers create-layer meta-mycompany
# Creates:
# meta-mycompany/
#   ├── conf/layer.conf
#   ├── recipes-example/
#   └── README
```
Add to build: `bitbake-layers add-layer meta-mycompany`

**23. Naming convention?**
All layers prefixed with `meta-`: `meta-poky`, `meta-openembedded`, `meta-ciena`, `meta-networking`, `meta-raspberrypi`.

**24. Poky?**
Reference distribution of Yocto. Contains: BitBake + meta-poky + meta-yocto-bsp + OE-Core. Starting point for custom distros.

**25. OpenEmbedded?**
Build framework that Yocto is built on top of. Provides: OE-Core (core recipes, classes), the build system architecture. Yocto = OpenEmbedded + reference distro (Poky) + tools + documentation.

**26. BSP (Board Support Package)?**
Layer providing support for specific hardware: kernel configuration, bootloader, device tree, hardware-specific drivers. Example: `meta-raspberrypi`, `meta-intel`.

**27. Yocto image types?**
- `core-image-minimal`: Bare minimum bootable image (~8MB)
- `core-image-base`: Console-only with hardware support
- `core-image-full-cmdline`: Full command-line tools
- `core-image-sato`: Reference GUI image
- Custom images defined in recipes

**28. Build workflow?**

```
Yocto/BitBake Build Workflow:

  Source Code                                              Output
  ┌────────┐     ┌────────┐     ┌────────┐     ┌────────┐     ┌──────────────┐
  │ Fetch   │────▶│ Unpack │────▶│ Patch  │────▶│Configure│────▶│ Compile      │
  │ (git,   │     │ (extract│    │ (apply │     │(autoconf│     │ (make,       │
  │  http,  │     │  tar)   │    │ .patch)│     │ cmake)  │     │  cmake)      │
  │  local) │     └────────┘    └────────┘     └────────┘     └──────┬───────┘
  └────────┘                                                        │
                                                                     ▼
  ┌──────────────┐     ┌────────────┐     ┌─────────────┐     ┌──────────────┐
  │  Image        │◀────│  Package   │◀────│  Install    │◀────│              │
  │  (rootfs      │     │  (.deb,    │     │  (staging   │     │              │
  │   .ext4,      │     │   .rpm,    │     │   area)     │     │              │
  │   .wic)       │     │   .ipk)    │     │             │     │              │
  └──────────────┘     └────────────┘     └─────────────┘     └──────────────┘

  sstate-cache: Caches output of EACH step
  If inputs unchanged → skip step → use cached output
  First build: 2-8 hours  │  With sstate: ~30 minutes
```

1. **Fetch**: Download source code (git, http, local)
2. **Unpack**: Extract sources
3. **Patch**: Apply patches
4. **Configure**: Run configure scripts (autoconf, cmake)
5. **Compile**: Build from source
6. **Install**: Install to staging area
7. **Package**: Create .deb/.rpm/.ipk packages
8. **Image**: Assemble rootfs image

**29. `source oe-init-build-env`?**
Sets up build environment:
- Creates `build/` directory
- Sets PATH to include BitBake
- Creates initial `conf/local.conf` and `conf/bblayers.conf`
- Must run in every new terminal session

**30. `local.conf`?**
Main build configuration:
```
MACHINE = "qemuarm64"           # Target hardware
DISTRO = "poky"                  # Distribution
DL_DIR = "/downloads"            # Source download cache
SSTATE_DIR = "/sstate-cache"     # Shared state cache
PARALLEL_MAKE = "-j 8"           # Build parallelism
BB_NUMBER_THREADS = "8"          # BitBake parallelism
IMAGE_INSTALL:append = " myapp"  # Add package to image
```

**31. `bblayers.conf`?**
Lists which layers are included in the build:
```
BBLAYERS = " \
  /path/to/poky/meta \
  /path/to/poky/meta-poky \
  /path/to/meta-openembedded/meta-oe \
  /path/to/meta-ciena \
"
```

**32. Add custom package to image?**
1. Write recipe in your layer: `meta-mycompany/recipes-apps/myapp/myapp_1.0.bb`
2. Add to image: `IMAGE_INSTALL:append = " myapp"` in `local.conf` or image recipe
3. Build: `bitbake core-image-minimal`

**33. Recipe append (.bbappend)?**
Modify existing recipe without editing it:
```
# meta-mycompany/recipes-core/systemd/systemd_%.bbappend
FILESEXTRAPATHS:prepend := "${THISDIR}/files:"
SRC_URI += "file://custom.conf"
```
`%` matches any version. Keeps customizations in your layer.

**34. MACHINE variable?**
Defines target hardware platform:
```
MACHINE = "qemuarm64"     # ARM64 emulator
MACHINE = "raspberrypi4"  # Raspberry Pi 4
MACHINE = "intel-corei7-64"  # Intel x86_64
```
Determines: kernel config, bootloader, hardware drivers, architecture.

**35. DISTRO variable?**
Defines distribution policy: init system, libc, package format, features:
```
DISTRO = "poky"            # Reference distro
DISTRO = "ciena-distro"    # Custom distro
```

**36. Debug failed build?**
```bash
bitbake myapp -c compile -f    # Force rerun compile task
bitbake myapp -e | less        # Show all variables
cat tmp/work/<arch>/myapp/1.0/temp/log.do_compile   # Build log
cat tmp/work/<arch>/myapp/1.0/temp/run.do_compile    # Exact commands run
bitbake myapp -DDD             # Maximum debug output
```

**37. Build artifacts?**
Output in `tmp/deploy/images/<MACHINE>/`:
- Root filesystem: `core-image-minimal-<machine>.ext4`
- Kernel: `zImage` or `Image`
- Device tree: `*.dtb`
- SDK: `tmp/deploy/sdk/`
- Packages: `tmp/deploy/rpm/` or `/deb/` or `/ipk/`

**38. Build time? Speed up?**
First build: 2-8 hours. Incremental: minutes. Speed up:
1. **sstate cache**: Reuse previously built artifacts
2. More CPU/RAM (32GB+ RAM, 8+ cores recommended)
3. SSD storage
4. Shared downloads directory (`DL_DIR`)
5. `PARALLEL_MAKE = "-j $(nproc)"`
6. `BB_NUMBER_THREADS = "$(nproc)"`
7. Use `tmpfs` for build directory

**39. sstate cache?**
Shared State cache: stores output of each build task as tarball. If inputs haven't changed, reuses cached output. Massively reduces rebuild time. Can be shared across machines (NFS, HTTP server).

**40. Shared state cache and CI?**
CI server maintains sstate cache. All CI builds contribute to and read from same cache. First build: 4 hours. Subsequent builds reuse cached tasks: 30 minutes. Setup: sstate cache on network storage, all Jenkins agents mount it.

---

## Embedded DevOps

**41. CI/CD for embedded vs web apps?**
| Embedded | Web Apps |
|---|---|
| Hours-long builds | Minutes |
| Cross-compilation required | Native compilation |
| Hardware testing needed | Software-only testing |
| Binary images/firmware | Container images |
| OTA updates | Rolling deploys |
| Long release cycles | Continuous deployment |
| Strict versioning | Semantic versioning |

**42. Challenges of CI/CD for embedded?**
1. Long build times (hours)
2. Need physical hardware for full testing
3. Cross-compilation complexity
4. Large binary artifacts (GBs)
5. Flashing/deploying to hardware
6. Hardware availability for testing
7. Regulatory/certification requirements

**43. Testing in CI?**
1. **Unit tests**: Run natively on build host
2. **QEMU emulation**: Run image in virtual hardware
3. **Hardware-in-the-loop (HIL)**: Flash actual hardware, run tests
4. **Static analysis**: Coverity, cppcheck
5. **Integration tests**: Test component interactions in emulator

**44. QEMU?**
Open-source hardware emulator. Emulates ARM, MIPS, x86, etc. In embedded CI: boot Yocto image in QEMU, run automated tests without physical hardware. `runqemu qemuarm64` in Yocto.

**45. Cross-compiler?**
Compiler that runs on one architecture (x86 host) but produces binaries for another (ARM target). Needed because embedded devices are too slow/limited to compile locally.

**46. Toolchain?**
Complete set of tools for cross-development: cross-compiler (gcc-arm), linker, libraries (libc), debugger (gdb), headers. Yocto generates toolchains via `bitbake meta-toolchain` or `-c populate_sdk`.

**47. Automate Yocto builds in Jenkins?**
```groovy
pipeline {
    agent { label 'yocto-builder' }
    stages {
        stage('Setup') {
            steps {
                sh 'source oe-init-build-env build'
            }
        }
        stage('Build') {
            steps {
                sh '''
                    source oe-init-build-env build
                    bitbake core-image-minimal
                '''
            }
        }
        stage('Test') {
            steps {
                sh 'runqemu qemuarm64 nographic &'
                sh 'run-tests.sh'
            }
        }
        stage('Archive') {
            steps {
                archiveArtifacts 'build/tmp/deploy/images/**/*'
            }
        }
    }
}
```

**48. Nightly build?**
Scheduled build running every night. Common in embedded because: builds take hours, catch integration issues early, always have a recent testable image. Jenkins: `triggers { cron('H 0 * * *') }`.

**49. Firmware versioning?**
```
<major>.<minor>.<patch>-<build>
Example: 3.2.1-456
```
- Major: breaking changes
- Minor: features
- Patch: bug fixes
- Build: CI build number
Track in `/etc/firmware-version` on device.

**50. OTA updates?**
Update firmware remotely without physical access:
- Tools: SWUpdate, Mender, RAUC, hawkBit

```
A/B Partition OTA Update:

  Flash Memory:
  ┌─────────────────────────────────────────────┐
  │ Bootloader │ Partition A (active) │ Partition B (standby)│
  │            │ v3.1.0 ★ booting   │ v3.0.0 (old)         │
  └────────────┴─────────────────────┴─────────────────────┘

  OTA Update Process:
  1. Download v3.2.0 to Partition B (standby)
  2. Verify checksum + signature
  3. Set bootloader to boot from B next time
  4. Reboot → boots into B (v3.2.0)
  5. Health check passes → mark B as active
  6. Health check fails  → revert to A (v3.1.0)

  Result: Zero downtime, guaranteed rollback!
```

- Strategy: A/B partition scheme (boot from A, update B, switch on success, rollback if failure)
- Security: Signed images, encrypted transport

**51. Hardware-specific testing in CI?**
- Dedicated hardware lab with test devices
- Jenkins agents connected to hardware via serial/JTAG
- Flash firmware → boot → run test suite → collect results
- Device farm management tools
- Reserve/release hardware for test runs

**52. Host build vs target build?**
- **Host build**: Compiled to run on build machine (x86). For: build tools, code generators
- **Target build**: Cross-compiled to run on embedded device (ARM). For: application, firmware
- Yocto handles both: `DEPENDS` (host tools) vs `RDEPENDS` (target runtime)

---

## Optical Networking Awareness

**53. Optical networking at high level?**
Transmitting data as light pulses through fiber optic cables. Enables: internet backbone, long-distance telecom, data center interconnects. Orders of magnitude faster than electrical. Ciena is a major player.

**54. DWDM?**
Dense Wavelength Division Multiplexing: send multiple light signals (wavelengths/colors) simultaneously through one fiber. Each wavelength carries independent data. Multiplies fiber capacity (80+ channels on one fiber).

**55. Transponder/muxponder?**
- **Transponder**: Converts client signal (Ethernet) to optical wavelength for transport. One client to one wavelength.
- **Muxponder**: Multiplexes multiple client signals onto fewer wavelengths. More efficient use of spectrum.

**56. Why custom embedded Linux?**
Optical networking hardware needs: specific kernel drivers for ASICs/DSPs, real-time performance, minimal footprint (limited flash), long-term stability (10+ year deployments), security hardening, specific management protocols (NETCONF/YANG).

**57. Software on optical network devices?**
- **Firmware**: Low-level hardware control (FPGA, DSP)
- **Control plane**: Routing, signaling, path computation
- **Management plane**: NETCONF/YANG, SNMP, CLI, REST APIs
- **Data plane**: Packet/optical forwarding (hardware-accelerated)

**58. NETCONF/YANG?**
- **NETCONF**: Network management protocol (like SSH for config). Operations: get, edit-config, commit, rollback.
- **YANG**: Data modeling language defining device configuration schema.
- Used by all modern network devices. Enables automation (Ansible `netconf` module).

**59. DevOps role in optical networking team?**
- CI/CD pipelines for Yocto builds
- Automated testing (QEMU + hardware-in-the-loop)
- Release management and firmware versioning
- Build infrastructure (Jenkins, artifact storage)
- Developer tooling and environment setup
- Deployment automation (OTA updates)

---

## Interview-Style

**60. Ramp up on Yocto in 2 weeks?**
**Week 1:**
- Days 1-2: Yocto docs + tutorials, build Poky for QEMU
- Days 3-4: Understand team's layer structure, build system, CI pipeline
- Day 5: Write a simple recipe, add custom package to image

**Week 2:**
- Days 6-7: Study team's custom layers and recipes
- Days 8-9: Understand CI/CD pipeline (Jenkins + Yocto), fix a simple build issue
- Day 10: Set up local dev environment mirroring CI, start contributing

"I'd pair with experienced team members, focus on the build pipeline first since that's my DevOps strength, and learn Yocto specifics in context."

**61. CI pipeline for Yocto project?**
1. **Trigger**: Git push / Gerrit merge / nightly schedule
2. **Repo sync**: `repo sync` to get all layers
3. **Build environment**: `source oe-init-build-env`
4. **sstate cache**: Mount shared cache for fast rebuilds
5. **Build**: `bitbake <image>` with parallelism
6. **Unit tests**: Run on host
7. **QEMU tests**: Boot image, run automated tests
8. **Archive**: Store image artifacts in Artifactory
9. **Notify**: Build status to team (Slack/email)

**62. Optimize 4-hour Yocto build?**
1. **sstate cache**: Biggest win — reuse cached tasks (4hr → 30min for incremental)
2. **Powerful build agents**: 32+ cores, 64GB RAM, NVMe SSD
3. **Shared DL_DIR**: Don't re-download sources
4. **Parallel builds**: `BB_NUMBER_THREADS` and `PARALLEL_MAKE`
5. **Build only changed layers**: Trigger selective builds
6. **tmpfs for /tmp**: RAM-based temporary storage
7. **Distributed builds**: icecream/distcc for distributed compilation
8. **Docker build agents**: Consistent environment, easy scaling

**63. Manage recipes across product variants?**
- Common base layer (`meta-ciena-common`) shared across products
- Product-specific layers (`meta-ciena-product-a`, `meta-ciena-product-b`)
- Use `MACHINE` and `DISTRO` for hardware/feature variants
- `.bbappend` files for product-specific customizations
- CI matrix builds: build all variants nightly

**64. Repo + Gerrit + Yocto + Jenkins — how to fit in?**
- **Google Repo**: Multi-repo management (manifest file lists all git repos). `repo sync` fetches all.
- **Gerrit**: Code review platform. Changes submitted as patchsets, reviewed, then merged.
- **My role**: Maintain Jenkins pipelines that: trigger on Gerrit merge events → `repo sync` → Yocto build → test → archive. Optimize build times, manage sstate cache, set up developer environments, improve CI reliability.

**65. Automated testing for embedded firmware?**
1. **Static analysis**: cppcheck, Coverity in CI
2. **Unit tests**: GoogleTest/CUnit on host
3. **QEMU integration tests**: Boot firmware, test APIs, verify services start
4. **Hardware-in-the-loop**: Jenkins agent connected to test boards, flash + test
5. **Regression suite**: Automated test runs on every build
6. **Performance tests**: Benchmark specific operations

**66. DevOps practices from web/cloud for embedded?**
1. **CI/CD pipelines**: Automated build + test (adapted for long builds)
2. **Infrastructure as Code**: Build environments in Docker/Ansible
3. **Monitoring**: Build metrics, test dashboards
4. **GitOps**: Version-controlled configuration
5. **Automated testing**: Shift-left testing
6. **Artifact management**: Proper versioning and storage
7. **ChatOps**: Build notifications in Slack

**67. Why apply without Go/Yocto experience?**
"My DevOps fundamentals — CI/CD, pipelines, automation, containers, Kubernetes, scripting — are directly transferable. Build systems share common concepts (dependencies, caching, parallelism). I've quickly learned new technologies before. The job requires DevOps expertise with ability to learn domain-specific tools, not someone who only knows Yocto. I'm excited about the embedded/optical networking domain."

**68. Learning new build systems and toolchains?**
"I follow a pattern: 1) Read official docs for fundamentals, 2) Build something simple end-to-end, 3) Study the team's existing setup, 4) Pair with experts, 5) Take on small tasks first. For Yocto, I'd build a basic image in QEMU within day 1, then study the team's layers and CI pipeline."

**69. Azure DevOps pipelines vs Jenkins for embedded CI?**
| Similarity | Implementation |
|---|---|
| Pipeline as code | YAML (Azure) vs Jenkinsfile (Jenkins) |
| Agent pools | Self-hosted agents for Yocto builds |
| Artifact management | Both support artifact storage |
| Parallel stages | Both support parallel execution |
| Triggers | Both support git, schedule, manual |
| Caching | Both support caching (sstate equivalent) |

Key difference: Jenkins has deeper embedded CI ecosystem, more plugins for hardware testing.

**70. Automate release process for embedded product?**
1. **Version tagging**: Git tag triggers release pipeline
2. **Full build**: Clean build from tag (no cache, reproducibility)
3. **Complete test suite**: Unit + QEMU + hardware tests
4. **Sign firmware**: Cryptographic signing for security
5. **Generate release notes**: From Git commits/Jira tickets
6. **Archive artifacts**: Store signed image in Artifactory with metadata
7. **Deploy to staging**: OTA update to test devices
8. **Approval gate**: Manual approval for production release
9. **Production release**: Push to OTA update server
10. **Notify stakeholders**: Release email/Slack notification
