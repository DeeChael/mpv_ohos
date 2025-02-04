# mpv on OpenHarmony
## 引言
这个仓库记录了我（DeeChael）从开始（2025.01.25）到结束（2025.02.04）一共 11 天的时间，尝试在 [OpenHarmony](https://www.openharmony.cn) 平台上编译 [mpv](https://github.com/mpv-player/mpv) 的过程。 \
当然，这个结果很明显，是失败。 \
不过我觉得通过我的编译过程，能为各位想尝试编译的大佬们提供一些思路。 \
从基础库到 [mpv](https://github.com/mpv-player/mpv) 编译的脚本我也在编写了，不过要保证成功实在比较困难，所以需要花一点时间。

## 步骤
**本环节所有工作都默认在 [Ubuntu](https://ubuntu.com) 系统环境下进行** \
**本环节所有编译产物都只讨论 `arm64-v8a` 架构的产物** \
**请确保过程中你始终拥有访问国际互联网的能力**

### 1. 环境搭建
我是建议在 [docker](https://www.docker.com) 中进行所有的操作的，将 [Lycium](https://gitee.com/han_jin_fei/lycium) 仓库拉取下来后，在 `docker` 文件夹里按照 README 的指引创建 [docker](https://www.docker.com) 容器即可。 \
不过请注意，里面安装的 [meson](https://mesonbuild.com) 默认版本是 1.1.?（我忘记版本了），比较老，进入容器后建议运行 `pip3 install --user --upgrade meson` 更新 meson 到最新的 `1.7.0` 版本。 \
其次建议将所有相关文件都放在同一个目录下挂载到这个容器中。

### 2. SDK 下载
官方在 [gitee](https://gitee.com/openharmony-sig/oh-inner-release-management/blob/master/Release-Testing-Version.md) 提供的 SDK 最后一次更新在一年前了，比较老，是 4.1。 \
我建议从 [OpenHarmony Rust 社区](https://github.com/openharmony-rs) 搭建的 [构建仓库](https://github.com/openharmony-rs/ohos-sdk/releases) 中获取最新的 5.0 版本的 SDK。 \
如第一步中所说，建议放在一个特定的目录中挂载到容器中，同时建议在启动容器时添加 `-e OHOS_SDK=/path/to/sdk` 以在启动时自动将 SDK 路径添加到环境变量中，`/path/to/sdk` 应该为挂在后在 容器里的位置，请注意。

### 3. C & C++ 库列表拉取
我建议将 [OpenHarmony-SIG/tpc_c_cplusplus](https://gitee.com/openharmony-sig/tpc_c_cplusplus) 整个仓库拉取下来放在一个合适的位置，因为里面很多库都是我们需要的。

### 4. 编译 libass
[libass](https://github.com/libass/libass) 是最好编译的库，同时 [OpenHarmony-SIG/tpc_c_cplusplus](https://gitee.com/openharmony-sig/tpc_c_cplusplus) 中也已经做了自动化编译脚本，将 [OpenHarmony-SIG/tpc_c_cplusplus](https://gitee.com/openharmony-sig/tpc_c_cplusplus) 拉取下来后，进入里面的 lycium 目录，直接在目录里执行 `./build.sh libass`。 \
编译完成后，在当前目录（`lycium`）下的 `usr` 目录中就可以找到编译好的 libass 及其依赖库。

### 5. 编译 FFmpeg
[FFmpeg](https://git.ffmpeg.org/ffmpeg.git) 编译比较麻烦，而且如果需要更多的格式支持或是功能支持，还需要编译更多的前置库，由于我没有编译成功 [FFmpeg](https://git.ffmpeg.org/ffmpeg.git)，这方面无法提供过多建议。 \
不过如果你不想编译 [FFmpeg](https://git.ffmpeg.org/ffmpeg.git)，可以在 [changsanjiang/ffmpeg_harmony_os](https://gitee.com/changsanjiang/ffmpeg_harmony_os/tree/main/ffmpeg/src/main/cpp/thirdparty) 仓库中找到编译好的 FFmpeg 及其依赖库。

### 6. 编译 libplacebo
[libplacebo](https://code.videolan.org/videolan/libplacebo) 由于没有人在 [OpenHarmony-SIG/tpc_c_cplusplus](https://gitee.com/openharmony-sig/tpc_c_cplusplus) 中提交，所以其交叉编译中出现的问题需要我们自己解决。
#### a. 将 libplacebo 拉取到本地
将 [libplacebo](https://code.videolan.org/videolan/libplacebo) 拉取到本地准备进行编译操作，拉取后不需要执行更新 submodule 的操作，我们直接将 submodule 中两个包含头文件的仓库手动拉取下来就可以。 \
在 `3rdparty` 目录里执行 `git clone https://github.com/KhronosGroup/Vulkan-Headers` 及 `git clone https://github.com/fastfloat/fast_float` \
执行 `pip3 install jinja2` 以安装 jinja2 环境，编译环境需要 jinja2 才能正常进行。
#### b. 修改 meson.build 以保证正常编译
请直接参考本仓库 `libplacebo` 目录下的 `src/meson.build.patch` 进行修改。
#### c. 准备交叉编译文件
建议直接使用本仓库 `libplacebo` 目录下的 `arm64-v8a-cross-file.txt`，并对其中的目录位置按照在容器中的绝对位置进行相应的修改，然后放置在拉取下来的项目目录中。 \
其中你可以注意到有些目录是指向 SDK 中的一些目录的，是为了导入 SDK 中在 [OpenHarmony](https://www.openharmony.cn) 上的标准库及 [Vulkan](https://www.vulkan.org) 库。
#### d. 执行指令进行编译及安装
现在项目目录中创建 `arm64-v8a-build` 目录来存放编译产物。 

执行 `meson arm64-v8a-build --cross-file arm64-v8a-cross-file.txt --prefix=/path/to/tpc_c_cplusplus/lycium/usr/libplacebo/arm64-v8a -Dvulkan=enabled -Dopengl=disabled -Dgl-proc-addr=disabled -Dd3d11=disabled -Dglslang=disabled -Ddovi=disabled -Dlibdovi=disabled -Ddemos=false -Dxxhash=disabled > arm64-v8a-build/build.log 2>&1`（别忘记修改 `--prefix` 参数目录地址），然后检查 `arm64-v8a-build` 目录下是否出现 `build.ninja` 文件，如果存在，说明这一步成功。 

继续执行 `ninja -v -C arm64-v8a-build >> arm64-v8a-build/build.log 2>&1` 开始编译，执行完成后检查`arm64-v8a-build` 目录下的 `build.log` 日志文件，编译过程中会有 `[xx/yy]` 的格式文字在每一行开头，其中 `xx` 指的是执行的步骤，`yy` 指的是总共的步骤，确保日志文件中存在一行 `xx` 和 `yy` 相等，即所有步骤都执行完成，则当前步骤执行成功。

执行 `ninja -v -C arm64-v8a-build install >> arm64-v8a-build/build.log 2>&1` 后，编译好的产物就会被自动安装到 `--prefix` 指定的地址，强烈建议 `--prefix` 设置的地址跟你拉取的 [OpenHarmony-SIG/tpc_c_cplusplus](https://gitee.com/openharmony-sig/tpc_c_cplusplus) 仓库中 lycium 编译产物的输出目录一致，这样方便后期打包库。

编译 [libplacebo](https://code.videolan.org/videolan/libplacebo) 的过程中，在执行 `meson` 指令时会发现我 disable 了很多的功能，是因为其依赖的别的库也需要自行编译，这边目标为编译 [mpv](https://github.com/mpv-player/mpv)，所以暂时先不编译其他需要的库。

### 7. 编译 mpv
这一步我没有很多可参考的内容，仅提供我编写的交叉编译文件，此仓库的 `mpv` 目录下的 `arm64-v8a-cross-file.txt`。请直接拉取 [mpv](https://github.com/mpv-player/mpv) 仓库，进行编译操作的尝试。 \
执行 `meson` 指令这一步我仍然是成功了，但是在使用 `ninja` 进行编译时出现了部分错误，由于我并没有学习过 C 及 C++ 相关知识，并不了解如何解决。
我是用的 `meson` 指令及参数：
```shell
meson arm64-v8a-build --cross-file arm64-v8a-cross-file.txt --prefix=/data/ohos/tpc_c_cplusplus/lycium/usr/mpv/arm64-v8a -Dlibmpv=true -Dalsa=enabled -Dpulse=enabled -Duchardet=enabled -Dzlib=enabled -Diconv=enabled -Dvulkan=enabled > arm64-v8a-build/build.log 2>&1
```
