---
layout: post
title: "mac osx下编译v8"
tags:
    - v8
    - mac osx
description: ""
---

v8相信大家都不陌生，本文将介绍一下，如何在mac osx下编译v8，并用一个例子来试验编译结果。

## 编译

首先获取一下源代码，v8在github上有镜像，可以直接clone github上的。

```bash
git clone https://github.com/v8/v8.git
```

代码量较大，请耐心等待...

下载完成后，我们还需要下载编译前的依赖。

```bash
make builddeps
```

_Note: 以上命令都比较悲剧，因为要跨越我朝的墙...建议使用vpn吧，用goagent也可以，不过会很慢。_

依赖下载完成之后，我们就可以正式build了：

```bash
make
```

更多编译选项，请查看[官网](https://code.google.com/p/v8/wiki/BuildingWithGYP)。

<!-- more -->

编译完成之后，在out目录下会生成类似以下目录：

* `out/ia32.release`
* `out/ia32.debug`
* `out/x64.release`
* `out/x64.debug`
* ...

按照平台划分，我的机器是x64，debug与release也有一些区别，比如log以及一些新特性等

我们以`x64.debug`为例，在该目录下，有一些可执行文件，比如`d8`，这个可执行文件可用来直接在命令行下运行javascript（当然，不包括dom、bom等）。

```bash
./d8
```

运行d8后，就会进入一个可交互的shell环境，可执行js，例如：

```bash
d8> 1 + 2
3
```

## 例子

在`out/x64.debug`下，除了一些可执行文件外，还生成了一些静态链接库，比如：

* `out/x64.debug/libv8_base.a`
* `out/x64.debug/libv8_libbase.a`
* ...

这些文件在编译v8一些例子的时候，需要链接进来。

我们来编译一下官网的Hello World例子，代码如下：

```cpp
#include <v8.h>

using namespace v8;

int main(int argc, char* argv[]) {
  // Create a new Isolate and make it the current one.
  Isolate* isolate = Isolate::New();
  Isolate::Scope isolate_scope(isolate);

  // Create a stack-allocated handle scope.
  HandleScope handle_scope(isolate);

  // Create a new context.
  Local<Context> context = Context::New(isolate);

  // Enter the context for compiling and running the hello world script.
  Context::Scope context_scope(context);

  // Create a string containing the JavaScript source code.
  Local<String> source = String::NewFromUtf8(isolate, "'Hello' + ', World!'");

  // Compile the source code.
  Local<Script> script = Script::Compile(source);

  // Run the script to get the result.
  Local<Value> result = script->Run();

  // Convert the result to an UTF8 string and print it.
  String::Utf8Value utf8(result);
  printf("%s\n", *utf8);
  return 0;
}
```

链接静态库，编译HelloWorld.cc：

```bash
clang++ -I./include samples/HelloWorld.cc -o ./out/HelloWorld \
    -Wl, ./out/x64.debug/libv8_base.a \
    -Wl, ./out/x64.debug/libv8_libbase.a \
    -Wl, ./out/x64.debug/libicui18n.a \
    -Wl, ./out/x64.debug/libicudata.a \
    -Wl, ./out/x64.debug/libicuuc.a \
    -Wl, ./out/x64.debug/libv8_snapshot.a \
    -stdlib=libstdc++ 
```

__注意最后的`stdlib`参数，最新版的mac使用的是llvm编译器，其默认的标准库是`libc++`，但v8依赖的是`libstdc++`。__

编译完成后，在out目录下就生成了可执行文件HelloWorld：

```bash
$ ./HelloWorld
Hello World!
```

大功告成！

网上其实有很多类似的v8编译博文，不过大部分都是比较老的文章，最新版的编译方式还不大一样，希望本文对您有所帮助！
