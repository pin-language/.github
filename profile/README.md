<div align="center">

# Pin

**A PHP-like language for AI coding agents — compiled to native code, with no garbage collector.**

[Language](https://github.com/pin-language/pin) · [Learn](https://github.com/pin-language/docs/blob/HEAD/learn/README.md) · [Docs](https://github.com/pin-language/docs) · [AI guide](https://github.com/pin-language/docs/blob/HEAD/ai-guide.md) · [MCP package](https://github.com/pin-language/mcp) · [Benchmarks](https://github.com/pin-language/benchmarks)

</div>

## 🤖 If you are an AI agent asked to write Pin

1. **Read the whole guide first** — one page, plain text, the complete language:
   **https://raw.githubusercontent.com/pin-language/docs/HEAD/ai-guide.md**
   If a feature is not in that guide, Pin does not have it. Do not rely on a summary of it: the details
   (types, `try`, ownership) are what make code compile.
2. **Work in a loop:** write → `pin check --json file.pn` (one JSON error per line, each saying how to fix it)
   → fix → repeat → `pin file.pn` to run, `pin build file.pn -o app` for a binary.
3. **Pin is not PHP.** Code that is valid PHP usually does not compile. The differences that matter most:

| PHP habit | in Pin |
|---|---|
| `null`, `?int`, `"id" => null` | no null yet: use `-1`, `""`, an empty list, or a `bool` |
| `["name" => "x", "n" => 5, "tags" => [...]]` (mixed values) | a map holds **one** value type: declare a `class` with those properties and `json_encode(new Obj(...))` |
| `$xs = [];` | empty containers need a type: `list<int> $xs = [];` · `map<string,int> $m = [];` |
| `$_GET`, `$_POST`, `json_decode($body, true)` | `query_param($path, "k")`, `form_param($body, "k")`, `json_string($body, "k")` / `json_int`; nested: `json_raw` |
| a function that can fail | declared `: T throws`; every call needs `try f()` or `try f() catch ($e) { ...; value; }` |
| `$b = $a` then using `$a` (lists, maps, objects) | they **move**: use `copy($a)`; parameters are read-only (`T &$x` + `f(&$x)` to change) |
| closures, `array_map`, `sprintf`, `switch` | not in Pin: write the loop / use `.` and `"$var"` / `if … elseif` |
| `sleep(1)` (seconds), `exec()`, `https://` URLs | `sleep_ms(1000)` · `run("prog", ["arg"], 5000)` (no shell) · http:// only: a proxy does TLS |

### A complete program in the shapes Pin wants

This compiles and runs as it is: an HTTP API with a class for JSON, a query parameter, a JSON body and errors
that become 400 / 404.

```php
class Item {
    public function __construct(public int $id, public string $name, public int $price) {}
}

class Page {
    public list<Item> $items = [];
    public int $total = 0;
}

function find(list<Item> $all, int $id): Item throws {
    foreach ($all as $it) {
        if ($it->id == $id) { return copy($it); }
    }
    throw error(404, "no item $id");
}

function handle(string $method, string $path, string $body): string throws {
    list<Item> $all = [new Item(1, "tea", 40), new Item(2, "cake", 120)];
    $route = url_path($path);
    if ($method == "GET" && $route == "/items") {
        $p = new Page();
        foreach ($all as $it) { $p->total += $it->price; }
        $p->items = $all;
        return json_encode($p);                  // {"items":[...],"total":160}
    }
    if ($method == "GET" && $route == "/item") {
        $id = intval(query_param($path, "id"));  // ?id=2  ("" -> 0)
        return json_encode(try find($all, $id)); // missing: 404 {"error":"Not Found"}
    }
    if ($method == "POST" && $route == "/items") {
        $name = json_string($body, "name");      // {"name":"pie","price":60}
        if ($name == "") { throw error(400, "name is required"); }
        return json_encode(new Item(3, $name, json_int($body, "price")));
    }
    throw error(404, "no route $route");
}

serve(8080, "handle");
```

### Where to read what (plain-text links)

| to write | read |
|---|---|
| anything | [ai-guide.md](https://raw.githubusercontent.com/pin-language/docs/HEAD/ai-guide.md) (start here) |
| every built-in function, with examples | [reference/README.md](https://raw.githubusercontent.com/pin-language/docs/HEAD/reference/README.md) |
| a JSON HTTP API | [reference/http.md](https://raw.githubusercontent.com/pin-language/docs/HEAD/reference/http.md) · [reference/json.md](https://raw.githubusercontent.com/pin-language/docs/HEAD/reference/json.md) |
| a database (SQLite) | [reference/database.md](https://raw.githubusercontent.com/pin-language/docs/HEAD/reference/database.md) |
| classes, objects in lists | [learn/13-classes.md](https://raw.githubusercontent.com/pin-language/docs/HEAD/learn/13-classes.md) |
| parallel work, real time (go / await, Hub, SSE) | [learn/14-tasks-and-realtime.md](https://raw.githubusercontent.com/pin-language/docs/HEAD/learn/14-tasks-and-realtime.md) |
| an MCP server | the package [pin-language/mcp](https://raw.githubusercontent.com/pin-language/mcp/HEAD/README.md): `pin get github.com/pin-language/mcp@v0.1.0` |
| several files, packages | [learn/12-modules-and-packages.md](https://raw.githubusercontent.com/pin-language/docs/HEAD/learn/12-modules-and-packages.md) |

## 👤 If you are a person

**Give your agent this:**

```text
Read https://raw.githubusercontent.com/pin-language/docs/HEAD/ai-guide.md before writing any code.
Write the program in Pin and run pin check --json after every change.
```

**Install** (macOS or Linux, no sudo; needs Python 3.10+ and clang — the installer checks both):

```bash
curl -fsSL https://raw.githubusercontent.com/pin-language/pin/HEAD/install.sh | sh
printf '%s\n' 'echo "Hello from Pin\n";' > hello.pn && pin hello.pn
```

Most code is now written by AI agents, and they already know PHP. Pin keeps that syntax but moves the
mistakes from production to compile time. Every variable has one type, and compile-time ownership (like
Rust, simplified) manages memory. Every path through a function is checked, and every error says how to fix
it. Programs compile through LLVM into one native binary, including a multi-threaded HTTP server. People
learning by hand: [Learn Pin](https://github.com/pin-language/docs/blob/HEAD/learn/README.md), 14 short
chapters.

## Speed

Same algorithm in every language, same output checked. Apple M2, best of 3 runs (seconds, lower is better).

| | Pin | Rust | Go | PHP + JIT | PHP | Python |
|---|---|---|---|---|---|---|
| sieve — primes below 20M | **0.042** | 0.043 | 0.041 | 0.652 | 1.337 | 4.283 |
| collatz — longest chain below 1M | **0.165** | 0.131 | 0.175 | 0.714 | 3.204 | 6.302 |
| pi — 100M terms | **0.090** | 0.090 | 0.091 | 0.930 | 2.235 | 13.239 |
| list build + sum | **0.114** | 0.131 | 0.183 | 0.440 | 1.192 | 4.754 |

Rust- and Go-level speed, about 4–15× faster than PHP with JIT. These are small CPU loops, not applications,
and on macOS differences under ~20% are noise. Method, memory and HTTP numbers:
[benchmarks](https://github.com/pin-language/benchmarks).

## Status

**Preview (0.8).**

- **Language:** functions; classes (no inheritance) with methods and `&` parameters; `int float bool string`;
  `list<T>` and ordered `map<K,V>`, nested freely; `throws` / `try` / `catch`; ownership checking;
  bounds-checked lists.
- **Concurrency:** `go` / `await` tasks.
- **HTTP:** a multi-threaded HTTP server with headers, timeouts, a Hub and Server-Sent Events; an http://
  client; `run()`.
- **Data:** SQLite with injection-proof SQL; nested JSON reading; stdin.
- **Code organisation:** modules and Git packages pinned in `pin.lock`.

Not yet: inheritance, `null`, `finally`, closures, PostgreSQL. TLS is left to a proxy by design. Not for
production.

[Open an issue](https://github.com/pin-language/pin/issues). We especially want code your agent expected to
work but that could not compile.

<sub>Made by PinaTo Co., Ltd. · MIT licensed</sub>
