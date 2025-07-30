# Ansible Spring Boot

## 概述

本项目是一个基于 [Ansible](https://www.ansible.com/) 的Spring Boot多服务多实例部署与运行维护工具，专为简化 Spring Boot 应用程序的生命周期管理而设计。通过 Playbook 与 Roles 的模块化结构，项目实现了对 Spring Boot 应用的安装、启动、停止、重启、更新、状态检查和卸载等操作的全自动化。此外，项目还提供了环境检测、参数验证、资源清理等功能，确保部署过程的安全性与可靠性。

该项目适用于云平台的底层运维工具，亦适用于 DevOps 流程中的持续集成/持续部署（CI/CD）环节，能够显著提升部署效率并降低人为错误的风险。

---

## 适配系统

本项目适配主流 Linux 发行版，包括但不限于：

- **Red Hat 系列**：如 CentOS、RHEL、Rocky Linux等
- **Debian 系列**：如 Ubuntu、Debian等
- **信创OS**：如 KylinV10、KylinV4等

由于 Ansible 是基于 SSH 协议进行通信的无代理架构，因此无需在目标主机上安装额外客户端软件即可完成远程操作。

## 使用指南

### 执行程序 `xctl`

`xctl` 是本项目的统一入口脚本程序，用于封装项目中复杂的Ansible 操作命令，使用户可以通过简洁的命令完成部署任务。

#### 基本命令格式

```shell
./xctl <action> -i <inventory_file> -c <config_file>
```

#### 支持的 action

| Action        | 功能说明               |
| :------------ | :--------------------- |
| `install`     | 安装应用               |
| `start`       | 启动 Spring Boot 应用  |
| `stop`        | 停止应用进程           |
| `restart`     | 重启服务               |
| `update`      | 更新应用版本           |
| `statusCheck` | 检查服务是否运行正常   |
| `uninstall`   | 卸载应用及清理残留文件 |

#### 参数说明

- `-i`：指定 Inventory 文件，可以是静态文件或动态脚本。
- `-c`：指定全局配置文件，用于定义应用参数和路径。

> 注意：`xctl`脚本程序依赖 Ansible 环境，请确保已正确安装 Ansible。

### 全局配置文件config.yml

项目的参考全局配置文件位于 `example/config.yml`，你可以将其复制到任意位置作为主配置文件使用。

你可以根据实际部署环境修改这些配置项，以适应不同的服务器需求。

```yml
# 环境配置
env:
  basePath: /opt/.heluox
  java:
    version: "17"
    home: "/usr/lib/jvm/java-17-openjdk-17.0.15.0.6-2.el9.aarch64"

# 服务元数据
service:
  name: "heluox-agent"
  displayName: "heluox主机代理服务"
  version: "v1.0"
  # 占用端口
  port:
    - 8401/TCP
    - 18401/TCP
  param:
    jvmOpts: "-Xms2048m -Xmx2048m -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=18401"
    mainArgs: ""
  path:
    repoFile: "/opt/.heluox/repository/java/heluox-agent/v2/heluox-agent-2.0.0.jar"
    installDir: ""
    logDir: ""
  healthCheck:
    httpGet:
      port: 8401
      path: "/test"
      scheme: "HTTP"
      bizStatus:
        statusKey: "code"
        successCode: "200"
    timeoutSeconds: 30                 # 超时时间
    periodSeconds: 5                   # 检测间隔
    successThreshold: 1                # 检查成功为 1 次表示就绪
    failureThreshold: 2                # 检测失败 2 次表示未就绪
```

## 单元测试

### 执行程序 `uctl`

`uctl` 是本项目的role单元测试脚本程序，可以任意组合与执行一个或多个role，使程序debug更加方便快捷。

#### 基本命令格式

```shell
# 下面roles是由多个以逗号分隔role名，eg: check_param,check_env
./xctl <roles> -i <inventory_file> -c <config_file>
```

#### 支持的 roles

见源码roles目录