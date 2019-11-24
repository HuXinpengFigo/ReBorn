---
title: 解决在Mac OS X Catalina上VScode无法调试的问题
date: 2019-11-23 13:20:54
index_img: /img/Xnip2019-11-23_13-39-22.png
banner_img: /img/Xnip2019-11-23_13-39-22.png
tags: [MacOS,VScode,环境搭建]
categories: 代码
---


* 原因：苹果在macOS Catalina上取消了对lldb支持，于是我们就必须寻找对应的替代品

* 解决方法：VScode官方插件**`CodeLLDB`**

<!-- more -->

----
![](/img/Xnip2019-11-21_17-19-35.jpg)

  > 安装它之后更改launch.json:

  ```json
{
      // 
      // 使用 IntelliSense 了解相关属性。 
      // 悬停以查看现有属性的描述。
      // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
      "version": "0.2.0",
      "configurations": [
          {
              "name": "Launch",
              "type": "lldb",
              "request": "launch",
              "args": [
                  "-arg1",
                  "-arg2"
              ],
              "program": "${fileDirname}/a.out",
              "args": [],
              "cwd": "${workspaceFolder}",
              "stopAtEntry": true, // if true then stop at the main entry (function)
              "environment": [],
              "externalConsole": true,
              "MIMode": "lldb",
              "preLaunchTask": "build hello world"
          }
      ]
  }
  ```

  > 对应的setting.json:

  ```json
{
      "python.pythonPath": "/Users/zjx/anaconda3/bin/python3",
      "code-runner.executorMap": {
          "c": "cd $dir && gcc $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
          "cpp": "cd $dir && g++ $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
      },
      "editor.renderWhitespace": "all",
      "editor.renderLineHighlight": "all",
      "editor.formatOnSave": true,
      "code-runner.runInTerminal": true,
      "code-runner.ignoreSelection": true,
      "code-runner.enableAppInsights": false,
      "C_Cpp.updateChannel": "Insiders",
      "[makefile]": {
          "editor.insertSpaces": true
      },
      "C_Cpp.default.includePath": [
          "/usr/local/opt/opencv@3/include"
          // "/usr/local/Cellar/opencv/4.0.1/include/opencv4"
          // "/usr/local/include/opencv4"
      ]
  }
  ```

  > 对应的tasks.json

  ```json
{
      // See https://go.microsoft.com/fwlink/?LinkId=733558
      // for the documentation about the tasks.json format
      "version": "2.0.0",
      "tasks": [
          {
              "type": "shell",
              "label": "build hello world",
              "command": "g++",
              "args": [
                  "-g",
                  "${file}",//自动选当前文件
                  //"${workspaceRoot}/start.cpp", // 这里是你自己的源文件的名字
                  //"${workspaceRoot}/calc.cpp", // 这也是源文件的名字，如果你有多个的话，都列上
                  "-o",
                  "${workspaceRoot}/a.out"
              ],
              "group": {
                  "kind": "build",
                  "isDefault": true
              },
              "problemMatcher": [
                  "$gcc"
              ]
          }
      ]
  }
  ```

  > 对应的c_cpp_properties.json:

  ```json
{
      "configurations": [
          {
              "name": "Mac",
              "defines": [],
              "macFrameworkPath": [
                  "/System/Library/Frameworks",
                  "/Library/Frameworks",
                  "${workspaceFolder}/**"
              ],
              "compilerPath": "/usr/bin/g++",
              "cStandard": "c11",
              "cppStandard": "c++17",
              "intelliSenseMode": "clang-x64",
              "browse": {
                  "path": [
                      "${workspaceFolder}"
                  ],
                  "limitSymbolsToIncludedHeaders": true,
                  "databaseFilename": ""
              }
          }
      ],
      "version": 4
  }
  ```

----

##### 记住更改完之后需要 Command + B 编译

