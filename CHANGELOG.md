# Changelog

All notable changes to the **app-images** container recipes are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
Image releases are tagged `<app>-v<version>` (e.g. `openrefine-v3.10.1`).

## [Unreleased]

### Added
- **`openrefine`** (`ghcr.io/spore-host/openrefine`) — OpenRefine 3.10.1 built
  from the official upstream release on `eclipse-temurin:21-jre`, bound to
  `0.0.0.0:3333` and auth-less (gated by spore-host/spawn's `:443` proxy token).
  The first app image, and the worked example for the "bring your own app to
  spawn" tutorial in the README (spore-host/spawn#590).
