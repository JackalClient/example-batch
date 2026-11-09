# Batch Starter · Jackal 批处理入门扩展

用顺序执行的 Jackal 命令为客户端添加模块。本示例启用时显示配置的欢迎语，等待 300 毫秒，再读取 FPS 并显示通知；停用时显示结束提示。

[扩展与 SDK 完整教程](https://doc.wormwake.com/extension-sdk.html)

## 1. 安装和体验

1. 准备支持批处理扩展的 JackalClient，下载本示例文件夹或 ZIP。
2. 打开 **扩展 (Extensions) → 导入本地 (Import local...)**，选择本文件夹或 ZIP。
3. 在 **其他 (Misc)** 分类开启 **批处理入门示例**。
4. 修改 **批处理欢迎语 (Batch Greeting)**，重新启用模块查看新的欢迎语。

这是客户端命令脚本，文件扩展名为 `.jbat`，运行方式是在 JackalClient 内启用模块。普通 Windows `.bat` 使用另一套命令语法。

## 2. 文件说明

| 文件 | 用途 |
| --- | --- |
| `extension.json` | 扩展信息、默认配置、脚本行为和文件清单 |
| `enable.jbat` | 启用模块时执行的命令 |
| `disable.jbat` | 停用模块时执行的命令 |
| `Dictionary.txt` | 本扩展的中英文词条 |

批处理运行时和客户端接口由 JackalClient 提供，适用于 x86、x64 客户端。

开发时可提前下载 `JackalExtensionSDK`，与 `example-batch` 放在同一父目录，参考 `../JackalExtensionSDK/README.md`；批处理通过 `/api` 使用客户端内置接口。

## 3. 修改这个示例

打开 `enable.jbat`：

```text
/api {"method":"notify","text":{"$ref":"/config/Batch Greeting"},"level":"success"}
/sleep 300
/api {"method":"variable.get","name":"fps"}
/api {"method":"notify","text":{"$ref":"/last"},"title":"FPS"}
```

每行按顺序执行。普通行使用客户端命令语法，可省略开头的 `/`；空行和 `#` 开头的注释会跳过。扩展行为脚本还提供：

| 指令 | 用途 |
| --- | --- |
| `/api JSON` | 调用客户端接口，等待结果并保存到 `/last` |
| `/let name JSON` | 将值保存在 `/vars/name`，供后续指令引用 |
| `/sleep ms`、`/sleepms ms` | 等待指定毫秒 |
| `/sleeps seconds`、`/sleepsec seconds` | 等待指定秒数 |

`{"$ref":"/config/Batch Greeting"}` 读取配置；`{"$ref":"/last"}` 使用上一条 API 的结果；引用保留原值类型。可使用 `/context` 读取当前扩展与模块信息。例如，先用 `/let fps {"$ref":"/last"}` 保存 FPS，稍后通过 `{"$ref":"/vars/fps"}` 使用。

创建自己的扩展时，修改清单的 `id`、名称、作者与模块 `name`。清单使用 `schemaVersion: 2`、`script.language: "jbat"`；`behaviors` 的值是 `files` 中的文件逻辑键。增加周期任务时，登记新的 `.jbat` 文件，并添加 `update` 行为与 `pollIntervalMs`。新增配置写入 `defaults`，翻译放入 `Dictionary.txt` 并通过 `contributes.Dictionary` 引用。

完整 API 和更多示例见官网教程；开发方式概览见同级 `../JackalExtensionSDK/README.md`。

## 4. 调试和发布

首次导入即可开启模块。修改脚本后，删除已安装扩展并重启客户端，再导入修改后的目录。出错时通知会标出脚本文件和行号；停用模块可取消等待中的动作。模块设置保存在当前配置档案。

本示例源码清单省略文件校验值，适合本地导入。发布前，为 `files` 中每个文件填写实际 `size`、`sha256`；修改文件后重新计算校验值。例如：

```powershell
(Get-Item -LiteralPath ./enable.jbat).Length
(Get-FileHash -LiteralPath ./enable.jbat -Algorithm SHA256).Hash.ToLowerInvariant()
```

发布包根目录放入 `extension.json`、`enable.jbat`、`disable.jbat` 和 `Dictionary.txt`。SDK 和 README 可随开发仓库保留。

如果使用完整客户端源码仓库，可在仓库根目录自动打包：

```powershell
py -3 scripts/extension-package.py publish --id example-batch --destination temp/script-preview
```

导入生成的 `temp/script-preview/example-batch` 即可测试发布包。独立示例仓库可按同级 `../JackalExtensionSDK/README.md` 中的发布说明，或[官网教程](https://doc.wormwake.com/extension-sdk.html)准备清单。
