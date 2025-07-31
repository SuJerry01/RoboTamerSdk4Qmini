
## 操作清单，涵盖：

1. 在 x86 环境下通过 pip 安装 ONNX Runtime

2. 在 CMake 中自动定位并链接到 Conda 环境下的 libonnxruntime.so

3. 设置 rpath，让可执行文件运行时能找到 ONNX Runtime 的共享库

4. 以 x86 架构成功编译你的项目


### 1、准备 Conda 环境并安装 ONNX Runtime

```

# 1. 创建并激活一个干净的 Python 3.11 环境
conda create -n qmini python=3.11 -y
conda activate qmini

# 2. 安装 Python 依赖
pip install numpy<2

# 3. 安装 ONNX Runtime（x86_64 版本）
pip install onnxruntime-gpu==1.19.0

# （可选）检查库文件是否就位
ls $CONDA_PREFIX/lib/python3.11/site-packages/onnxruntime/capi/libonnxruntime.so*
# 应该至少看到：
#   libonnxruntime.so
#   libonnxruntime.so.1.19.0
```
### 2、修改 CMakeLists.txt
```
# 1. 先去 Conda 下的 onnxruntime/capi 目录里找

find_library(ONNXRT_LIB
  NAMES onnxruntime
  HINTS
    $ENV{CONDA_PREFIX}/lib/python3.11/site-packages/onnxruntime/capi
    $ENV{CONDA_PREFIX}/lib
)
if (NOT ONNXRT_LIB)
  message(FATAL_ERROR "找不到 Conda 环境下的 libonnxruntime.so，请确认已激活环境并安装 onnxruntime(-gpu)")
endif()
message(STATUS "Found ONNX Runtime library: ${ONNXRT_LIB}")

# 2. 链接时加入 ONNX Runtime

target_link_libraries(run_interface ddsc unitree_sdk2 ddscxx rt pthread ${ONNXRT_LIB}  yaml-cpp Python3::Python JsonCpp::JsonCpp libUnitreeMotorSDK_Linux64.so)

# 3. 设置 rpath，让运行时能找到上面找到的 libonnxruntime.so

set_target_properties(run_interface PROPERTIES
  BUILD_RPATH "$ENV{CONDA_PREFIX}/lib/python3.11/site-packages/onnxruntime/capi"
)
```

### 3、以 x86 架构编译

```
# 1. 回到项目 build 目录
（如果不存在先 mkdir build && cd build）

cd ~/RoboTamerSdk4Qmini
mkdir -p build && cd build

```

### 2. 调用 CMake，指定 x86_64（Linux64）平台
```
cmake .. \
  -DPLATFORM=Linux64 \
  -DCMAKE_BUILD_TYPE=Release
```
### 3. 编译并进入 bin
```
make -j$(nproc) 
cd ../bin
```
(or)调用 2. CMake(x86)平台 + 3. 编译并进入 bin 
```
cmake -DPLATFORM=x86 .. && make && cd ../bin
(add x86)
```

### 4、运行并验证
```
# 确保环境变量指向 Conda 的共享库目录
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib/python3.11/site-packages/onnxruntime/capi:$LD_LIBRARY_PATH

# 直接跑
./run_interface
```

