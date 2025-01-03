# VSCode插件配置

1. 打开VSCode扩展商店，安装插件：
- Chinese (Simplified) (简体中文) Language Pack for Visual Studio Code - VSCode汉化
- C/C++ - C/C++ 插件
- clangd - 代码补全提示
- CMake Tools - CMake工具
- vscode-icons 美化目录
- Typora
2. `按Ctrl + ,`进入设置
- 找到扩展 -> C/C++ -> IntelliSense -> C_Cpp: AutoComplete，改为disable



# Vscode配置clangd

文章链接：[Vscode配置clangd](https://blog.csdn.net/xzq1207105685/article/details/126091947?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522b024de59c0ac36f2d937717555935a77%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=b024de59c0ac36f2d937717555935a77&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-8-126091947-null-null.142^v101^pc_search_result_base9&utm_term=vscode%E9%85%8D%E7%BD%AEclangd&spm=1018.2226.3001.4187)

## 下载clangd

```sh
sudo apt-get update
sudo apt-get install clangd
```



## 修改基础配置

在根目录下建立`.vscode`文件夹，创建`settings.json`文件。如下图

<img src="docs\Vscode配置\1.png">

`settings.json`文件内容如下

```json
{
  "files.associations": {
    "iostream": "cpp",
    "intrinsics.h": "c",
    "ostream": "cpp",
    "vector": "cpp"
  },
  // 开启粘贴保存自动格式化
  "editor.formatOnPaste": true,
  "editor.formatOnSave": true,
  "editor.formatOnType": true,
  "C_Cpp.errorSquiggles": "Disabled",
  "C_Cpp.intelliSenseEngineFallback": "Disabled",
  "C_Cpp.intelliSenseEngine": "Disabled",
  "C_Cpp.autocomplete": "Disabled", // So you don't get autocomplete from both extensions.
  "clangd.path": "/usr/bin/clangd",
  // Clangd 运行参数(在终端/命令行输入 clangd --help-list-hidden 可查看更多)
  "clangd.arguments": [
    // 让 Clangd 生成更详细的日志
    "--log=verbose",
    // 输出的 JSON 文件更美观
    "--pretty",
    // 全局补全(输入时弹出的建议将会提供 CMakeLists.txt 里配置的所有文件中可能的符号，会自动补充头文件)
    "--all-scopes-completion",
    // 建议风格：打包(重载函数只会给出一个建议）
    // 相反可以设置为detailed
    "--completion-style=bundled",
    // 跨文件重命名变量
    "--cross-file-rename",
    // 允许补充头文件
    "--header-insertion=iwyu",
    // 输入建议中，已包含头文件的项与还未包含头文件的项会以圆点加以区分
    "--header-insertion-decorators",
    // 在后台自动分析文件(基于 complie_commands，我们用CMake生成)
    "--background-index",
    // 启用 Clang-Tidy 以提供「静态检查」
    "--clang-tidy",
    // Clang-Tidy 静态检查的参数，指出按照哪些规则进行静态检查，详情见「与按照官方文档配置好的 VSCode 相比拥有的优势」
    // 参数后部分的*表示通配符
    // 在参数前加入-，如-modernize-use-trailing-return-type，将会禁用某一规则
    "--clang-tidy-checks=cppcoreguidelines-*,performance-*,bugprone-*,portability-*,modernize-*,google-*",
    // 默认格式化风格: 谷歌开源项目代码指南
    // "--fallback-style=file",
    // 同时开启的任务数量
    "-j=2",
    // pch优化的位置(memory 或 disk，选择memory会增加内存开销，但会提升性能) 推荐在板子上使用disk
    "--pch-storage=disk",
    // 启用这项时，补全函数时，将会给参数提供占位符，键入后按 Tab 可以切换到下一占位符，乃至函数末
    // 我选择禁用
    "--function-arg-placeholders=false",
    // compelie_commands.json 文件的目录位置(相对于工作区，由于 CMake 生成的该文件默认在 build 文件夹中，故设置为 build)
    "--compile-commands-dir=build"
  ],
}

```



## 生成compile_commands.json文件

确保项目能够正常编译的前提下

```sh
cd build
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 .. -G 'Unix Makefiles'
```

之后在build目录下就会生成对应的compile_commands.json,格式如下。**请务必确保存在compile_commands.json文件，这是clangd补全依赖文件，否则会失效**。

<img src="docs\Vscode配置\2.png">

重启Vscode即可



