# Vulkan 基础教程 · 配套代码

视频系列「Vulkan 基础教程」每一期、每一步的代码。内容按 [vulkan-tutorial.com](https://vulkan-tutorial.com) 编排：从一个空窗口开始，一步一步写到三角形、旋转的方块、贴图、维京小屋模型、Mipmap、多重采样，最后用计算着色器推动八千个粒子。

**每一步都是一个提交，都打了 tag，都能编译运行。** 看视频时画面右上角显示的 `ep03.2` 这类标签，就是这里的 tag：

```bash
git checkout ep03.2        # 切到第 3 期第 2 步
git checkout ep05          # 第 5 期结束时的代码
git diff ep03.1 ep03.2     # 这一步改了什么
```

## 编译和运行

需要：C++17 编译器、CMake ≥ 3.20、Vulkan SDK（头文件、加载器、验证层、glslc）、GLFW、GLM。

| 系统 | 安装 |
|---|---|
| Windows | [LunarG Vulkan SDK](https://vulkan.lunarg.com/)；GLFW、GLM 可以用 vcpkg 安装 |
| Ubuntu 24.04 | `sudo apt install build-essential cmake vulkan-tools libvulkan-dev vulkan-validationlayers spirv-tools glslc libglfw3-dev libglm-dev`（没有独立显卡驱动时再装 `mesa-vulkan-drivers`，用 CPU 软件渲染 lavapipe） |
| macOS | `brew install molten-vk vulkan-loader vulkan-headers vulkan-validationlayers shaderc glfw glm cmake`（或 LunarG SDK） |

```bash
cmake -S . -B build
cmake --build build
cd build && ./vulkan_app       # 第 12 期另有 ./particles
```

程序按相对路径读取 `shaders/`、`textures/`、`models/`（CMake 会把它们放进 `build/`），所以请在 `build/` 目录里运行。默认是 Debug 编译，验证层打开；Release 编译（`-DCMAKE_BUILD_TYPE=Release`）会关掉验证层。

**测试过的平台：** macOS（Apple M4 + MoltenVK，视频就是在这上面录的）；Ubuntu 24.04（容器里用 Mesa lavapipe，全部 tag 编译运行通过）。Windows 未实测。

**macOS：** 苹果系统通过 MoltenVK 运行 Vulkan，代码里用 `#ifdef __APPLE__` 包住的几行（`VK_KHR_portability_enumeration`、`VK_KHR_get_physical_device_properties2`、`VK_KHR_portability_subset`）就是为它准备的；在 Windows / Linux 上这些行不参与编译。

## 分期与 tag

| 期 | 内容 | tag |
|---|---|---|
| 1 | 开张大吉：窗口、实例与验证层 | ep01.1 – ep01.4 |
| 2 | 找到显卡：物理设备、队列族与逻辑设备、窗口表面 | ep02.1 – ep02.4 |
| 3 | 一叠盘子：交换链与图像视图 | ep03.1 – ep03.5 |
| 4 | 搭流水线：着色器与图形管线 | ep04.1 – ep04.5 |
| 5 | 第一个三角形：帧缓冲、命令与同步 | ep05.1 – ep05.4 |
| 6 | 稳稳地跑：多帧并行与重建交换链 | ep06.1 – ep06.2 |
| 7 | 自己送顶点：顶点缓冲、暂存缓冲与索引缓冲 | ep07.1 – ep07.4 |
| 8 | 转起来：Uniform 缓冲与描述符 | ep08.1 – ep08.2 |
| 9 | 贴上图片：纹理与采样器 | ep09.1 – ep09.3 |
| 10 | 走进三维：深度缓冲与模型加载 | ep10.1 – ep10.4 |
| 11 | 更精致：Mipmap 与多重采样 | ep11.1 – ep11.2 |
| 12 | 不只画画：计算着色器粒子（`src/particles.cpp`） | ep12.1 – ep12.3 |

`git log --oneline` 可以看到每一步的说明。

### 故意留下的"翻车"

有三个 tag 运行时验证层会报错，这是视频里"翻车现场"的一部分，下一步就修好：

- `ep05.3`：按官方写法每帧一个"渲染完成"信号量，新版验证层报 `VUID-vkQueueSubmit-pSignalSemaphores-00067`；`ep05.4` 改成每张交换链图像一个。
- `ep07.1`：管线描述了顶点输入，但还没有绑定顶点缓冲（`VUID-vkCmdDraw-None-04007`）；`ep07.2` 补上。
- `ep08.1`：管线用到了描述符集，但还没有绑定（`ep08.2` 补上）。

`ep10.1` 的画面前后顺序是错的（远处的矩形盖住近处的），`ep10.2` 加上深度缓冲后修好。

## 和官方代码的差异

1. macOS 可移植性扩展（见上）。
2. "渲染完成"信号量按交换链图像分配，而不是按帧（见 `ep05.3` / `ep05.4`）。官方代码在新版验证层下会报上面的错误。
3. 每一步多打印几行信息（显卡名、队列族、交换链参数、模型顶点数等），方便和视频对照。
4. 粒子初值用固定的随机种子，每次运行都一样。

## 许可

本仓库自己写的代码以 [CC0 1.0](LICENSE) 放弃版权，随便用，不用署名（与官方教程的代码清单一致）。下面列出的第三方文件保留各自的许可。

## 第三方内容

- `third_party/stb_image.h`：stb_image v2.30，公有领域 / MIT（文件内说明）。
- `third_party/tiny_obj_loader.h`：tinyobjloader，MIT。
- `models/viking_room.obj`、`textures/viking_room.png`：[Viking room](https://sketchfab.com/3d-models/viking-room-a49f1b8e4f5c4ecf9e1fe7d81915ad38) by nigelgoh，CC BY 4.0（官方教程调整过尺寸和朝向的版本）。
- `textures/texture.jpg`：官方教程使用的 CC0 图片（pixabay）。
- 程序结构和大部分代码来自官方教程的代码清单（CC0 1.0）。
