# tutorial

## What and why WebAssembly, a bit of history
- NaCL/PNaCl, asm.js: too slow
- wasm: 
  - fast 
  - all major tech companies in support
  - overcomes many disadvantages of pure javascript (compilation, optimization, size, ...)
  - does not matter anymore which language you pick
  - still making specs based on proposals
  - reports a lot of improvement on speed
- recent movements
  - off-the-browser: WASI
  - Photoshop, AutoCAD, Google Earth, ...
  - involvement in blockchain
  - danger (binary)
  - advent of wasm standalone runtimes like `wasmtime`

## Basic setup for **emscripten**
### 1. Download & configure emscripten

Help link: https://emscripten.org/docs/getting_started/downloads.html
```bash
# Get the emsdk repo
git clone https://github.com/emscripten-core/emsdk.git

# Enter that directory
cd emsdk

# Fetch the latest version of the emsdk (not needed the first time you clone)
git pull

# Download and install the latest SDK tools.
./emsdk install latest

# Make the "latest" SDK "active" for the current user. (writes ~/.emscripten file)
./emsdk activate latest

# Activate PATH and other environment variables in the current terminal
source ./emsdk_env.sh
```

### 2. Find `emcc` 
> Navigate with the command prompt to the emscripten directory under the SDK. This is a folder below the emsdk root directory, typically `<emsdk root directory>/fastcomp/emscripten/` (for the older “fastcomp” compiler; for the newer upstream LLVM wasm backend it will be `<emsdk root directory>/upstream/emscripten/`). The examples below will depend on finding files relative to that location.

Help link: https://emscripten.org/docs/getting_started/Tutorial.html

```bash
$ cd <emsdk root directory>/upstream/emscripten/
$ ls
AUTHORS                emconfigure.py
CONTRIBUTING.md        emlink.py
ChangeLog.md           emmake
LICENSE                emmake.bat
README.md              emmake.py
__pycache__            emranlib
cmake                  emranlib.bat
docs                   emrun
em++                   emrun.bat
em++.bat               emrun.py
em++.py                emscons
em-config              emscons.py
em-config.bat          emscripten-version.txt
emar                   emscripten.py
emar.bat               emscripten.pyc
emar.py                media
embuilder.py           package-lock.json
emcc                   package.json
emcc.bat               site
emcc.py                src
emcmake                system
emcmake.bat            tests
emcmake.py             third_party
emconfigure            tools
emconfigure.bat
```

### 3. Check it's correctly installed

```bash
(base) ➜  emscripten git:(master) ./emcc -v
emcc (Emscripten gcc/clang-like replacement + linker emulating GNU ld) 1.39.0
clang version 10.0.0 (/b/s/w/ir/cache/git/chromium.googlesource.com-external-github.com-llvm-llvm--project e44524736c4a97ae4fb37193e58647f838f6d36a)
Target: x86_64-apple-darwin18.7.0
Thread model: posix
InstalledDir: /Users/jm/Documents/Code/emsdk/upstream/bin
shared:INFO: (Emscripten: Running sanity checks)
```

### 4. Make a shortcut so that it's easy to access `emcc`

```bash
$ vim ~/.profile (or .zshrc or whatever)

... at the end of .profile, perhaps:

alias emcc="<path-to-emsdk>/upstream/emscripten/emcc"

$ source ~/.profile
```

### 5. Make `hello-world.cpp`

```
touch hello-world.cpp
echo "#include <iostream>
int main() {
  std::cout << "hello, world!\n";
  return 0;
}"  >> hello-world.cpp
```

### 6. Check if `emcc` is working with `hello-world.cpp` example

```bash
$ emcc hello-world.cpp -o hello-world.js # this is going to generate 
(base) ➜  tutorial git:(master) ✗ emcc hello-world.cpp
cache:INFO: generating system library: libcompiler_rt.a... (this will be cached in "/Users/jm/.emscripten_cache/wasm-obj/libcompiler_rt.a" for subsequent builds)
cache:INFO:  - ok
cache:INFO: generating system library: libc-wasm.a... (this will be cached in "/Users/jm/.emscripten_cache/wasm-obj/libc-wasm.a" for subsequent builds)
cache:INFO:  - ok
cache:INFO: generating system library: libc++-noexcept.a... (this will be cached in "/Users/jm/.emscripten_cache/wasm-obj/libc++-noexcept.a" for subsequent builds)
cache:INFO:  - ok
cache:INFO: generating system library: libc++abi-noexcept.a... (this will be cached in "/Users/jm/.emscripten_cache/wasm-obj/libc++abi-noexcept.a" for subsequent builds)
cache:INFO:  - ok
cache:INFO: generating system library: libdlmalloc.a... (this will be cached in "/Users/jm/.emscripten_cache/wasm-obj/libdlmalloc.a" for subsequent builds)
cache:INFO:  - ok
cache:INFO: generating system library: libpthread_stub.a... (this will be cached in "/Users/jm/.emscripten_cache/wasm-obj/libpthread_stub.a" for subsequent builds)
cache:INFO:  - ok
cache:INFO: generating system library: libc_rt_wasm.a... (this will be cached in "/Users/jm/.emscripten_cache/wasm-obj/libc_rt_wasm.a" for subsequent builds)
cache:INFO:  - ok
(base) ➜  tutorial git:(master) ✗ ls tests/results
hello-world      hello-world.js   hello-world.wasm
```

Now you know that `emcc` is working well. 

## Run it from a browser
### 1. Install and launch http-server to host .wasm file

```bash
npm i -g http-server

http-server --cors . # current directory should have .wasm file
```

### 2. Write `index.html`

```html
<html>
<head>
  <script>
    window.onload = () => {
     const importObject = { imports: { imported_func: arg => console.log(arg) } };

      WebAssembly.instantiateStreaming(fetch('localhost:8000/hello-world.wasm'), importObject)
      .then(obj => obj.instance.exports.exported_func());
    }
  </script>
</head>
<body>
</body>
</html>
```

## Basic setup for Assemblyscript

```bash
npm install --save-dev assemblyscript

npx asinit .
```
