# Bekos.ProcessMemory

**Bekos.ProcessMemory** is a C# library for reading and writing the memory of Windows processes.
It provides a typed API for common memory operations — including pointer-chain resolution, value subscriptions, and memory freezing.

> ⚠️ **Notice:** This library uses Windows kernel APIs (`ReadProcessMemory`, `WriteProcessMemory`) and requires Administrator privileges. It is intended for legitimate use cases such as game modding, debugging tools, and process introspection. The author is not responsible for any misuse.

## Features

- **Generic read/write** — Read and write typed values (`bool`, `byte`, `int`, `long`, `float`, `double`, `string`, custom structs) directly from process memory.
- **String address parsing** — Resolve addresses from human-readable strings: plain hex, `module.exe+offset`, and full pointer chains (`module.exe+base,offset1,offset2`).
- **Subscribe** — Poll a memory address at a set interval and invoke a callback whenever it is read, enabling reactive memory monitoring.
- **Freeze** — Continuously overwrite a memory address at a set interval to keep its value locked.
- **Async connection waiting** — Wait for a process to start before connecting, without blocking the calling thread.
- **32-bit and 64-bit support** — Automatically detects whether the target process is 64-bit and adjusts pointer reads accordingly.

## Details

- Written in **C#**.
- Windows only — uses `kernel32.dll` via P/Invoke (`ReadProcessMemory`, `WriteProcessMemory`, `CreateToolhelp32Snapshot`).
- Requires **Administrator privileges** at runtime; throws `UnauthorizedAccessException` otherwise.
- No external NuGet dependencies.

## Disclaimer

This library is intended for educational, modding, and debugging purposes only. Use it only on processes you own or have explicit permission to inspect. The author is not responsible for any damages or misuse arising from this software.

## Usage

### Connecting to a process

```csharp
// By process name
var mem = new MemoryManipulator("game.exe");

// By PID
var mem = new MemoryManipulator(1234);

// Wait for the process to start (polls every 500 ms)
await mem.WaitForConnectAsync(500);
```

### Reading memory

```csharp
// Read an int at a static address (hex)
int health = mem.Read<int>("0x12AB34CD");

// Read via module + offset
float speed = mem.Read<float>("game.exe+0x1A2B3C", isOffsetsHexdecimal: true);

// Read via pointer chain: dereference base+0x10, then add 0x20, then 0x3C
int ammo = mem.Read<bool>("game.exe+0x10,0x20,0x3C", isOffsetsHexdecimal: true);

// Read a UTF-8 string (up to 32 bytes, null-terminated)
string name = mem.ReadString("0x00AABBCC", bytesLength: 32);

// Resolve a pointer, then read a field at an offset from it
UIntPtr module = mem.GetModuleAddressByName("engine.dll");
UIntPtr entity = mem.ReadPointer(module + 0x1A2B3C);
int health = mem.Read<int>(entity + 0x100);
```

### Writing memory

```csharp
mem.Write<int>("game.exe+0x1A2B3C", 100, isOffsetsHexdecimal: true);
mem.Write<float>("0x12AB34CD", 9999.0f);
```

### Subscribing to memory changes

```csharp
// Invoke callback every 100 ms with the current value
mem.Subscribe<int>("game.exe+0x1A2B3C", value =>
{
    Console.WriteLine($"Health: {value}");
}, 
interval: 100, isOffsetsHexdecimal: true);

// Stop polling
mem.Unsubscribe("game.exe+0x1A2B3C");
```

### Freezing a value

```csharp
// Lock health to 100, rewriting every 50 ms
mem.Freeze<int>("game.exe+0x1A2B3C", 100, interval: 50, isOffsetsHexdecimal: true);

// Unfreeze
mem.Unfreeze("game.exe+0x1A2B3C");
```

### Cleanup

```csharp
mem.Dispose();
```
