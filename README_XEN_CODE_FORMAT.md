# Build clang-format
```
cd ~/w/llvm-project

test -d build || mkdir build
cd build
cmake -DLLVM_ENABLE_PROJECTS=clang -DCMAKE_BUILD_TYPE=Debug -G "Unix Makefiles" --target clang-format ../llvm

make clang-format -j12
```

# Use clang-format

Put .clang-format file in one of the parent directories of the source file (or current directory for stdin).

For example, copy .clang-format file to the Xen project directory.

```
cd /path/to/xen/project
```
Print formatted code to console:
```
time find ./xen -name '*.c' -print0 | xargs -0 -n 1 -P 12 ~/w/llvm-project/build/bin/clang-format
```
Apply clang-format changes to .c files:
```
time find ./xen -name '*.c' -print0 | xargs -0 -n 1 -P 12 ~/w/llvm-project/build/bin/clang-format -i
```
Dump clang-format options from a specific style:
```
~/w/llvm-project/build/bin/clang-format -style=llvm -dump-config > .clang-format
```

# Test new features in clang-format

Run all tests for clang-format:
```
~/w/llvm-project/build/tools/clang/test$ make check-clang-unit -j4
```
