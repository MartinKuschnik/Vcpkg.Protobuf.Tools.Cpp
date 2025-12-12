# Vcpkg.Protobuf.Tools.Cpp 

[![build](https://github.com/MartinKuschnik/Vcpkg.Protobuf.Tools.Cpp/workflows/NuGet/badge.svg)](https://github.com/MartinKuschnik/Vcpkg.Protobuf.Tools.Cpp/actions) 
[![NuGet Status](http://img.shields.io/nuget/v/Vcpkg.Protobuf.Tools.Cpp.svg?style=flat)](https://www.nuget.org/packages/Vcpkg.Protobuf.Tools.Cpp/)

**Protocol Buffer compiler integration for native C++ projects using vcpkg.**

This NuGet package provides seamless integration of Protocol Buffer (`.proto`) compilation into Visual Studio C++ projects that use vcpkg for dependency management.

> **Note**: If you need gRPC support in addition to Protocol Buffers, consider using [Vcpkg.Grpc.Tools.Cpp](https://github.com/MartinKuschnik/Vcpkg.Grpc.Tools.Cpp) instead.

---

## Features

- **Automatic Compilation**: `.proto` files are automatically compiled during build
- **Incremental Builds**: Only changed `.proto` files are recompiled (saves build time!)
- **Per-File Configuration**: Configure compilation settings individually for each `.proto` file
- **Visual Studio Integration**: Full property page support in Visual Studio
- **vcpkg Integration**: Works seamlessly with vcpkg's manifest and classic mode
- **Smart Import Handling**: Flexible import path configuration per file
- **Zero Configuration**: Works out-of-the-box with sensible defaults

---

## Prerequisites

- **Visual Studio** (2015 or later) with C++ development tools
- **Windows** operating system
- **vcpkg** installed and integrated with Visual Studio
- **protobuf** package installed via vcpkg

---

## Quick Start

### 1. Install Required vcpkg Packages

#### Option A: Using vcpkg Manifest Mode (Recommended)

Add to your `vcpkg.json`:
```json
{
  "dependencies": [
    "protobuf"
  ]
}
```

#### Option B: Using vcpkg Classic Mode

```bash
vcpkg install protobuf:x64-windows
```

### 2. Install the NuGet Package

Add the NuGet package to your C++ project:

```powershell
Install-Package Vcpkg.Protobuf.Tools.Cpp
```

Or via Package Manager UI in Visual Studio.

### 3. Add .proto Files to Your Project

1. Add your `.proto` files to your project
2. After installing the NuGet package, all `.proto` files are automatically configured with the **ProtobufCompile** item type
3. **Important**: A restart of Visual Studio may be required for the new item type to be recognized

### 4. Build Your Project

When you build your project, the following files are automatically generated for each `.proto` file (e.g., `messages.proto`):
- `messages.pb.h` / `messages.pb.cc` - Protocol Buffer message definitions

Generated files are placed in a subdirectory of your project's intermediate directory (typically `x64\Debug\` or `x64\Release\`):
- `protobuf\` subdirectory - for `.pb.h` and `.pb.cc` files

---

## How It Works

### Build Process

1. **Validation**: Verifies that vcpkg is enabled and the required protobuf tools are installed
2. **File Selection**: Identifies all `.proto` files marked for compilation in your project
3. **Up-to-Date Check**: Compares timestamps between source `.proto` files and their generated outputs to determine which files need recompilation
4. **Compilation**: Executes the Protocol Buffer compiler (`protoc.exe`) for any outdated files
5. **Integration**: Automatically adds the generated `.cc` files to your project's compilation without cluttering the Solution Explorer

### Generated File Locations

Generated files are organized in your project's intermediate directory:

```
Intermediate Directory (e.g., x64\Debug\)
- protobuf\
  - messages.pb.h
  - messages.pb.cc
```

Include paths are automatically configured, so you can simply:
```cpp
#include "messages.pb.h"
```

---

## Example Usage

### Simple Message Definition

**protos/person.proto:**
```protobuf
syntax = "proto3";

package example;

message Person {
  string name = 1;
  int32 age = 2;
  string email = 3;
}
```

**main.cpp:**
```cpp
#include "person.pb.h"
#include <iostream>

int main() {
    example::Person person;
    person.set_name("John Doe");
    person.set_age(30);
    person.set_email("john@example.com");
    
    std::cout << "Name: " << person.name() << std::endl;
    std::cout << "Age: " << person.age() << std::endl;
    
    return 0;
}
```

### Working with Serialization

**protos/addressbook.proto:**
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

**main.cpp:**
```cpp
#include "addressbook.pb.h"
#include <fstream>
#include <iostream>

int main() {
    tutorial::AddressBook address_book;
    
    // Add a person
    tutorial::Person* person = address_book.add_people();
    person->set_name("John Doe");
    person->set_id(1234);
    person->set_email("john@example.com");
    
    // Serialize to file
    std::fstream output("addressbook.bin", std::ios::out | std::ios::binary);
    if (!address_book.SerializeToOstream(&output)) {
        std::cerr << "Failed to write address book." << std::endl;
        return -1;
    }
    output.close();
    
    // Deserialize from file
    tutorial::AddressBook read_address_book;
    std::fstream input("addressbook.bin", std::ios::in | std::ios::binary);
    if (!read_address_book.ParseFromIstream(&input)) {
        std::cerr << "Failed to parse address book." << std::endl;
        return -1;
    }
    input.close();
    
    // Read the data
    for (const auto& p : read_address_book.people()) {
        std::cout << "Person ID: " << p.id() << std::endl;
        std::cout << "  Name: " << p.name() << std::endl;
        std::cout << "  Email: " << p.email() << std::endl;
    }
    
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
1. Clean the project (Build -> Clean Solution)
2. Rebuild (Build -> Rebuild Solution)

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

## Known Issues

### IntelliSense Not Updating After Proto File Changes

When you modify a `.proto` file, the header file is regenerated correctly, but Visual Studio's IntelliSense database updates only every 60 minutes by default. This can cause IntelliSense to show outdated information.

**Solutions**:

1. **Keep the generated header file open**: Opening the `.pb.h` file in Visual Studio and keeping it open triggers immediate IntelliSense database updates whenever the file changes
2. **Manual rescan**: Right-click on your project in Solution Explorer ? **Rescan Solution** to manually update the IntelliSense database
3. **Adjust update interval**: Change the IntelliSense update frequency in Visual Studio settings:
   - Tools ? Options ? Text Editor ? C/C++ ? Advanced
   - Modify the "Database Refresh Interval" setting to a lower value

> **Tip**: If you're making frequent changes to `.proto` files, the most effective solution is to keep the generated header files open in Visual Studio. This ensures IntelliSense updates immediately after each build. Alternatively, you can reduce the IntelliSense update interval for a smoother development experience.

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
