# JSim Modernization Status

Last verified: 2026-08-24

## Preservation constraints

Scientific reproducibility is the primary constraint. Modernization must not intentionally change numerical behavior, model semantics, solver behavior, units, project/model file formats, or historical reference outputs. Numerical libraries must not be replaced without compatibility evidence and regression baselines.

## Repository baseline

- Controlling build system: Bash orchestration in `SRC2.0/build/` plus GNU Make for native code. The documented full entry point is `jsbuild.all`.
- Current branch: `master`, tracking the community fork's `origin/master`; upstream is `NSR-Physiome/JSim`.
- Preserved release tags: `v2.18`, `v2.19`, and `v2.20`.
- `git shortlog -sne --all` reports 120 commits under `NSR-Physiome <barthj@uw.edu>` and 4 under the NSR-Physiome GitHub noreply identity. Existing copyright and attribution identify the University of Washington and National Simulation Resource.
- License: `SRC2.0/license.html` is a three-clause BSD-style license with additional third-party license conditions. It must remain with source and binary distributions.

## Current build status

The documented build is not complete on this host. Parser generation and Java compilation succeed with JDK 11. The original earliest actionable failure was JNI header generation because JDK 11 no longer ships `javah`.

`SRC2.0/build/jsbuild.jh` now preserves the historical `javah` path when available and falls back to `javac -h` on modern JDKs. The focused header stage generates all 11 expected nonempty JNI headers. The dependent native `math` target then compiles `natmath.c` successfully and reaches the linker.

The next blocker is the macOS native linker configuration: `SRC2.0/home/lib/Makefile.global.macos` hard-codes `/usr/local/gfortran`, `/usr/local/gfortran/lib/libgfortran.dylib`, and an `x86_64-apple-darwin14` GCC runtime path. The installed native compiler and runtime are under `/opt/homebrew` for `arm64`.

The historical scripts do not consistently propagate failures. `jsbuild.native` exits zero after its loop even when an inner Make invocation fails, and `jsbuild.all` continues into the Fortran package after earlier stage failures. Build completion therefore cannot currently be inferred from the top-level exit status alone.

## Verified environment and prerequisites

Host used for this record:

- macOS 15.7.9 (24G830), Darwin 24.6.0, Apple Silicon `arm64`.
- zsh 5.9; documented instructions assume a Bash shell and the scripts use `#!/bin/bash`.
- GNU Make 3.81.
- Apple Clang 17.0.0; `/usr/bin/gcc` resolves to Apple Clang.
- Homebrew GNU Fortran 15.2.0 at `/opt/homebrew/bin/gfortran`.
- Amazon Corretto OpenJDK/JDK 11.0.26 at `/Library/Java/JavaVirtualMachines/amazon-corretto-11.jdk/Contents/Home`.
- Bison 2.3 and Apple Flex 2.6.4 are installed, but parser generation uses the bundled `JFlex.jar` and CUP jars.
- Ant is not installed and is not used by the controlling build path.

The upstream documentation targets macOS High Sierra, Oracle/OpenJDK 8, Xcode command-line tools, GCC/gfortran (historically version 4 or 6), and a matching `jsimauxlib`. JDK 8 remains the documented baseline; this session only verifies that parser generation, Java compilation, RMI stub generation, and modern JNI header generation run under JDK 11.

Required native and external components include C/C++, GNU Fortran and its runtime, libSBML plus its matching Java jar, libxml2, Antimony/libNOM, and the preserved solver sources (including Radau, Dopri5, CVODE, and Toms731). Bundled client jars include JFlex/CUP, Xerces, JAMA, Colt, SGT, SBW, and related historical dependencies.

The bundled `jsimauxlib/macos` native libraries are not Apple Silicon compatible: libSBML, libSBML Java JNI, Antimony/libNOM, libgfortran, and libquadmath are `x86_64`; libxml2 contains only `i386` and `x86_64`. Matching `arm64` or carefully verified universal builds are required before a native Apple Silicon runtime can be validated.

## Commands attempted

Environment inspection included `sw_vers`, `uname -a`, `arch`, `/usr/libexec/java_home`, `java -version`, `javac -version`, compiler version checks, and binary inspection with `file`.

The documented build was attempted without source edits using:

```bash
cd /Users/collins/Projects/JSim
env JSIMSRC="$PWD/SRC2.0" \
  JSIMSDK="$(/usr/libexec/java_home)" \
  JSIMAUXLIB="$PWD/jsimauxlib" \
  PATH="$PWD/SRC2.0/build:$PATH" \
  "$PWD/SRC2.0/build/jsbuild.all"
```

After the compatibility change, the narrow validations were:

