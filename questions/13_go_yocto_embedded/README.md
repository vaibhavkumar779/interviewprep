> **[← Back to All Topics](../README.md)**

## 📁 Files in This Section

| File | Description |
|------|-------------|
| **README.md** | Deep-dive learning guide (this file) |
| [complete.md](complete.md) | Complete question bank |
| [answers.md](answers.md) | All answers |

---

# Go, Yocto & Embedded — Deep-Dive Learning Guide

---

## 1. Go (Golang) — Overview for DevOps

Go is a statically typed, compiled language created at Google. Popular in DevOps because Docker, Kubernetes, Terraform, Prometheus — all written in Go.

### Why Go for DevOps?

```
✅ Single binary output (no runtime dependencies!)
✅ Cross-compilation (build for Linux on Windows)
✅ Fast compilation
✅ Built-in concurrency (goroutines, channels)
✅ Strong standard library (HTTP, JSON, crypto, testing)
✅ Static typing catches bugs at compile time
```

---

## 2. Go Basics

```go
package main

import (
    "fmt"
    "os"
    "strings"
)

// ─── Variables ───
func main() {
    // Type inference
    name := "DevOps"                    // short declaration
    var count int = 5                   // explicit type
    var active bool                     // zero value: false

    // Constants
    const maxRetries = 3

    // String operations
    fmt.Printf("Hello %s, count=%d\n", name, count)
    fmt.Println(strings.ToUpper(name))  // "DEVOPS"
    fmt.Println(len(name))              // 6

    // Conditionals
    if count > 3 {
        fmt.Println("High count")
    } else if count > 1 {
        fmt.Println("Medium count")
    } else {
        fmt.Println("Low count")
    }

    // For loop (only loop in Go — no while!)
    for i := 0; i < 5; i++ {
        fmt.Println(i)
    }

    // While-style
    retries := 0
    for retries < maxRetries {
        retries++
    }

    // Range (iterate over slice/map/string)
    servers := []string{"web1", "web2", "db1"}
    for i, server := range servers {
        fmt.Printf("%d: %s\n", i, server)
    }
}
```

### Data Structures

```go
// ─── Slices (dynamic arrays) ───
servers := []string{"web1", "web2"}
servers = append(servers, "web3")       // Add element
fmt.Println(len(servers))               // 3
sub := servers[0:2]                     // Slice of slice

// ─── Maps ───
config := map[string]string{
    "host": "localhost",
    "port": "8080",
}
config["env"] = "prod"                  // Add key
val, ok := config["host"]              // Check if key exists
if ok {
    fmt.Println(val)
}
delete(config, "env")                  // Remove key

// ─── Structs ───
type Server struct {
    Name     string
    IP       string
    Port     int
    IsActive bool
}

s := Server{Name: "web1", IP: "10.0.1.5", Port: 8080, IsActive: true}
fmt.Println(s.Name)
```

### Functions & Error Handling

```go
// Go functions return errors (no exceptions!)
func deploy(service string, version string) (string, error) {
    if service == "" {
        return "", fmt.Errorf("service name cannot be empty")
    }
    result := fmt.Sprintf("Deployed %s:%s", service, version)
    return result, nil     // nil = no error
}

// Caller MUST handle the error
result, err := deploy("web-api", "v2.0")
if err != nil {
    fmt.Fprintf(os.Stderr, "Error: %v\n", err)
    os.Exit(1)
}
fmt.Println(result)
```

### Concurrency (goroutines + channels)

```go
// Goroutine = lightweight thread (~2KB stack vs ~1MB OS thread)
func healthCheck(server string, results chan<- string) {
    // ... check server health
    results <- fmt.Sprintf("%s: healthy", server)  // Send to channel
}

func main() {
    servers := []string{"web1", "web2", "web3", "db1"}
    results := make(chan string, len(servers))       // Buffered channel

    for _, server := range servers {
        go healthCheck(server, results)              // Launch goroutine
    }

    for i := 0; i < len(servers); i++ {
        fmt.Println(<-results)                       // Receive from channel
    }
}
```

### Building & Cross-Compiling

```bash
# Build
go build -o myapp main.go           # Binary for current OS

# Cross-compile (from Windows → Linux binary!)
GOOS=linux GOARCH=amd64 go build -o myapp-linux main.go
GOOS=darwin GOARCH=arm64 go build -o myapp-mac main.go

# Run
go run main.go                       # Compile + run (dev only)

# Test
go test ./...                        # Run all tests
go test -v ./...                     # Verbose
go test -cover ./...                 # Coverage

# Modules
go mod init myproject                # Initialize module
go mod tidy                          # Clean dependencies
go get github.com/pkg/errors         # Add dependency
```

---

## 3. Yocto Project — Overview

Yocto is a **build framework** for creating custom Linux distributions for embedded devices.

