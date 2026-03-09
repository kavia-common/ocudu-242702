# Repository Analysis

## High-level purpose of the repository

This repository contains the OCUDU radio access network (RAN) software stack. The top-level repository description states that OCUDU is a permissively licensed, open-source 5G (and beyond) CU/DU project that is designed for commercial deployment and research, is compliant with 3GPP and O-RAN Alliance specifications, and includes a full Layer 1/2/3 implementation with minimal external dependencies. In addition to the core RAN implementation, the repository contains build tooling, configuration examples, containerized deployment assets, and developer-oriented documentation tooling.

From the code layout and build system, the repository is intended to build multiple deployable binaries for different node topologies, including a co-located gNB (CU + DU), a combined CU (CU-CP + CU-UP), and a DU that connects to a CU split over standard interfaces.

## Main components

The repository is organized as a monolithic CMake-based C++ project with clear separations between public headers, library implementations, application entry points, tests, and operations tooling.

### Core libraries (`include/` and `lib/`)

The bulk of the protocol stack and platform support is implemented as libraries under `include/ocudu/**` (public headers) and `lib/**` (implementations). The `lib/CMakeLists.txt` enumerates the major library areas that make up the stack, including:

Centralized Unit and Distributed Unit layers and protocol stacks:

- `cu_cp`, `cu_up`, and `du`, which implement the CU control-plane, CU user-plane, and DU functionality.
- `rrc`, `pdcp`, `rlc`, `sdap`, `mac`, `scheduler`, and `phy`, which provide the L3/L2/L1 protocol and scheduling/PHY processing components typically required for a 5G gNB.
- `ngap`, `f1ap`, `e1ap`, and `xnap`, which implement key control-plane interfaces (N2, F1, E1, and Xn).
- `f1u` and `gtpu`, which provide user-plane transport (F1-U and GTP-U).
- `e2` and `asn1`, which provide E2/E2SM support and ASN.1 types/encoding used across multiple interfaces.
- `ofh` and `ru`, which provide Open Fronthaul-related components and Radio Unit implementations (including an SDR-focused RU implementation).
- `radio`, which provides radio backends and integration points (for example UHD and ZMQ are configurable via build options).
- `gateways`, which is conditionally built when SCTP support is enabled, and provides network gateway functionality used by various interfaces.

Support libraries and cross-cutting concerns:

- `support`, `instrumentation`, `pcap`, and `ocudulog`, which provide utilities and infrastructure including timers, tracing, logging, metrics plumbing, and packet capture support.
- `security`, which is conditionally built when mbedTLS is enabled, and provides cryptographic/security-related functionality.
- `ocuduvec` and `hal`, which provide vectorized/SIMD utilities and (optionally) hardware acceleration integration. DPDK support is present behind build options and conditional compilation.

### Applications (`apps/`)

The `apps/` tree contains build targets and entry points for runnable binaries and supporting services. The top-level `apps/CMakeLists.txt` shows that core node applications are built when SCTP and mbedTLS support are not disabled, and additional examples, helpers, and services are always built.

Key application entry points include:

- `apps/gnb/gnb.cpp`, which implements a co-located gNB application that combines CU-CP, CU-UP, and DU in one process. The source file explicitly documents that it does not run the F1 and E1 interfaces over real SCTP connections internally, but it does expose N2/N3 to the AMF/UPF over standard ports.
- `apps/cu/cu.cpp`, which implements a combined CU application (CU-CP + CU-UP) and exposes external F1, N2, and N3 interfaces over the standard UDP/SCTP ports.
- `apps/du/du.cpp`, which implements a DU application intended to be split from CU-CP and CU-UP and connected via the F1 interface.

The `apps/services/**` tree contains reusable application-level services (for example, metrics manager, remote control server, worker manager, resource usage reporting, buffer pools, and CLI command dispatching) that are used by these entry points.

### Configuration examples (`configs/`)

The `configs/` directory contains example YAML configuration files and documentation describing how configuration is layered and overridden. The configuration documentation indicates that applications support loading and overlaying multiple configuration files, and that command line parameters can override configuration file values.

### Containerized deployments and operations tooling (`docker/`)

The `docker/` folder provides multi-container deployment configurations intended to run OCUDU together with:

- A 5G core network (Open5GS) as part of a local end-to-end deployment.
- A “split” CU/DU deployment, where CU-CP, CU-UP, and DU run as separate containers instead of a single gNB container.
- A monitoring stack with Telegraf, InfluxDB, and Grafana.

