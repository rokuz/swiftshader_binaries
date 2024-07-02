# SwiftShader binaries for multiple platforms

Download binaries from the latest [release](https://github.com/rokuz/swiftshader_binaries/releases/tag/release_1).

## Rebuild

SwiftShader's commit: [a0ec371d8331d787c61eccc89fb411019330314e](https://github.com/google/swiftshader/commit/a0ec371d8331d787c61eccc89fb411019330314e)

LLVM: [llvmorg-18.1.8](https://github.com/llvm/llvm-project/tree/llvmorg-18.1.8)

You need to apply [the following patch](https://github.com/rokuz/swiftshader_binaries/blob/main/swiftshader.patch) to swiftshader.

### macOS
#### arm64
```
cd build
cmake .. -DSWIFTSHADER_BUILD_TESTS=ON -DREACTOR_BACKEND="LLVM-Submodule" -DCMAKE_BUILD_TYPE=Release
cmake --build . --parallel --config Release
./vk-unittests
```

#### x86_64
```
cd build
cmake .. -DSWIFTSHADER_BUILD_TESTS=ON -DREACTOR_BACKEND="LLVM-Submodule" -DCMAKE_OSX_ARCHITECTURES="x86_64" -DCMAKE_BUILD_TYPE=Release
cmake --build . --parallel --config Release
./vk-unittests
```

### Windows
#### arm64
```
```

#### x86_64
```
cd build
cmake .. -DSWIFTSHADER_BUILD_TESTS=ON -DREACTOR_BACKEND="LLVM-Submodule" -DCMAKE_BUILD_TYPE=Release
cmake --build . --parallel --config Release
cd Release
.\vk-unittests.exe
```