```
What Yocto does:
  Takes:   Source packages + configuration + recipes
  Outputs: Complete Linux image (kernel + rootfs + bootloader)
           Customized for YOUR specific hardware

Why Yocto?
  - Build a minimal Linux for a router, IoT device, car infotainment
  - Only include what you need (tiny image: 8MB-200MB)
  - Reproducible builds (same input → same output)
  - Cross-compilation (build ARM image on x86)
```

### Yocto Architecture

```
┌─── Yocto Build System ────────────────────────────────────┐
│                                                            │
│  ┌──────────────┐                                         │
│  │  Metadata    │  Recipes (.bb), config, machine defs    │
│  │  (Layers)    │                                         │
│  └──────┬───────┘                                         │
│         │                                                  │
│  ┌──────▼───────┐                                         │
│  │  BitBake     │  Build engine (like make/cmake)         │
│  │  (scheduler) │  Parses recipes, resolves deps,         │
│  │              │  schedules tasks                        │
│  └──────┬───────┘                                         │
│         │                                                  │
│  ┌──────▼───────┐  ┌──────────────┐  ┌─────────────────┐ │
│  │  Fetch       │  │  Compile     │  │  Package        │ │
│  │  (download   │  │  (cross-     │  │  (create .rpm,  │ │
│  │   sources)   │  │   compile)   │  │   .deb, .ipk)   │ │
│  └──────────────┘  └──────────────┘  └─────────────────┘ │
│         │                                                  │
│  ┌──────▼────────────────────────────────────────────┐    │
│  │  Image Generation                                  │    │
│  │  (rootfs + kernel + bootloader = flashable image) │    │
│  └────────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────────┘
```

### Key Yocto Concepts

