# SwiftShader binaries for multiple platforms

Download binaries from the latest [release](https://github.com/rokuz/swiftshader_binaries/releases/tag/release_1).

## Rebuild

SwiftShader's commit: [a0ec371d8331d787c61eccc89fb411019330314e](https://github.com/google/swiftshader/commit/a0ec371d8331d787c61eccc89fb411019330314e)

LLVM: [llvmorg-18.1.8](https://github.com/llvm/llvm-project/tree/llvmorg-18.1.8)

SwiftShader patch:
```
diff --git a/CMakeLists.txt b/CMakeLists.txt
index c5cd16d55..6f86054e9 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -681,6 +681,8 @@ set_target_properties(llvm PROPERTIES FOLDER "third_party")
 if(${REACTOR_BACKEND} STREQUAL "LLVM-Submodule")
     set(LLVM_INCLUDE_TESTS FALSE)
     set(LLVM_ENABLE_RTTI TRUE)
+    set(LLVM_ENABLE_ZSTD FALSE)
+    #set(LLVM_TARGETS_TO_BUILD "X86")
     add_subdirectory(${THIRD_PARTY_DIR}/llvm-project/llvm EXCLUDE_FROM_ALL)
     if(ARCH STREQUAL "aarch64")
         llvm_map_components_to_libnames(llvm_libs orcjit aarch64asmparser aarch64codegen)
diff --git a/src/Reactor/LLVMJIT.cpp b/src/Reactor/LLVMJIT.cpp
index 262c69e8e..6fcb2b4e6 100644
--- a/src/Reactor/LLVMJIT.cpp
+++ b/src/Reactor/LLVMJIT.cpp
@@ -47,7 +47,7 @@ __pragma(warning(push))
 #include "llvm/IR/DiagnosticInfo.h"
 #include "llvm/IR/Verifier.h"
 #include "llvm/Support/CommandLine.h"
-#include "llvm/Support/Host.h"
+#include "llvm/TargetParser/Host.h"
 #include "llvm/Support/TargetSelect.h"
 #include "llvm/Transforms/InstCombine/InstCombine.h"
 #include "llvm/Transforms/Instrumentation/AddressSanitizer.h"
@@ -190,7 +190,7 @@ public:
 private:
 	JITGlobals(llvm::orc::JITTargetMachineBuilder &&jitTargetMachineBuilder, llvm::DataLayout &&dataLayout);
 
-	static llvm::CodeGenOpt::Level toLLVM(int level);
+	static llvm::CodeGenOptLevel toLLVM(int level);
 
 	const llvm::orc::JITTargetMachineBuilder jitTargetMachineBuilder;
 	const llvm::DataLayout dataLayout;
@@ -303,26 +303,26 @@ JITGlobals::JITGlobals(llvm::orc::JITTargetMachineBuilder &&jitTargetMachineBuil
 {
 }
 
-llvm::CodeGenOpt::Level JITGlobals::toLLVM(int level)
+llvm::CodeGenOptLevel JITGlobals::toLLVM(int level)
 {
 	// TODO(b/173257647): MemorySanitizer instrumentation produces IR which takes
 	// a lot longer to process by the machine code optimization passes. Disabling
 	// them has a negligible effect on code quality but compiles much faster.
 	if(__has_feature(memory_sanitizer))
 	{
-		return llvm::CodeGenOpt::None;
+		return llvm::CodeGenOptLevel::None;
 	}
 
 	switch(level)
 	{
-	case 0: return llvm::CodeGenOpt::None;
-	case 1: return llvm::CodeGenOpt::Less;
-	case 2: return llvm::CodeGenOpt::Default;
-	case 3: return llvm::CodeGenOpt::Aggressive;
+	case 0: return llvm::CodeGenOptLevel::None;
+	case 1: return llvm::CodeGenOptLevel::Less;
+	case 2: return llvm::CodeGenOptLevel::Default;
+	case 3: return llvm::CodeGenOptLevel::Aggressive;
 	default: UNREACHABLE("Unknown Optimization Level %d", int(level));
 	}
 
-	return llvm::CodeGenOpt::Default;
+	return llvm::CodeGenOptLevel::Default;
 }
 
 class MemoryMapper final : public llvm::SectionMemoryManager::MemoryMapper
diff --git a/src/Reactor/LLVMReactor.cpp b/src/Reactor/LLVMReactor.cpp
index dda6d9166..03fcd0b78 100644
--- a/src/Reactor/LLVMReactor.cpp
+++ b/src/Reactor/LLVMReactor.cpp
@@ -3851,7 +3851,7 @@ void promoteFunctionToCoroutine()
 	auto i1Ty = llvm::Type::getInt1Ty(*jit->context);
 	auto i8Ty = llvm::Type::getInt8Ty(*jit->context);
 	auto i32Ty = llvm::Type::getInt32Ty(*jit->context);
-	auto i8PtrTy = llvm::Type::getInt8PtrTy(*jit->context);
+	auto i8PtrTy = llvm::PointerType::getUnqual(*jit->context);
 	auto promiseTy = jit->coroutine.yieldType;
 	auto promisePtrTy = promiseTy->getPointerTo();
 
@@ -4016,7 +4016,7 @@ void Nucleus::createCoroutine(Type *YieldType, const std::vector<Type *> &Params
 	// coroutine.
 	auto voidTy = llvm::Type::getVoidTy(*jit->context);
 	auto i1Ty = llvm::Type::getInt1Ty(*jit->context);
-	auto i8PtrTy = llvm::Type::getInt8PtrTy(*jit->context);
+	auto i8PtrTy = llvm::PointerType::getUnqual(*jit->context);
 	auto handleTy = i8PtrTy;
 	auto boolTy = i1Ty;
 	auto promiseTy = T(YieldType);
```

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
