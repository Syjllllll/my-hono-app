# 使用说明

- windows系统并且已下载安装[nvm(for windows)](https://github.com/coreybutler/nvm-windows),在项目根目录下创建.nvmrc文件，文件内容为node版本号，如："20.17.0",在项目根目录下打开终端([powershell](#powershell) 或者[git bash](#git-bash) )，即可切换到指定版本的node。
- 也可使用 [Volta](https://volta.sh/)，管理和自动切换node版本。

## PowerShell

1. **找到你的 PowerShell 配置文件**：

   - 你可以使用 `$PROFILE` 变量来找到当前用户的 PowerShell 配置文件路径。运行以下命令来查看配置文件路径：

   ```powershell copy
   echo $PROFILE
   ```

2. **创建或编辑配置文件**：

   - 如果配置文件不存在，你可以使用以下命令来创建它：

   ```powershell copy
   if (!(Test-Path -Path $PROFILE)) {
       New-Item -Type File -Path $PROFILE -Force
   }
   ```

3. **将脚本添加到配置文件中**：

   - 打开配置文件进行编辑：

   ```powershell copy
   notepad $PROFILE
   ```

   - 将如下的 `Find-Nvmrc` 函数和调用代码添加到配置文件中：

   ```powershell copy

    function Find-Nvmrc {
        param (
            [string]$dir
        )

        # 检查当前目录
        $nvmrcPath = Join-Path -Path $dir -ChildPath ".nvmrc"
        if (Test-Path $nvmrcPath) {
            # 读取 .nvmrc 文件中的版本号
            $projectNodeVersion = Select-String -Path $nvmrcPath -Pattern '"\d+(\.\d+){0,2}"' | ForEach-Object { $_.Matches.Value.Trim('"') }

            if ($projectNodeVersion) {
                # 获取当前使用的 Node.js 版本
                $currentNodeVersion = node -v
                $currentNodeVersion = $currentNodeVersion.TrimStart('v')

                # 比较大版本号
                $projectNodeMajorVersion = $projectNodeVersion.Split('.')[0]
                $currentNodeMajorVersion = $currentNodeVersion.Split('.')[0]

                if ($projectNodeMajorVersion -eq $currentNodeMajorVersion) {
                    # Write-Output "No need to switch Node.js version"
                    return
                } else {
                    Write-Output "Target node is v$projectNodeVersion"
                    Write-Output "Current node is v$currentNodeVersion"
                    Write-Output "Switching to node v$projectNodeVersion..."
                    nvm use $projectNodeVersion
                }
                return
            } else {
                Write-Output "No valid version number found in .nvmrc file"
            }
        }
    }

    Find-Nvmrc (Get-Location)
   ```

4. **保存并关闭配置文件**：

   - 保存文件并关闭编辑器。

     现在，每次你启动 PowerShell 时，配置文件中的脚本都会自动运行，并查找 `.nvmrc` 文件以切换到指定的 Node.js 版本。

## Git Bash

1. **找到你的 Git Bash 配置文件**：

   - 切换到你的主目录：

   ```bash copy
   cd $HOME
   ```

2. **创建或编辑配置文件**：

   - 如果配置文件不存在，你可以使用以下命令来创建它：

   ```bash copy
   if [ ! -f $HOME/.bashrc ]; then
       touch $HOME/.bashrc
   fi
   ```

3. **将脚本添加到配置文件中**：

   - 打开配置文件进行编辑：

   ```bash copy
    notepad $HOME/.bashrc
   ```

   - 将如下的 `find_nvmrc` 函数和调用代码添加到配置文件中：

   ```bash
    #!/bin/bash

    find_nvmrc() {
        local dir="$1"

        if [ -f "$dir/.nvmrc" ]; then
                project_node_version=$(grep -oP '"\d+(\.\d+){0,2}"' "$dir/.nvmrc" | tr -d '"')
                # echo .nvmrc file
                # cat "$dir/.nvmrc"
                if [ -n "$project_node_version" ]; then
                    nvm_ls_output=$(nvm ls)
                    current_node_version=$(echo "$nvm_ls_output" | grep -oP '(?<=\*\s)v?\d+\.\d+\.\d+' | head -1 | tr -d 'v')

                    project_node_major_version=$(echo "$project_node_version" | cut -d. -f1)
                    current_node_major_version=$(echo "$current_node_version" | cut -d. -f1)

                    if [ "$project_node_major_version" -eq "$current_node_major_version" ]; then
                        # echo "No need to switch Node.js version"
                        return
                    else
                        echo "Target node is v$project_node_version"
                        echo "Current node is v$current_node_version"
                        echo "Switching to node v$project_node_version..."
                        nvm use "$project_node_version"
                    fi
                    return
                else
                    echo "No valid version number found in .nvmrc file"
                fi
            fi
    }

    find_nvmrc "$(pwd)"
   ```

4. **保存并关闭配置文件**：

   - 保存文件并关闭编辑器，重启 Git Bash。

     现在，每次你启动 Git Bash 时，配置文件中的脚本都会自动运行，并查找 `.nvmrc` 文件以切换到指定的 Node.js 版本。