| Concept | Description |
|---------|------------|
| **Recipe (.bb)** | Instructions to build one package (fetch, compile, install) |
| **Layer** | Collection of recipes and config (meta-openembedded, meta-raspberrypi) |
| **BitBake** | Build engine — parses recipes, runs tasks |
| **Poky** | Reference distribution (Yocto's default) |
| **BSP (Board Support Package)** | Hardware-specific layer (kernel config, bootloader) |
| **Machine** | Target hardware configuration (MACHINE = "raspberrypi4") |
| **Distro** | Distribution policy (init system, libc, features) |
| **Image** | Final output — complete Linux system |

### Yocto Recipe Example

```bitbake
# recipes-apps/myapp/myapp_1.0.bb
SUMMARY = "My custom application"
DESCRIPTION = "A simple hello world application"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://LICENSE;md5=abc123"

SRC_URI = "git://github.com/myorg/myapp.git;branch=main;protocol=https"
SRCREV = "a1b2c3d4e5f6"

S = "${WORKDIR}/git"

inherit cmake       # Use cmake build system

do_install() {
    install -d ${D}${bindir}
    install -m 0755 ${B}/myapp ${D}${bindir}/myapp
}
```

### Yocto Build Commands

```bash
# Setup build environment
source oe-init-build-env build-dir

# Configure (conf/local.conf)
MACHINE = "raspberrypi4-64"
DISTRO = "poky"
IMAGE_INSTALL:append = " python3 nginx openssh"

# Build
bitbake core-image-minimal        # Minimal image (~8MB)
bitbake core-image-base            # Base image with networking
bitbake core-image-sato            # Full desktop image

# Build single recipe
bitbake myapp                      # Build just myapp
bitbake -c clean myapp             # Clean myapp
bitbake -c devshell myapp          # Open dev shell for debugging

# Output: tmp/deploy/images/<machine>/
#   core-image-minimal-raspberrypi4.wic  (flashable image)
#   zImage (kernel)
#   rootfs.tar.gz
```

### Yocto Layer Structure

```
meta-mylayer/
├── conf/
│   └── layer.conf                 # Layer metadata
├── recipes-apps/
│   └── myapp/
│       ├── myapp_1.0.bb           # Recipe
│       └── files/
│           └── myapp.service      # systemd service file
├── recipes-core/
│   └── images/
│       └── my-custom-image.bb     # Custom image recipe
└── README.md
```

---

## 4. Embedded Linux — Key Concepts

### Boot Process

```
Power On
    │
┌───▼───┐
│ BIOS/ │  Hardware init, find bootloader
│ UEFI  │
└───┬───┘
    │
┌───▼───────┐
│ Bootloader│  U-Boot, GRUB
│           │  Load kernel + device tree into RAM
└───┬───────┘
    │
┌───▼───────┐
│ Kernel    │  Hardware drivers, memory init
│           │  Mount root filesystem
└───┬───────┘
    │
┌───▼───────┐
│ Init      │  systemd (PID 1)
│ System    │  Start services, mount filesystems
└───┬───────┘
    │
┌───▼───────┐
│ Userspace │  Applications, daemons
│           │  System ready!
└───────────┘
```

### Cross-Compilation

```
Build Host (x86_64 laptop)     Target Device (ARM)
┌──────────────────────┐       ┌──────────────────────┐
│  Cross-compiler:     │       │                      │
│  arm-linux-gnueabi-  │──────►│  Runs ARM binary     │
│  gcc                 │       │  (can't run x86!)    │
│                      │       │                      │
│  Builds ARM binary   │       │  Embedded Linux      │
│  on x86 machine      │       │  (Yocto output)      │
└──────────────────────┘       └──────────────────────┘

Why cross-compile?
  - Target device too slow to compile (minutes vs hours)
  - Target may have limited storage
  - CI/CD runs on x86 servers, deploys to ARM devices
```

---

## 5. DevOps for Embedded

```
┌─── Embedded CI/CD Pipeline ───────────────────────────────┐
│                                                            │
│  Code Push ──► Build (Yocto/cross-compile)                │
│                  │                                         │
│               ┌──▼───────────┐                            │
│               │ Unit Tests   │  Run on build host (QEMU)  │
│               └──┬───────────┘                            │
│                  │                                         │
│               ┌──▼───────────┐                            │
│               │ Flash to     │  Deploy to test device      │
│               │ Test Board   │  (or emulator)             │
│               └──┬───────────┘                            │
│                  │                                         │
│               ┌──▼───────────┐                            │
│               │ Integration  │  Hardware-in-the-loop tests│
│               │ Tests        │  (serial, GPIO, network)   │
│               └──┬───────────┘                            │
│                  │                                         │
│               ┌──▼───────────┐                            │
│               │ OTA Update   │  Deploy to fleet of devices│
│               │ (SWUpdate,   │  (staged rollout)          │
│               │  Mender)     │                            │
│               └──────────────┘                            │
└────────────────────────────────────────────────────────────┘

Challenges:
  - Long build times (Yocto: 1-8 hours for full build)
  - Hardware dependencies (need actual boards for testing)
  - OTA updates (can't just redeploy like cloud apps)
  - Regulatory compliance (safety-critical, certifications)
```

### OTA (Over-the-Air) Updates

```
Tools for embedded OTA:
  Mender     — open source, client-server, A/B partition
  SWUpdate   — lightweight, dual-bank updates
  RAUC       — redundant A/B updating
  Balena     — container-based IoT fleet management

A/B Partition Strategy:
  ┌──────────┐  ┌──────────┐
  │ Slot A   │  │ Slot B   │
  │ (active) │  │ (update) │
  │ v1.0     │  │ v1.1     │  ← download new version here
  └──────────┘  └──────────┘
                       │
       If v1.1 boots OK → mark as active
       If v1.1 fails    → revert to Slot A (v1.0)
       Always have a working fallback!
```

---

## 6. Go in DevOps Tools

```
Tools written in Go (why it matters for Ciena):

Container & Orchestration:
  Docker (dockerd, containerd)
  Kubernetes (all components)
  Helm
  Podman

IaC & Config:
  Terraform
  Packer
  Consul
  Vault

CI/CD:
  Drone CI
  Tekton

Monitoring:
  Prometheus
  Grafana (backend)
  Jaeger

Networking:
  CoreDNS
  Envoy (control plane)
  Cilium
  Traefik

Security:
  Trivy
  Falco

Understanding Go helps you:
  - Read source code of these tools
  - Write plugins and extensions
  - Build custom operators for K8s
  - Write CLI tools for automation
```

---

## 7. Go CLI Tool Example

```go
// Simple DevOps CLI tool
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "time"
)

type HealthResponse struct {
    Status  string `json:"status"`
    Version string `json:"version"`
}

func checkHealth(url string, timeout time.Duration) (*HealthResponse, error) {
    client := &http.Client{Timeout: timeout}
    resp, err := client.Get(url)
    if err != nil {
        return nil, fmt.Errorf("request failed: %w", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != 200 {
        return nil, fmt.Errorf("unhealthy: status %d", resp.StatusCode)
    }

    var health HealthResponse
    if err := json.NewDecoder(resp.Body).Decode(&health); err != nil {
        return nil, fmt.Errorf("decode failed: %w", err)
    }

    return &health, nil
}

func main() {
    if len(os.Args) < 2 {
        fmt.Fprintf(os.Stderr, "Usage: %s <url>\n", os.Args[0])
        os.Exit(1)
    }

    health, err := checkHealth(os.Args[1], 5*time.Second)
    if err != nil {
        fmt.Fprintf(os.Stderr, "ERROR: %v\n", err)
        os.Exit(1)
    }

    fmt.Printf("Status: %s, Version: %s\n", health.Status, health.Version)
}
```
