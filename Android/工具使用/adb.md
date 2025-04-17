## 一、设备连接与管理 ​​

| 命令                     | 说明                                                     |
| ------------------------ | -------------------------------------------------------- |
| adb devices              | 列出已连接的设备（含序列号和状态）                       |
| adb connect <IP:端口>    | 通过 TCP/IP 连接设备（需先 USB 连接执行 adb tcpip 5555） |
| adb disconnect <IP:端口> | 断开 TCP/IP 设备连接                                     |
| adb kill-server          | 终止 ADB 服务进程（解决连接问题）                        |
| adb start-server         | 启动 ADB 服务进程                                        |
| adb -s                   | <设备序列号> <命令> 指定设备执行命令（多设备时必需）     |

## 二、应用管理 ​

| 命令                                         | 说明                                                                          |
| -------------------------------------------- | ----------------------------------------------------------------------------- |
| adb install -r <APK 路径>                    | 安装应用（-r 替换现有应用，-t 允许测试包）                                    |
| adb uninstall <包名>                         | 卸载应用                                                                      |
| adb shell pm list packages                   | 列出所有已安装应用的包名                                                      |
| adb shell pm path <包名>                     | 查看应用 APK 路径                                                             |
| adb shell am start -n <包名>/<Activity 类名> | 启动应用的 Activity                                                           |
| adb shell am force-stop <包名>               | 强制停止应用                                                                  |
| adb shell am broadcast -a <广播 Action>      | 发送广播（如 adb shell am broadcast -a android.intent.action.BOOT_COMPLETED） |

## 三、文件操作 ​

| 命令                           | 说明                   |
| ------------------------------ | ---------------------- |
| adb push <本地路径> <设备路径> | 将文件从电脑推送到设备 |
| adb pull <设备路径> <本地路径> | 将文件从设备拉取到电脑 |
| adb shell ls <路径>            | 查看设备目录内容       |
| adb shell rm <文件路径>        | 删除设备文件           |
| adb shell mkdir <目录路径>     | 创建目录               |

## 四、日志与调试

| 命令                       | 说明                                                    |
| -------------------------- | ------------------------------------------------------- |
| adb logcat                 | 查看实时日志                                            |
| adb logcat                 | -v time > log.txt 将日志输出到文件（含时间戳）          |
| adb logcat -s <TAG>        | 按标签过滤日志（如 adb logcat -s ActivityManager）      |
| adb logcat \*:E            | 仅显示错误（Error）级别日志                             |
| adb shell dumpsys <服务名> | 查看系统服务信息（如 dumpsys meminfo、dumpsys battery） |
| adb shell top -n 1         | 查看当前进程 CPU 和内存占用                             |

## 五、屏幕与输入

| 命令                                      | 说明                                                        |
| ----------------------------------------- | ----------------------------------------------------------- |
| adb shell screencap -p                    | <路径> 截屏（如 adb shell screencap -p /sdcard/screen.png） |
| adb shell screenrecord                    | <路径> 录屏（默认 3 分钟，--time-limit 10 设置秒数）        |
| adb shell input tap <x> <y>               | 模拟点击屏幕坐标                                            |
| adb shell input swipe <x1> <y1> <x2> <y2> | 模拟滑动操作                                                |
| adb shell input text "text"               | 输入文本                                                    |

## 六、其他

| 命令                                     | 说明                                                                        |
| ---------------------------------------- | --------------------------------------------------------------------------- |
| adb shell netstat                        | 查看网络连接状态                                                            |
| adb forward tcp:<PC 端口> tcp:<设备端口> | 端口转发（如 adb forward tcp:8080 tcp:80）                                  |
| adb shell ping <IP>                      | 测试网络连通性                                                              |
| adb shell ifconfig                       | 查看网络接口信息                                                            |
| adb shell getprop                        | 查看所有系统属性                                                            |
| adb shell wm size                        | 查看屏幕分辨率                                                              |
| adb shell settings get global <键>       | 读取系统设置（如 settings get global airplane_mode_on）                     |
| adb shell settings put global <键> <值>  | 修改系统设置（需系统权限）                                                  |
| adb shell run-as <包名>                  | 访问应用私有目录（如 run-as com.example.app ls /data/data/com.example.app） |
| adb backup -f backup.ab <包名>           | 备份应用数据（需用户确认）                                                  |
| adb restore backup.ab                    | 恢复应用数据                                                                |
| adb shell monkey -p <包名> -v 500        | 对应用执行 500 次随机事件（压力测试）                                       |
| adb shell su -c "<命令>"                 | 执行 Root 权限命令                                                          |
