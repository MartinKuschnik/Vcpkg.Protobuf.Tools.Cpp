# Vcpkg.Protobuf.Tools.Cpp [![build](https://github.com/MartinKuschnik/Vcpkg.Protobuf.Tools.Cpp/workflows/NuGet/badge.svg)](https://github.com/MartinKuschnik/Vcpkg.Protobuf.Tools.Cpp/actions) [![NuGet Status](http://img.shields.io/nuget/v/Vcpkg.Protobuf.Tools.Cpp.svg?style=flat)](https://www.nuget.org/packages/Vcpkg.Protobuf.Tools.Cpp/)

**MSBuild integration of Protocol Buffers for C++ projects using vcpkg.**

---

## What is this?

`Vcpkg.Protobuf.Tools.Cpp` is a NuGet package that seamlessly integrates Protocol Buffers compilation into your C++ project's build process. It automatically compiles `.proto` files into C++ source and header files during build, with full support for incremental builds.

**Key Benefits:**
- ? Automatic compilation of `.proto` files
- ? Incremental build support (only changed files are recompiled)
- ? Works with vcpkg classic and manifest mode
- ? Visual Studio project system integration
- ? Zero manual configuration needed

---

## Prerequisites

Before using this package, you need:

1. **vcpkg** - [Installation Guide](https://github.com/microsoft/vcpkg#getting-started)
2. **protobuf library** - Install via vcpkg (see below)

### Installing protobuf via vcpkg

**Option A: Classic Mode**
```bash
vcpkg install protobuf:x64-windows
vcpkg integrate install
```

**Option B: Manifest Mode (vcpkg.json)**
```json
{
  "dependencies": [
    "protobuf"
  ]
}
```

Enable manifest mode in your project:
```xml
<PropertyGroup>
  <VcpkgEnableManifest>true</VcpkgEnableManifest>
</PropertyGroup>
```

---

## Installation

Install the `Vcpkg.Protobuf.Tools.Cpp` NuGet package into your C++ project.

**Via NuGet Package Manager Console:**
```powershell
Install-Package Vcpkg.Protobuf.Tools.Cpp
```

**Via Visual Studio UI:**
1. Right-click your project ? **Manage NuGet Packages**
2. Search for `Vcpkg.Protobuf.Tools.Cpp`
3. Click **Install**

**Via .NET CLI:**
```bash
dotnet add package Vcpkg.Protobuf.Tools.Cpp
```

---

## Usage

### 1. Add Proto Files to Your Project

Simply add `.proto` files to your C++ project. Visual Studio will automatically recognize them.

**Example: `person.proto`**
```protobuf
syntax = "proto3";

package tutorial;

message Person {
  string name = 1;
  int32 id = 2;
  string email = 3;
}

message AddressBook {
  repeated Person people = 1;
}
```

### 2. Build Your Project

The `.proto` files are automatically compiled during build. Generated files are placed in:
```
$(IntDir)protobuf\
```

For example:
- Debug: `x64\Debug\protobuf\person.pb.h` and `person.pb.cc`
- Release: `x64\Release\protobuf\person.pb.h` and `person.pb.cc`

### 3. Use Generated Code

Include the generated headers in your C++ code:

```cpp
#include "person.pb.h"
#include <iostream>
#include <fstream>

int main() {
    // Create a Person message
    tutorial::Person person;
    person.set_name("John Doe");
    person.set_id(1234);
    person.set_email("john@example.com");

    // Serialize to file
    std::fstream output("person.bin", std::ios::out | std::ios::binary);
    if (!person.SerializeToOstream(&output)) {
        std::cerr << "Failed to write person." << std::endl;
        return -1;
    }
    output.close();

    // Deserialize from file
    tutorial::Person read_person;
    std::fstream input("person.bin", std::ios::in | std::ios::binary);
    if (!read_person.ParseFromIstream(&input)) {
        std::cerr << "Failed to parse person." << std::endl;
        return -1;
    }
    input.close();

    std::cout << "Name: " << read_person.name() << std::endl;
    std::cout << "ID: " << read_person.id() << std::endl;
    std::cout << "Email: " << read_person.email() << std::endl;

    return 0;
}
```

---

## Troubleshooting

### Error: "Vcpkg not enabled"

**Solution**: Enable vcpkg integration in Visual Studio:
```bash
vcpkg integrate install
```

### Error: "protobuf not installed"

**Solution**: Install the required package via vcpkg:
```bash
vcpkg install protobuf:x64-windows
```

Or add it to your `vcpkg.json` if using manifest mode.

### Generated Files Not Found

**Solution**: 
1. Check that the build succeeded without errors
2. Look in your intermediate directory subdirectory `protobuf\` (typically found in `x64\Debug\protobuf\` or `x64\Release\protobuf\`)
3. Rebuild the project to regenerate files

### Proto File Changes Not Detected

This package uses **CustomBuild** items with MSBuild's incremental build tracking. Changes should be detected automatically. If not:
1. Clean the project (Build ? Clean Solution)
2. Rebuild (Build ? Rebuild Solution)

### Include Errors in Generated Files

If you see errors like `cannot open include file: 'google/protobuf/...'`:

**Solution**: Make sure the protobuf include directories are in your project's additional include directories. This is usually handled automatically by vcpkg integration.

---

## Incremental Build Support

**Key Feature**: Only changed `.proto` files are recompiled!

The package uses MSBuild's `CustomBuild` item type with input/output tracking:
- Compares timestamps of `.proto` files and generated outputs
- Skips compilation if files are up-to-date
- Significantly reduces build times in large projects
- Detects changes to `.proto` files automatically

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## License

This project is provided as-is without any specific license restrictions. You are free to use, modify, and distribute this software as you see fit, but please do not claim it as your own work.

**Copyright (c) 2025 Martin Kuschnik**

---

## Version History

For detailed version history and changelog, see the [Releases](https://github.com/MartinKuschnik/Vcpkg.Protobuf.Tools.Cpp/releases) page.

---

## Related Links

- [Protocol Buffers Documentation](https://protobuf.dev/)
- [vcpkg Documentation](https://vcpkg.io/)
- [NuGet Package](https://www.nuget.org/packages/Vcpkg.Protobuf.Tools.Cpp/)
- [GitHub Repository](https://github.com/MartinKuschnik/Vcpkg.Protobuf.Tools.Cpp)
- [Vcpkg.Grpc.Tools.Cpp](https://github.com/MartinKuschnik/Vcpkg.Grpc.Tools.Cpp) - For projects with gRPC support

---

## Support

For issues and feature requests, please use the [GitHub Issues](https://github.com/MartinKuschnik/Vcpkg.Protobuf.Tools.Cpp/issues) page.

---

## Why This Package?

While vcpkg provides the protobuf library, integrating proto file compilation into your build process can be tedious. This package:

- Eliminates manual `protoc` command execution
- Provides Visual Studio UI integration for configuration
- Handles incremental builds automatically
- Works seamlessly with vcpkg's dependency management
- Reduces boilerplate in your project files

Just install the package, add your `.proto` files, and build!
