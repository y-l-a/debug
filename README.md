# debug——针对内存越界访问，内存爆炸，段错误等
## gdb debug 
只能找到错误在哪个函数内，不太准确
```
1.在cmakelists中添加编译选项-g set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3 -g -march=native") # set(CMAKE_CXX_FLAGS "-g")
2.修改g++生成报错core文件的大小为unlimited **ulimit -c unlimited**（仅在当前终端取消了限制，换终端就重新）
3.查看core路径是否显示core **cat /proc/sys/kernel/core_pattern**
4.关闭错误报告 sudo service apport stop
5.运行可执行文件，若存在段错误，就会在build文件夹下生成core文件
6.gdb ./可执行文件 core
7.core-file core
8.where
```

## vscode debug
首先要在cmakelists中设置模式为debug模式，debug模式影响性能，调试结束后关闭
**注：若debug时，断点为灰色，且断点处显示module containing this breakpoint has not yet loaded 
#or thebreakpoint address could not be obtained 是因为没有编译成debug版本 要**
```
SET(CMAKE_BUILD_TYPE Debug)
add_definitions("-Wall -g")
```
**并且有时候若开始调试时，忘了设置这两个，则再继续调试，可能cmakelists的改动不会立即生效，可以重新编译一下，再debug**
```
set(CMAKE_BUILD_TYPE DEBUG)
```
可以单步慢慢调试，真实用，三大json文件
- c_cpp_properties.json：负责配置代码智能补齐，报错；通常第三方库没有代码补齐，需要配置c_cpp_properties
  - 生成：ctrl+shift+p 输入edit configurations 即可打开
```

{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                **"/usr/include/**" **// 添加这句话，可以有第三方库的代码提示
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "c17",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```
- task.json:是把cpp文件编译成了一个可执行文件
  - label：与launch.json的preLaunchTask要对应，即编译生成的可执行文件的标签
  - command：选择cmake就好！，args空着就行;args是跟在command后面的，必须要空着，不然就每步都写正确
    - cd ./build ;cmake .. ;make;这样可以在build文件夹下编译，否则中间文件都在主文件夹了很乱
    - cmake 好像也不乱
  - type：选择shell
  - 生成：点击菜单栏的终端，再选择配置任务就可(重点！！！！点终端之前，一定要先选中一个cpp文件！！，否则无法生成task)，再选择C/C++：g++生成活动文件即可
```
1.
{
    //这几个综合来说就是把一个.cpp文件编译成了一个可执行文件 .exe
    "tasks": [
        {
            "type": "shell",
            "label": "dd", //对应launch.json中的 "preLaunchTask"；
                                              //（一定要一致，决定了launch.json之前先运行哪个配置，tasks是一个array类型，里面理论来说可以存多个配置，即多个label）
            "command": "cd ./build ;cmake .. ;make",//编译器的命令，相当于选择了哪个编译器 通过cmake编译
            "args": [ //跟在编译器命令后面 command后面 因为cmake不需要其他命令，所以直接cmake，因为和cmakelists在同一文件夹下
            // "-fdiagnostics-color=always",
                // "-g",
                // "${workspaceFolder}/*.cc",
                // "-o",
                // "${fileDirname}/${fileBasenameNoExtension}"
            ],
        }
    ],
    "version": "2.0.0"
}
2.
{
    //这几个综合来说就是把一个.cpp文件编译成了一个可执行文件 .exe
    "tasks": [
        {
            "type": "shell",
            "label": "dd", //对应launch.json中的 "preLaunchTask"；
                                              //（一定要一致，决定了launch.json之前先运行哪个配置，tasks是一个array类型，里面理论来说可以存多个配置，即多个label）
            "command": "cmake",//编译器的命令，相当于选择了哪个编译器 通过cmake编译
            "args": [ //跟在编译器命令后面 command后面 因为cmake不需要其他命令，所以直接cmake，因为和cmakelists在同一文件夹下
                // "-fdiagnostics-color=always",
                // "-g",
                // "${workspaceFolder}/*.cc",
                // "-o",
                // "${fileDirname}/${fileBasenameNoExtension}"
            ],
        }
    ],
    "version": "2.0.0"
}
```

- launch.json:负责debug的文件，需要设置program为待调试的可执行文件，且需要添加"preLaunchTask":与task的label相对应
  - preLaunchTask：由task编译生成的可执行文件名，这个名字必须与task.json的label一致
  - program：要调试的可执行文件名
  - 生成：launch.json在左侧菜单栏选择运行与调试，生成launch.json文件即可（同样，点菜单的运行与调试之前，一定要先选中一个cpp文件！！，否则无法生成launch）,再在配置中选择c++ /g++
```
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [

        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
           ** "program": "${workspaceFolder}/build/binaryKMeans"**, //要调试的可执行文件名
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
           ** "preLaunchTask": "dd"** //意思就是在launch之前运行的任务名，这个名字必须与task.json的label一致
        }
    ]
}
```

### 针对ros launch文件的debug
- task.json: 生成：点击菜单栏的终端，再选择配置任务就可(重点！！！！点终端之前，一定要先选中一个cpp文件！！，否则无法生成task)，再选择catkin_make:build生成活动文件即可
- launch.json:launch.json在左侧菜单栏选择运行与调试，生成launch.json文件即可（同样，点菜单的运行与调试之前，一定要先选中一个cpp文件！！，否则无法生成launch），再在配置中搜索选择ros launch(attach是单个节点的调试)

launch.json
```
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "ROS: Launch",
            "type": "ros",
            "request": "launch",
            "target": "/home/yla/dongfeng/ground/src/efficient_online_segmentation/launch/efficient_online_segmentation.launch"
        }

    ]
}
```
task.json
```
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "catkin_make",
			"args": [
				// "--directory",
				// "/home/yla/dongfeng/ground",
				// "-DCMAKE_BUILD_TYPE=RelWithDebInfo"
			],
			"problemMatcher": [
				"$catkin-gcc"
			],
			"group": "build",
			"label": "catkin_make: build"
		}
	]
}

```
