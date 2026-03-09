<!--
SPDX-FileCopyrightText: Copyright (C) 2021-2026 Software Radio Systems Limited
SPDX-License-Identifier: BSD-3-Clause-Open-MPI
-->

# OCUDU Documentation

This directory contains the automated API documentation generation (Doxygen) for OCUDU code, as well as repository-local wiki-style documentation pages.

## Structure

```txt
docs/
├── .env                     # env file for docker-compose
├── docker-compose.yml       # Docker services for documentation
├── doxygen/                 # Doxygen project
├── repository-analysis.md   # Wiki-style repository overview (this repo)
└── README.md                # This file
```

## Wiki pages

The following pages are maintained in this repository:

- [Repository Analysis](repository-analysis.md)

## Docker Services

### Usage

Builds doxygen locally with

```bash
docker compose -f docs/docker-compose.yml up
```

You can select another doxygen target using the environmental variable

```bash
DOXYGEN_TARGET=doxygen-support docker compose -f docs/docker-compose.yml up
```

### Environment Variables

To run the docker-compose, you may need to adjust the variables defined in the .env file.

- `UID`/`GID`: Your user/group IDs for file permissions in Docker
