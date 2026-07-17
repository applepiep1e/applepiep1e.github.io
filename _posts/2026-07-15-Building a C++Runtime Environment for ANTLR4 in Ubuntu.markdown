---
title: Building a C++Runtime Environment for ANTLR4 in Ubuntu
layout: post
---
# 一、What is ANTLR?

**ANTLR** (Another Tool for Language Recognition) is a powerful syntax analyzer generator used for reading, processing, executing, or translating structured text or binary files. It is widely used to build language tools such as parsers, interpreters, and compilers for programming languages, configuration files, and data formats (such as JSON, XML).

# 二、Environment Configuration
## Basic dependencies
Ubuntu version: Ubuntu: 22.04

```bash
DEBIAN_FRONTEND=noninteractive apt-get install -y openjdk-17-jdk-headless
java -version
```
## Configure ANTLR4
1. Download the tool

```bash
cd /usr/local/lib
sudo wget http://www.antlr.org/download/antlr-4.13.2-complete.jar
```
2. Create a Command Line for the Tool
Create an antlr4 file and add the following content:

```bash
#! /bin/bash
export CLASSPATH=".:/usr/local/lib/antlr-4.13.2-complete.jar:$CLASSPATH"
java -jar /usr/local/lib/antlr-4.13.2-complete.jar "$@"
```
```bash
chmod +x antlr4
sudo mv antlr4 /usr/local/bin
```
```bash
nano ~/.bashrc
```
Set the compilation process to the command line.
Add the following content:

```bash
export CLASSPATH=".:/root/antlr4/antlr-4.13.2-complete.jar:$CLASSPATH"
alias antlr4="java -jar /root/antlr4/antlr-4.13.2-complete.jar"
alias grun="java org.antlr.v4.gui.TestRig"
```
```bash
source ~/.bashrc
```
## C++ Runtime
Download and compile runtime libraries. Note that the version of the runtime library and the version of the. jar tool should be consistent.

```bash
wget https://www.antlr.org/download/antlr4-cpp-runtime-4.13.2-source.zip
unzip antlr4-cpp-runtime-4.13.2-source.zip && cd antlr4-cpp-runtime-4.13.2-source
cmake .. -DCMAKE_BUILD_TYPE=Release -DWITH_DEMO=OFF -DWITH_TESTS=OFF
make -j$(nproc)
make install
ldconfig
```
# 三、Simple Example
1. Prepare the Hello.g4 syntax file

```bash
grammar Hello; 
r : 'hello' ID ; 
ID : [a-zA-Z]+ ; 
WS : [ \t\r\n]+ -> skip ;
```
Prepare the main.cpp file

```cpp
#include <iostream>
#include "antlr4-runtime/antlr4-runtime.h" 
#include "HelloLexer.h"
#include "HelloParser.h"

using namespace antlr4;

int main() {
    std::string input_text = "hello world";
    ANTLRInputStream input(input_text);
    HelloLexer lexer(&input);
    CommonTokenStream tokens(&lexer);
    HelloParser parser(&tokens);
    tree::ParseTree* tree = parser.r();

    std::cout << "Input Characters: " << input_text << std::endl;
    std::cout << "Output Parse Tree: " << tree->toStringTree(&parser) << std::endl;
    return 0;
}
```
2. Compile the file

```bash
java -jar ~/antlr4/antlr-4.13.2-complete.jar -Dlanguage=Cpp Hello.g4
g++ -std=c++17 main.cpp Hello*.cpp -o hello_world -I/usr/local/include/antlr4-runtime -lantlr4-runtime
./hello_world
```
## View syntax tree using Grun
1. Use the. g4 file above

```bash
javac Hello*.java
grun Hello r -tree
# Input `hello world``
# Input ctrl+D to End the Input.
# the expected result:
## hello world
## (r hello world)
```
---
# Summary
The above is the process of building the compiler tool ANTLR4 and a runtime that supports C++language on Ubuntu 22.04. Using ANTLR4 allows for the creation of custom compilers.