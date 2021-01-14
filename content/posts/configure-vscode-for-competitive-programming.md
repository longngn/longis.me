---
title: Configure VSCode for competitive programming
date: 2017-08-30
---

## What is competitive programming?

Sites like CodeForces, TopCoder, HackerRank, CodeChef,... ACM-ICPC, Olympiad in Informatics (for high school students), Google Code Jam, Facebook Hacker Cup,...
Anything involves solving short problems in 2--5 hours by code using algorithms and data structures.

<!--more-->

### Requirements

- High coding speed.
- One-click compile and run for one source file.
- Automatic linting
- C++ (though Java and Python are viable, C++ is still preferable for superior performance).
- Debugging

### Why VSCode?

Lightweight, customizable, big userbase, open-source and has an awesome community.

### Limitations

I still haven't been able to figure out how to debug, even after reading VSCode's official guide. So I actually have to resort to Xcode for (seldom) C++ debugging.

## Instructions

Beware! The instructions below have only been tested on MacOS. I need some volunteers to help me test them on the other OSes.

1.  Setup a C++ compiler of your choice (GCC or Clang). After this step, you should be able to run the code by `g++ a.cpp -o a.out && ./a.out` or `clang++ a.cpp -o a.out && ./a.out`
2.  Setup these extensions:

- [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) obviously
- [C/C++ Clang Command Adapter](https://marketplace.visualstudio.com/items?itemName=mitaki28.vscode-clang) for linting, you can config Clang flags to *-Weverything* and disable warnings you don't need.
- [Code Runner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner) for one-click compile and run, you should set Code Runner to run in VSCode's integrated Terminal, otherwise it would be impossible to use standard input.

3.  Set your hotkey for Code Runner, create your snippets and templates, and you are ready to go.

_By the way, here is my personal settings:_

```json
"code-runner.runInTerminal": true,
"code-runner.executorMap": {
    "cpp": "clang++ -Wl,-stack_size -Wl,0x10000000 -g -Wall -Wextra -pedantic -std=gnu++14 -O2 -Wshadow -Wformat=2 -Wfloat-equal -Wconversion -Wcast-qual -Wcast-align -D_GLIBCXX_DEBUG -D_GLIBCXX_DEBUG_PEDANTIC -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fsanitize=address -fsanitize=undefined -fno-sanitize-recover=all -fstack-protector $fileName -o Executable && ./Executable"
},
"code-runner.saveFileBeforeRun": true,
"code-runner.preserveFocus": false,
"code-runner.ignoreSelection": true,
"clang.cxxflags": ["-Weverything", "-pedantic", "-std=gnu++14", "-O2", "-Wno-c++98-compat", "-Wno-c++98-compat-pedantic", "-Wno-missing-variable-declarations", "-Wno-missing-prototypes"]
```

## Useful stuff

- [Setting Sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync)
- [Catching silly mistakes with GCC](http://codeforces.com/blog/entry/15547)
- [C++ programming with Visual Studio Code](https://code.visualstudio.com/docs/languages/cpp)

**UPDATE:** There seems to be a cool extension dedicated for competitive programming which is [IORun](https://marketplace.visualstudio.com/items?itemName=hoangnc.io-run). You guys should check it out too!
