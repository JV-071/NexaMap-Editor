# Windows CI builds

The workflow builds and packages Release for x86 and x64 using the pinned
vcpkg baseline, Ninja and unity compilation. Files with conflicting private
helpers are compiled separately without changing editor behavior.

Dependency downloads and vcpkg binary packages are cached separately for each
triplet. Keys include the MSVC toolset and manifest hash. Prefix restoration
can seed a new manifest from older packages; vcpkg checks package ABIs before
reuse. Only successful configuration saves a new cache, preventing an incomplete
dependency install from occupying an immutable exact key. Build directories,
executables and test results are never restored from this dependency cache.

Tests run in parallel, with their existing individual timeouts and a 120-second
default. Assertions remain active in test executables even in Release. The
editor itself retains normal Release compilation options.

Every run uploads a `Windows-<arch>-diagnostics` artifact, including stage logs,
CTest JUnit results when tests run, CMake configuration records and vcpkg build
logs. The job summary reports stage exit codes and elapsed seconds. Environment
metadata records the commit, runner image, dependency baseline and cache hit.
Diagnostics expire after seven days; validated runtime packages after fourteen.

Compare successful runs on the same runner/toolchain with cold and warm caches
before attributing a speedup to a change. Cache restoration and upload also cost
time; the workflow deliberately retains compiler optimizations, packaging checks
and both architectures. A green workflow validates builds, automated tests and
package structure, but does not replace interactive editor testing.