```bash
env JSIMSRC="$PWD/SRC2.0" JSIMSDK="$(/usr/libexec/java_home)" \
  JSIMAUXLIB="$PWD/jsimauxlib" PATH="$PWD/SRC2.0/build:$PATH" \
  "$PWD/SRC2.0/build/jsbuild.jh"

env JSIMSRC="$PWD/SRC2.0" JSIMSDK="$(/usr/libexec/java_home)" \
  JSIMAUXLIB="$PWD/jsimauxlib" PATH="$PWD/SRC2.0/build:$PATH" \
  "$PWD/SRC2.0/build/jsbuild.native" math
```

The first command passed and produced 11 headers. The second proved the missing-header failure is resolved, compiled the C object, and stopped at the hard-coded legacy Fortran runtime paths.

## Generated files

The build writes `.class` files directly under `SRC2.0/JSim`, regenerates parser/scanner Java sources from `.cup` and `.lex` inputs, creates JNI headers below `SRC2.0/JSim/nonJava`, and creates native objects/libraries under the source tree and `SRC2.0/native/macos`. Parser generators embed timestamps, so regeneration can create nonfunctional diffs. The repository currently ignores only `CVS/`; build-output isolation or precise ignore rules are still needed for reproducible clean builds.

## Known blockers

- Replace hard-coded macOS compiler and Fortran runtime locations with detected or explicitly supplied toolchain values, while preserving compilation and link flags for a reference Intel build.
- Produce or recover license-compatible `arm64` builds of the exact external library versions expected by JSim, especially libSBML 5.14.0 with its matching `libsbmlj.jar`, Antimony/libNOM, and libxml2.
- Remove the unconditional `JSIMARCH=amd64` assignment in `jsbuild.osarch` only after architecture-specific output and auxiliary-library selection are defined and tested.
- Make build stages fail fast and isolate generated products so a successful command status corresponds to a complete artifact.
- Establish an Intel reference build and verification baseline before accepting native Apple Silicon numerical results.

## Numerical-compatibility risks

- Changing gfortran generations, optimization flags, BLAS/LAPACK implementations, floating-point contraction, or architecture can alter rounding, convergence, and solver trajectories.
- Rebuilding CVODE, Radau, Dopri5, Toms731, FPACK, or other native numerical code with a modern compiler may expose undefined behavior or produce platform-dependent results.
- Replacing libSBML, Antimony/libNOM, XML libraries, or their Java bindings can change import/export semantics and serialized model content.
- Existing verification references are architecture-sensitive; the upstream documentation already describes macOS comparisons as less exact than Linux comparisons.
- The working tree contained pre-existing changes to PDE verification project/reference files before this session. They were not modified or used as a new baseline.

## Roadmap

The immediate next step is milestone 1: make the existing macOS toolchain locations configurable, then prove that the smallest JNI library links without changing compiler optimization or numerical source.

### Milestone 1: Reproducible native toolchain

- [ ] Add explicit overrides for the C compiler, Fortran compiler, and Fortran runtime paths while preserving historical defaults where they still exist.
- [ ] Detect `arm64` and `x86_64` without changing architecture-specific auxiliary-library selection until both layouts are defined.
- [ ] Make `jsbuild.native math` return a nonzero status when compilation or linking fails.
- [ ] Record compiler paths, versions, flags, target architecture, and linked-library identities in a build manifest.
- [ ] Build and load the smallest JNI library on native Apple Silicon and an Intel reference environment.

Completion evidence: repeatable `math` JNI builds from clean trees on both architectures, with archived commands and binary dependency reports.

### Milestone 2: Exact auxiliary libraries

- [ ] Inventory source versions, patches, configurations, checksums, and licenses for every bundled native dependency.
- [ ] Build exact-version `arm64` or universal libSBML 5.14.0, its matching Java JNI binding, libxml2, Antimony/libNOM, and required Fortran runtimes.
- [ ] Keep rebuilt auxiliary artifacts separate from historical binaries and identify them by architecture and checksum.
- [ ] Validate library loading and focused SBML/Antimony import-export round trips without changing fixtures or formats.
- [ ] Document any dependency that cannot be reproduced exactly before considering a version change.

Completion evidence: provenance-complete auxiliary artifacts that load independently on both architectures and preserve focused serialization fixtures.

### Milestone 3: Scientific compatibility baseline

- [ ] Make all build stages fail fast and isolate or precisely ignore generated outputs.
- [ ] Produce clean Intel and Apple Silicon builds from documented environments.
- [ ] Run the existing verification suite without replacing historical references.
- [ ] Classify differences as exact, platform-rounding, compiler-sensitive, or semantic and define explicit reviewable tolerances.
- [ ] Archive toolchain manifests, logs, binary checksums, and verification reports for each accepted build.

Completion evidence: independently repeatable builds and reviewed cross-architecture verification results, with no unexplained model, unit, solver, or file-format differences.