These are orchestrated via multiple docker compose files that can be combined using Docker Compose’s multi-file merge/override mechanism.

### Automated documentation tooling (`docs/`)

The `docs/` directory contains Doxygen configuration and a `docs/docker-compose.yml` flow for generating API documentation in a container. The Doxygen CMake configuration defines multiple documentation targets, including full documentation and narrower targets for PHY, support libraries, DU-high (MAC/RLC), and FAPI.

### Testing (`tests/`)

The repository includes an extensive testing structure, with directories for unit tests, integration tests, benchmarks, and test doubles. The top-level build uses `BUILD_TESTING` to enable/disable compilation of these targets and uses GoogleTest for test discovery.

## Programming languages used

The repository is predominantly implemented in C++17, as indicated by the root `CMakeLists.txt` which sets `CMAKE_CXX_STANDARD 17` and the README badge that labels the code as C++17. In addition, the repository uses:

- CMake language (`CMakeLists.txt` and modules under `cmake/`) for build configuration and dependency discovery.
- YAML (for configuration examples under `configs/` and docker compose files under `docker/`).
- Shell scripting is present in supporting operational assets (for example within `docker/scripts/` and other helper areas), but the primary codebase is C++.

## Key technologies and frameworks

### Build system and packaging

The project is built with CMake and has an emphasis on configurable build options and optional dependencies. The build system also includes Software Bill of Materials (SBOM) generation via `external/cmake-sbom` and CMake integration in the top-level `CMakeLists.txt`.

### External libraries and optional integrations

From the top-level build options and dependency discovery logic, key technologies include:

- GoogleTest, used when `BUILD_TESTING` is enabled.
- yaml-cpp, required for building the project and used for configuration parsing/serialization.
- Optional ZeroMQ support (`ENABLE_ZEROMQ`) for some radio backend paths.
- Optional UHD support (`ENABLE_UHD`) for USRP hardware integration.
- Optional FFT libraries (FFTW, Intel MKL, AMD AOCL-FFTZ, ARM Performance Libraries) enabled via flags and discovered based on architecture.
- Optional DPDK support (`ENABLE_DPDK`) and related acceleration options, with conditional inclusion of `hal` components when DPDK is found.
- Optional backward-cpp support (`ENABLE_BACKWARD`) for improved stack traces/backtraces.
- Optional tracy profiler support (guarded by `ENABLE_OCUDU_TRACY`) via an external repo that must be present under `external/tracy/`.

The repository also vendors or includes third-party source under `external/`, including `fmt`, `uWebSockets`, and `CLI11.hpp`, which indicates use of the CLI11 argument parsing library.

### Containerized deployment and observability stack

The `docker/` directory defines:

- Open5GS-based 5G core container images and configuration.
- OCUDU node containers (gNB and split CU/DU deployments) built from the repository sources.
- A monitoring stack composed of Telegraf, InfluxDB 3, and Grafana.

The monitoring stack is designed to consume OCUDU metrics over a network-accessible endpoint; documentation in `docker/README.md` describes a WebSocket-based connection model for the monitoring stack to collect metrics.

### Documentation tooling

The repository generates API documentation using Doxygen. The Doxygen setup includes optional Graphviz `dot` integration and optional PlantUML support (by detecting `plantuml.jar`).

## Overall system functionality

OCUDU, as represented by this repository, implements a complete 5G CU/DU software stack with multiple deployment modes.

At runtime, the primary binaries orchestrate a combination of:

- Configuration ingestion and validation via CLI11 and YAML-based configuration parsing, including support for overlaying multiple configuration files.
- Initialization of logging, tracing, metrics, and application-level services such as worker/executor management and buffer pools.
- Instantiation of protocol stack components for control-plane and user-plane interfaces, including (depending on the chosen binary and configuration) N2/N3 (NGAP / GTP-U), F1-C/F1-U, E1, Xn, and E2 connectivity.
- Integration with radio and fronthaul implementations (for example SDR-based RU paths and Open Fronthaul support).
- Operational capabilities including packet capture generation, metrics export (stdout and JSON), and an optional remote control server for metrics/operations.

The repository’s docker compose configurations demonstrate an end-to-end functional environment where OCUDU can be run together with a local 5G core (Open5GS) and optionally instrumented using a monitoring stack (Telegraf/InfluxDB/Grafana). This reflects that the project is not only a library implementation, but also a deployable system intended for functional testing, development, and operational use.
