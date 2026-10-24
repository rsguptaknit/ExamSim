# Windows Build and Run

Status: NOT VERIFIED. The current project has the React Native CLI Windows/Android-compatible base, but native Windows integration is not yet completed.

The Windows target requires the React Native Windows community platform package, supported Visual Studio and Windows SDK workloads, New Architecture/Fabric, and Win32 with Windows App SDK. At scaffold time the npm registry reported React Native Windows 0.84.0 with a peer requirement of React Native 0.84.1, while this CLI scaffold uses React Native 0.88.0. Do not add the incompatible package; select a compatible aligned version set before Windows integration. Validate ZIP handling, video, filesystem access, and secure storage before selecting production packages.

The M1 distribution criterion is a portable folder, not a single-file executable. Bundle `msvcp140.dll`, `msvcp140_1.dll`, `msvcp140_2.dll`, `vcruntime140.dll`, and `vcruntime140_1.dll`, then launch on a clean Windows machine and reach Welcome before reporting PASS. These tasks remain outstanding.