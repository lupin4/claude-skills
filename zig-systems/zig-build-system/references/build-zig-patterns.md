# build.zig API Patterns

## Key Functions

| Function | Purpose |
|----------|---------|
| `addStaticLibrary` | Create Zig static library |
| `addSharedLibrary` | Create Zig shared library |
| `addExecutable` | Create executable |
| `addTest` | Create test executable |
| `addSystemCommand` | Run shell command as build step |
| `addRunArtifact` | Run a built artifact |
| `linkSystemLibrary` | Link a system/local library |
| `addLibraryPath` | Add library search path |
| `addIncludePath` | Add include search path |
| `installArtifact` | Install to zig-out/ |

## Step Dependencies

```zig
// B depends on A completing first
step_b.step.dependOn(&step_a.step);

// Named step for `zig build step-name`
const my_step = b.step("name", "description");
my_step.dependOn(&some_step.step);
```
