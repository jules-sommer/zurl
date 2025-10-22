# zig-curl

[![supports Zig v0.16.0-dev](https://img.shields.io/badge/supports-Zig_v0.16.0_dev-yellow.svg)](https://ziglang.org)

<p align="center"><img src="docs/logo.svg" width="35%"/></p>

Zig bindings for [libcurl](https://curl.haxx.se/libcurl/), forked from [jiacai2050/zig-curl](https://github.com/jiacai2050/zig-curl) and updated for Zig 0.16.0-dev with additional changes to the API.

The vendored libraries consist of:

| Library | Version |
|---------|---------|
| libcurl | [8.16.0](https://github.com/curl/curl/tree/curl-8_16_0) |
| zlib    | [1.3.1](https://github.com/madler/zlib/tree/v1.3.1) |
| mbedtls | [3.6.0](https://github.com/Mbed-TLS/mbedtls/tree/v3.6.0) |

## Usage

Example program:
```zig
const std = @import("std");
const curl = @import("curl");

const URL = "https://edgebin.liujiacai.net/anything";

pub fn main() !void {
    var gpa: std.heap.DebugAllocator(.{}) = .init;
    defer if (gpa.deinit() != .ok) @panic("leak");
    const allocator = gpa.allocator();

    const ca_bundle = try curl.allocCABundle(allocator);
    defer ca_bundle.deinit();
    const easy = try curl.Easy.init(.{
        .ca_bundle = ca_bundle,
    });
    defer easy.deinit();

    {
        std.debug.print("GET without body\n", .{});
        const resp = try easy.fetch(URL, .{});
        std.debug.print("Status code: {d}\n", .{resp.status_code});
    }

    {
        std.debug.print("\nGET with fixed buffer as body\n", .{});
        var buffer: [1024]u8 = undefined;
        var writer = std.Io.Writer.fixed(&buffer);
        const resp = try easy.fetch(URL, .{ .writer = &writer });
        std.debug.print("Status code: {d}\nBody: {s}\n", .{ resp.status_code, writer.buffered() });
    }
}
```

Check the `examples/` directory for more examples.

## Installation

Fetch the dependency into your project:

```sh
zig fetch --save=zurl git+https://github.com/jules-sommer/zurl
```

Import the fetched dependency in your build.zig:

```zig
const zurl_dep = b.dependency("zurl", .{});

const exe_mod = b.createModule(.{
    .root_source_file = b.path("src/main.zig"),
    .target = target,
    .optimize = optimize,
    .imports = &[_]std.Build.Module.Import{
        .{
            .name = "zurl",
            .module = zurl.module("zurl"),
        },
    },
});

```

By default this library links to the vendored libcurl. To disable vendoring and link against the system libcurl instead:

```zig
const zurl_dep = b.dependency("zurl", .{ .link_vendor = false });

const exe_mod = b.createModule(.{
    // ... other fields omitted for brevity ...
    .link_libc = true,
});

exe_mod.linkSystemLibrary("curl", .{});
```

## License

[MIT](LICENSE)
