---
title: Fuzzing Tools -- DIE
layout: post
---
# 一、What is DIE?
DIE tool is an experimental gray box testing tool, and the related article "Fuzzing JavaScript Engines with Aspect preserving Mutation" was published at the top security conference SSP in 2020. This tool proposes a subtree replacement mutation method that preserves key structures by analyzing the triggering conditions of specific vulnerabilities. For the testing of mainstream JS engines ChakraCore, JavaScript Core, and V8, this tool triggered 5.7x unique crashes and 2.4x syntax valid test cases compared to the baseline fuzz. The experimental results show that this method outperforms the baseline fuzzer in triggering vulnerabilities and generating semantically effective test cases.
# 二、Steps to Use
## 1.Configure Environment&Compile
- ubuntu 22.04
1. Configure dependencies
```bash
apt-get update
apt-get install -y build-essential redis git python3 clang-6
```
- Configure npm 14
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
chmod +x ~/.nvm/nvm.sh
source ~/.bashrc
nvm –v
nvm install 14
```
- Configure ts node
```bash
npm install -g ts-node
```
2. Download source code
```bash
git clone https://github.com/sslab-gatech/DIE.git
```
3. Compile
In the root directory
```
./compile.sh
```
## 2.Use
1. Creating Test Cases 
```bash
mkdir testcase
cd testcase
mkdir input.js
```
2. Perform type inference of initial seeds
```bash
cd DIE/fuzz/TS/typer
python3 typer.py testcase/
```
3. Compile the AFL section of DIE
```bash
cd DIE/fuzz/afl
make
```
4. Instrument the Target
```bash
git clone https://github.com/jerryscript-project/jerryscript.git
CC=/opt/DIE/fuzz/afl/afl-gcc /usr/bin/python3 ./tools/build.py --clean --debug --compile-flag=-m32 --lto=off --error-message=on --system-allocator=on --compile-flag=-Wno-uninitialized --compile-flag=-Wno-missing-field-initializers --link-lib="m" --stack-limit=15 --compile-flag=-g --compile-flag=-O0
```
5. Upload seeds to Redis database
```bash
./redis-server # 启动redis数据库
cd DIE/fuzz/afl
./afl-fuzz -i testcase/ -o output/ -m none -- jerryscript/build/bin/jerry @@
```
6. Test begins
```bash
./afl-fuzz -o output/ -m none -- jerryscript/build/bin/jerry @@
```
---
# Summary
The above is the use of the fuzz testing tool DIE. The underlying of the DIE tool is still based on the grey box fuzzy testing tool AFL, which extends and implements dynamic type inference and type constrained subtree replacement mutation methods. Implement distributed testing functionality for multiple fuzzers based on Redis database.

# Problem to be solved
When compiling AFL's LLVM with clang-6, an error occurs
```bash
unset AFL_USE_ASAN AFL_USE_MSAN AFL_INST_RATIO; AFL_QUIET=1 AFL_PATH=. AFL_CC=clang ../afl-clang-fast -O0 -funroll-loops -Wall -D_FORTIFY_SOURCE=2 -g -Wno-pointer-sign -DAFL_PATH=\"/usr/local/lib/afl\" -DBIN_PATH=\"/usr/local/bin\" -DVERSION=\"2.52b\"  ../test-instr.c -o test-instr
clang-6.0: error: unable to execute command: Segmentation fault (core dumped)
clang-6.0: error: clang frontend command failed due to signal (use -v to see invocation)
clang version 6.0.0 (tags/RELEASE_600/final)
Target: x86_64-unknown-linux-gnu
Thread model: posix
InstalledDir: /opt/clang-6/bin
clang-6.0: note: diagnostic msg: PLEASE submit a bug report to http://llvm.org/bugs/ and include the crash backtrace, preprocessed source, and associated run script.
clang-6.0: error: unable to execute command: Segmentation fault (core dumped)
clang-6.0: note: diagnostic msg: Error generating preprocessed source(s).
make: *** [Makefile:98: test_build] Error 254
```