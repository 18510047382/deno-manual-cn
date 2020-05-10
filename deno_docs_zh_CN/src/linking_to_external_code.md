# 与外部代码连接

在 [入门](./getting_started.md) 章节中，我们看到 Deno 能够从 URL 执行脚本。像浏览器中的 JavaScript 一样，Deno 可以从 URL 直接导入代码库。

这个示例使用 URL 来导入一个断言库：

```ts
import { assertEquals } from "https://deno.land/std/testing/asserts.ts";

assertEquals("hello", "hello");
assertEquals("world", "world");

console.log("Asserted! 🎉");
```

尝试运行一下：

```shell
$ deno run test.ts
Compile file:///mnt/f9/Projects/github.com/denoland/deno/docs/test.ts
Download https://deno.land/std/testing/asserts.ts
Download https://deno.land/std/fmt/colors.ts
Download https://deno.land/std/testing/diff.ts
Asserted! 🎉
```

对于这个程序，我们不需要提供 `--allow-net` 选项。当它访问网络时，Deno 运行时有着特殊权限来下载模块并缓存到磁盘。

Deno 在一个特殊目录缓存了远程模块，该路径可以被 `$DENO_DIR` 指定，如果没有指定，默认为系统缓存目录。下一次运行这个程序时无需下载。如果这个程序没有改动，它不会被再次编译。

系统缓存目录默认为：

- Linux/Redox: `$XDG_CACHE_HOME/deno` or `$HOME/.cache/deno`
- Windows: `%LOCALAPPDATA%/deno` (`%LOCALAPPDATA%` = `FOLDERID_LocalAppData`)
- macOS: `$HOME/Library/Caches/deno`

如果失败，该路径设置为 `$HOME/.deno`。

## FAQ

### 如果 `https://deno.land/` 宕机会发生什么？

依赖外部服务在开发时很方便，但在生产环境很脆弱。生产级软件总是应该打包所有依赖。

在 Deno 中，您应该将 `$DENO_DIR` 检入版本控制系统，并在运行时指定 `$DENO_DIR` 环境变量。

### 如何信任可能更改的 URL？

使用 `--lock` 命令行选项，通过一个锁文件 (lock file)，您可以确保自己运行的是所期望的代码。更多信息请看[这里](./linking_to_external_code/integrity_checking.md)

### 如何导入特定版本？

只需在 URL 中指定版本。举个例子，这个 URL 指定了要运行的版本 `https://unpkg.com/liltest@0.0.5/dist/liltest.js`。结合之前提到的在生产中设置 `$DENO_DIR` 的方法，您可以完全指定要运行的代码，并且无需访问网络。

### 到处导入 URL 似乎很麻烦

> 如果其中一个 URL 链接到一个完全不同的库版本，该怎么办？

> 在大型项目中到处维护 URL 是否容易出错？

解决办法是在一个中心化的 `deps.ts` 中重新导出所依赖的外部库，它和 Node 的 `package.json` 具有相同的作用。

举个例子，您正在一个大型项目中使用一个断言库，您可以创建一个 `deps.ts` 文件来导出第三方代码，而不是到处导入 `"https://deno.land/std/testing/asserts.ts"`。

```ts
export {
  assert,
  assertEquals,
  assertStrContains,
} from "https://deno.land/std/testing/asserts.ts";
```

在这个项目中，您可以从 `deps.ts` 导入，避免对相同的 URL 产生过多引用。

```ts
import { assertEquals, runTests, test } from "./deps.ts";
```

这种设计规避了包管理、集中式代码存储库和多余的文件格式带来的过多复杂性。
