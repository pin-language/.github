<div align="center">

# Pin

**A PHP-like language for AI coding agents — compiled to native code, with no garbage collector.**

[Language](https://github.com/pin-language/pin) · [Docs](https://github.com/pin-language/docs) · [AI guide](https://github.com/pin-language/docs/blob/HEAD/ai-guide.md) · [Benchmarks](https://github.com/pin-language/benchmarks)

</div>

Most code is now written by AI agents, and they already know PHP. Pin keeps that syntax but moves the
mistakes from production to compile time: every variable has one type, memory is managed by compile-time
ownership (like Rust, simplified), every path through a function is checked, and every error says how to fix
it — as text, or as JSON for the agent's loop. Programs compile through LLVM into one native binary,
including a multi-threaded HTTP server.

```php
function squares(int $n): list<int> {   // the caller owns the returned list
    list<int> $out = [];
    for ($i = 1; $i <= $n; $i++) { $out[] = $i * $i; }
    return $out;
}

function handle(string $method, string $path, string $body): string {
    if ($path == "/squares") { return json_encode(squares(10)); }
    http_status(404);
    return json_encode(["error" => "not found"]);
}

serve(8080, "handle");   // one binary, one thread per core, no locks needed
```

## Speed

Same algorithm in every language, same output checked. Apple M2, best of 3 runs (seconds, lower is better).

| | Pin | Rust | Go | PHP + JIT | PHP | Python |
|---|---|---|---|---|---|---|
| sieve — primes below 20M | **0.048** | 0.074 | 0.046 | 0.642 | 1.385 | 4.420 |
| collatz — longest chain below 1M | **0.163** | 0.133 | 0.174 | 0.715 | 3.223 | 6.370 |
| pi — 100M terms | **0.098** | 0.096 | 0.093 | 0.943 | 2.260 | 13.265 |
| list build + sum | **0.116** | 0.141 | 0.178 | 0.459 | 1.210 | 5.280 |

Rust- and Go-level speed, 4–13× faster than PHP with JIT. Small CPU loops, not applications; on macOS
differences under ~20% are noise. Method, memory and HTTP numbers: [benchmarks](https://github.com/pin-language/benchmarks).

## Try it with your agent

```bash
git clone https://github.com/pin-language/pin.git && cd pin && ./pin examples/hello.pn
```

Then tell your agent: *"Read https://raw.githubusercontent.com/pin-language/docs/HEAD/ai-guide.md, write
your program in Pin, and run `pin check --json` after every change."*

Needs Python 3.10+ and clang (macOS: `xcode-select --install`). Tested on macOS and Linux.

## Status

**v0.1 preview.** Functions, `int float bool string`, `list<T>`, ordered `map<K,V>`, ownership checking,
bounds-checked lists, a JSON HTTP server. Not yet: classes, `null`, `try`/`catch`, closures, modules,
`json_decode`, request headers. Not for production; do not expose the server to the internet yet.

[Open an issue](https://github.com/pin-language/pin/issues) — we especially want code your agent expected to
work but could not compile.

<sub>Made by PinaTo Co., Ltd. · MIT licensed</sub>
