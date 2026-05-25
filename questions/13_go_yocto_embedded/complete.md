# Go, Yocto & Embedded DevOps - COMPLETE (Critical for Ciena)
## Questions Only - Test Yourself

### Go (Golang) Basics
1. What is Go? Who created it and why?
2. What are Go's key features? Name 5.
3. What is the basic structure of a Go program?
4. How do you compile and run a Go program? (`go build`, `go run`)
5. What is `go mod init`? What is `go.mod`?
6. What is `go get`? What is `go mod tidy`?
7. What are goroutines? How are they different from threads?
8. What are channels in Go? What are they used for?
9. What is the difference between Go and Python for DevOps tooling?
10. Why are many DevOps tools written in Go? (Docker, K8s, Terraform, Prometheus)
11. How do you cross-compile Go binaries? (GOOS, GOARCH)
12. Why is cross-compilation important for embedded systems?
13. What is `go test`? How do you write tests in Go?
14. What is a Go interface?
15. How do you handle errors in Go? (no try/catch)

### Yocto Project (CRITICAL for Ciena ON team)
16. What is the Yocto Project?
17. What problem does Yocto solve?
18. What is embedded Linux? How is it different from desktop/server Linux?
19. Why would an optical networking company (Ciena) use Yocto?
20. What is BitBake? What is its role in Yocto?
21. What is a recipe (.bb file)? What does it contain?
22. What is a layer in Yocto? How do you create one?
23. What is the naming convention for layers? (meta-*)
24. What is Poky? How does it relate to Yocto?
25. What is OpenEmbedded? How does it relate to Yocto?
26. What is a BSP (Board Support Package)?
27. What is a Yocto image? What types exist? (core-image-minimal, core-image-full)
28. What is the build workflow in Yocto? (fetch, unpack, patch, configure, compile, install, package, image)
29. What is `source oe-init-build-env`? What does it set up?
30. What is `local.conf`? What do you configure in it?
31. What is `bblayers.conf`?
32. How do you add a custom package to a Yocto image?
33. What is a recipe append file (.bbappend)?
34. What is `MACHINE` variable? What does it configure?
35. What is `DISTRO` variable?
36. How do you debug a failed Yocto build?
37. What are Yocto build artifacts? Where are they output?
38. How long does a typical Yocto build take? How do you speed it up?
39. What is sstate cache in Yocto? How does it improve build times?
40. What is the shared state cache? How does it relate to CI?

### Embedded DevOps (How CI/CD works in embedded)
41. How is CI/CD for embedded different from web applications?
42. What are the challenges of CI/CD for embedded systems?
43. How do you test embedded software in CI? (emulators, QEMU, hardware-in-the-loop)
44. What is QEMU? How is it used in embedded CI?
45. What is a cross-compiler? Why is it needed?
46. What is a toolchain in embedded development?
47. How do you automate Yocto builds in Jenkins?
48. What is a nightly build? Why is it common in embedded development?
49. How do you version embedded software? (firmware versioning)
50. What is OTA (Over-the-Air) updates for embedded devices?
51. How do you handle hardware-specific testing in CI?
52. What is the difference between a host build and a target build?

### Optical Networking Awareness (Ciena-specific)
53. What is optical networking at a high level?
54. What is a DWDM (Dense Wavelength Division Multiplexing)?
55. What is a transponder/muxponder in optical networks?
56. Why does optical networking hardware need custom embedded Linux?
57. What kind of software runs on optical network devices? (firmware, management plane, control plane)
58. What is NETCONF/YANG? Why is it relevant to network devices?
59. What is the role of DevOps in an optical networking software team?

### Interview-Style
60. You've never used Yocto before. How would you ramp up in 2 weeks?
61. How would you set up a CI pipeline for a Yocto-based project?
62. A Yocto build takes 4 hours. How would you optimize the CI pipeline?
63. How would you manage recipes and layers across multiple product variants?
64. The team uses Google Repo + Gerrit + Yocto + Jenkins. How do you fit in as DevOps?
65. How would you implement automated testing for embedded firmware?
66. What DevOps practices from web/cloud can you bring to embedded development?
67. Why did you apply for this role despite not having Go/Yocto experience?
68. How do you handle learning new build systems and toolchains?
69. What similarities do you see between Azure DevOps pipelines and Jenkins for embedded CI?
70. How would you automate the release process for an embedded software product?
