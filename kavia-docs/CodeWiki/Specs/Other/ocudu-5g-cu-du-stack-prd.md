# OCUDU 5G CU/DU Stack — Product Requirements Document (PRD)

## Overview

OCUDU is a permissively licensed, open-source 5G (and beyond) Radio Access Network (RAN) software stack that provides a full L1/L2/L3 implementation and supports multiple deployment topologies, including a co-located gNB and split CU/DU architectures. The repository is a monolithic C++17, CMake-based codebase intended to produce deployable binaries and supporting tooling for building, configuring, testing, and running the stack in development and operational environments.

This PRD describes the product goals and requirements that are evidenced by the repository’s current capabilities, structure, and operational workflows. It is written to align engineering, operations, and stakeholders around what the repository delivers today and what “done” means for key product behaviors.

## Goals and non-goals

### Goals

OCUDU should provide a complete, standards-oriented 5G RAN software implementation that can be built and run in multiple node topologies and integrated with real or simulated radio and core-network environments. The product should emphasize:

Reliable execution of CU/DU functions and interfaces in supported deployment modes.
Operational readiness through observability, configuration management, and containerized deployment assets.
Developer productivity via an accessible build system, extensive automated testing, and auto-generated API documentation.

### Non-goals

OCUDU is not intended to provide an HTTP/REST API for external control or data-plane traffic. Repository-provided interfaces are primarily CLI applications, telecom protocol interfaces over SCTP/UDP/IP, and radio/fronthaul integration points.

OCUDU does not aim to be a hosted SaaS product. It is delivered as source code plus tooling, with optional container-based deployments.

## Target users and personas

### RAN developer

A developer working on PHY/MAC/RLC/PDCP/RRC and control/user plane interfaces who needs a modern C++ codebase with strong test coverage, benchmarks, and modular libraries.

### Systems integrator

An engineer integrating OCUDU with external components such as a 5G core, radio hardware (for example UHD/USRP), or Open Fronthaul RU, and who needs stable deployment modes and configuration patterns.

### Operator / lab engineer

A user running end-to-end deployments (for example with Open5GS) and needing containerized setups, metrics visualization, and operational controls (logs, metrics, remote control endpoints).

## Product scope

### In-scope capabilities (as implemented in this repository)

OCUDU includes library implementations and application entry points for a broad 5G NR stack, including major CU/DU components and associated interfaces. The repository structure indicates the following major areas:

Core libraries implemented under `include/ocudu/**` and `lib/**`, including CU-CP, CU-UP, DU, PHY, scheduler, MAC, RLC, PDCP, RRC, and key interfaces such as NGAP, F1AP, E1AP, XNAP, and E2/E2SM.
Multiple runnable applications under `apps/**`, including a co-located gNB app, a combined CU app, and a DU app.
Configuration examples under `configs/**` with support for layering multiple config files and command-line overrides.
Containerized deployment assets under `docker/**` supporting gNB + core, split CU/DU, and monitoring UI stacks.
Extensive automated tests under `tests/**` including unit tests, integration tests, benchmarks, and test doubles.
Automated API documentation tooling under `docs/**` (Doxygen-based), including multiple Doxygen targets.

### Out-of-scope capabilities

A general-purpose web control plane (HTTP API) is not part of the product surface in this repository.
A GUI for configuration and control is not part of the repository; the provided UI components focus on metrics visualization (Grafana stack) rather than node configuration management.

## Supported deployment modes

The repository indicates the product supports multiple primary runtime topologies via separate binaries and container stacks:

### Co-located gNB (single process)

The gNB application combines CU-CP, CU-UP, and DU in a single executable, while still exposing external N2/N3 interfaces to connect to AMF/UPF using standard networking.

### Split CU/DU architecture

The repository provides separate applications for a combined CU (CU-CP + CU-UP) and a DU to connect over the F1 interface. Docker Compose files also support a split deployment where `cu-cp`, `cu-up`, and `du` run as separate services.

### Containerized end-to-end environment with core network

Docker compose assets support running OCUDU alongside an Open5GS-based 5G core for local end-to-end scenarios.

### Observability stack (metrics visualization)

Docker compose assets provide a monitoring stack using Telegraf, InfluxDB, and Grafana. The monitoring stack connects to the gNB’s metrics via WebSocket and visualizes collected metrics.

## Functional requirements

### FR-1: Build and packaging

OCUDU must build as a C++17 project using CMake, producing multiple runnable binaries that correspond to supported deployment modes. The build system must support optional features and integrations via build-time flags and dependency discovery.

Acceptance criteria: A developer can configure and build the project using CMake, and the resulting build includes application targets under `apps/**` and library targets under `lib/**`, with optional components enabled/disabled based on build configuration.

### FR-2: CLI-driven node applications

OCUDU must provide CLI applications to run key node roles, including:
A co-located gNB application.
A CU application (CU-CP + CU-UP).
A DU application.

These applications must accept configuration via one or more configuration files and also via CLI overrides.

Acceptance criteria: Configuration examples and documentation demonstrate using multiple `-c` config file arguments and show that later files override earlier ones, and CLI overrides take precedence over file values.

### FR-3: Configuration layering and discoverability

OCUDU must support layered configuration, where multiple config files can be combined in-order, and CLI options override config-file values. OCUDU must also allow users to discover available configuration options via CLI help output and subcommand help.

Acceptance criteria: Documentation describes overlay semantics and shows example CLI help outputs with subcommands for major configuration areas (for example logging, PCAP, AMF, CU-CP, RU backends, cell configuration, and expert PHY).

### FR-4: Containerized deployment support

OCUDU must provide Docker Compose configurations for common deployments:
gNB + core network.
Split CU/DU deployment replacing the gNB.
Optional monitoring stack deployment.

Acceptance criteria: The repository contains compose files for these scenarios and documentation that explains how to combine multiple compose files using Docker Compose’s merge/override mechanism.

### FR-5: Observability and metrics integration

OCUDU must support exporting operational metrics in a form consumable by the provided monitoring stack. The monitoring stack must be able to connect to OCUDU via a configurable WebSocket endpoint to collect metrics.

Acceptance criteria: Documentation describes enabling metrics and remote control in node configuration and describes the `WS_URL` parameter for Telegraf connectivity in multiple scenarios (gNB in Docker, on host, Kubernetes, remote).

### FR-6: Documentation generation

OCUDU must support generating API documentation using Doxygen, including the ability to select different Doxygen targets.

Acceptance criteria: The repository includes `docs/docker-compose.yml` and documentation showing how to run the documentation container and select the target via `DOXYGEN_TARGET`.

### FR-7: Testing and benchmarks

OCUDU must include automated tests across multiple levels:
Unit tests for core libraries.
Integration tests for key system interactions.
Benchmarks for performance-sensitive areas.
Test doubles to support isolation and controlled test environments.

Acceptance criteria: The repository includes distinct `tests/unittests`, `tests/integrationtests`, `tests/benchmarks`, and `tests/test_doubles` directories and integrates with GoogleTest when testing is enabled.

## Non-functional requirements

### NFR-1: Performance orientation

The system must be suitable for high-performance RAN workloads, including support for SIMD/vectorized implementations and optional hardware acceleration integrations.

Acceptance criteria: The repository includes vectorization and HAL components and build options for acceleration-related dependencies (for example DPDK and optional FFT backends).

### NFR-2: Reliability and operability

The system must support operational workflows through logging, tracing, metrics reporting, and the ability to run in containerized environments.

Acceptance criteria: Repository contains structured support code for logging/metrics/tracing and Docker Compose scenarios that expose observability endpoints and include troubleshooting guidance for connectivity.

### NFR-3: Modularity and maintainability

The codebase should remain modular at the library level, with clear public headers under `include/ocudu/**` and implementations under `lib/**`, and runnable entry points under `apps/**`.

Acceptance criteria: The repository layout consistently separates headers, library implementations, and applications, and CMake files reflect this structure.

### NFR-4: Documentation and developer enablement

Developers should be able to generate local API documentation and find repository-local analysis documentation.

Acceptance criteria: `docs/README.md` documents the documentation structure and points to maintained wiki-style pages (for example the repository analysis).

## Dependencies and external integrations

OCUDU integrates with and/or optionally depends on the following categories of external software, as evidenced by repository documentation and build/deployment tooling:

A 5G core network environment for end-to-end scenarios (Open5GS in the provided Docker stack).
Radio backends including UHD (USRP) and ZeroMQ paths (optional, build-config dependent).
Optional acceleration and performance libraries, including DPDK and optional FFT libraries (architecture- and flag-dependent).
A monitoring stack using Telegraf, InfluxDB, and Grafana, configured to connect to OCUDU via WebSocket for metric ingestion.

## User experience and workflows

### Build and run workflow (developer / lab)

A typical workflow should be:
Build OCUDU via CMake using local toolchain and selected options.
Run a node application using CLI with one or more configuration files.
Inspect logs and observe metrics, either via stdout/JSON output or via the monitoring stack.
Iterate configuration using layered config files and CLI overrides, validating changes via CLI help and test suites.

### Containerized workflow (operator / integrator)

A typical workflow should be:
Start a baseline deployment using `docker compose` with `docker/docker-compose.yml`.
Switch to split deployment by adding `docker/docker-compose.split.yml`.
Enable observability by adding `docker/docker-compose.ui.yml`, ensuring node configuration enables metrics and remote control.
Configure the metrics collector using environment variables (notably `WS_URL`) depending on where the node is running.

## Acceptance criteria (release-level)

A release of the repository that meets this PRD should satisfy the following:

The repository builds as a C++17 CMake project and produces runnable CU/DU-related binaries under `apps/**`.
Node applications support configuration by multiple config files and CLI overrides, and users can discover options using `-h` and subcommand help.
Docker Compose assets support (1) gNB + core, (2) split CU/DU, and (3) monitoring stack, with documentation describing how to combine deployments.
Metrics ingestion via WebSocket works with the provided Telegraf/InfluxDB/Grafana stack when the recommended config options are enabled.
Doxygen documentation can be generated via the documented `docs/docker-compose.yml` flow, with selectable targets.
The repository contains and builds a substantial automated test suite, including unit and integration tests, plus benchmarks.

## Risks and constraints

OCUDU is a systems-level telecom stack with many optional integrations. As a result:
Build and runtime behavior can vary materially based on enabled features, target architecture, and installed dependencies.
End-to-end deployments depend on correct network configuration and interoperability with external components (for example Open5GS and any radio backend).
Metrics visibility depends on node configuration enabling metrics export and remote control, and on correct WebSocket connectivity between collectors and the node.

## References

This PRD is grounded in the following repository sources:

The root `README.md` for project positioning and scope.
`docs/repository-analysis.md` for repository layout and component overview.
`docker/README.md` for supported container deployments and metrics UI connectivity model.
`configs/readme.md` for configuration layering and CLI help patterns.
`docs/README.md` for documentation generation workflows.
The repository layout under `apps/`, `include/`, `lib/`, `tests/`, and `docker/` for evidencing implemented capabilities.
