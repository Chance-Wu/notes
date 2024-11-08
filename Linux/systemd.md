systemd 是目前主流Linux发行版中广泛使用的系统和服务管理器。它负责在系统启动时初始化系统组件，并管理服务（称为units）。使用 systemd 管理服务的功能强大且灵活，支持**并行启动**、**按需激活**、**日志管理**等。

### 一、systemd基本概念

---

#### 1.1 Unit 文件

systemd 的配置文件被称为 **"unit 文件"**，每个服务、挂载点、设备等都对应一个 unit。systemd 支持多种 unit 类型，最常见的类型有：

- **Service unit** （`*.service`）：定义和管理服务
- **Target unit**（`*.target`）：定义系统状态或同步点
- **Socket unit**（`*.socket`）：定义套接字，用于激活相关的服务

通常，这些 unit 文件位于以下路径中：

- `/etc/systemd/system/`：用户定义的 unit 文件
- `/lib/systemd/system/`：系统自带的 unit 文件

#### 1.2 Unit文件结构

systemd 的 unit 文件结构由多个部分组成，其中最常见的是 **[Unit]**、**[Service]** 和 **[Install]** 部分。

示例 `*.service` 文件：

```ini
[Unit]
Description=My Custom Service
After=network.target

[Service]
ExecStart=/usr/bin/my-service-start-command
ExecStop=/usr/bin/my-service-stop-command
Restart=always
User=serviceuser

[Install]
WantedBy=multi-user.target
```

`[Unit]`：描述服务的元数据及其依赖项。

- `Description`：简要描述服务。
- `After`：指定服务启动顺序，比如 `network.target` 表示服务在网络服务启动后运行。

`[Service]`：定义服务的行为。

- `ExecStart`：指定启动服务时执行的命令。
- `ExecStop`：指定停止服务时执行的命令（可选）。
- `Restart`：设置服务是否自动重启（如 `always` 表示服务停止后自动重启）。
- `User`：指定运行服务的用户。

`[Install]`：定义服务如何与特定的 `target` 关联。

- `WantedBy`：指定服务在哪个 `target` 中被启用（例如 `multi-user.target`）。



### 二、服务管理命令

---

#### 2.1 启动服务

仅当前会话，系统重启后不自动启动。

```shell
sudo systemctl start <服务名>
```

#### 2.2 停止服务

```shell
sudo systemctl stop <服务名>
```

#### 2.3 重启服务

```shell
sudo systemctl restart <服务名>
```

#### 2.4 重新加载服务配置

在不停止服务的情况下重新加载服务配置。

```shell
sudo systemctl reload <服务名>
```

#### 2.5 查看服务状态

```shell
systemctl status <服务名>
```

#### 2.6 启用服务自启动

设置某个服务为开机自启动。

```shell
sudo systemctl enable <服务名>
```

#### 2.7 禁用服务自启动

```shell
sudo systemctl disable <服务名>
```

#### 2.8 查看服务是否启用自启动

```shell
systemctl is-enabled <服务名>
```

#### 2.9 列出所有服务

包括活动和非活动的。

```shell
systemctl list-units --type=service
```



### 三、journalctl 日志管理

---

`systemd` 通过 `journal` 进行日志管理。`journalctl` 是查看和管理 `systemd` 日志的工具。

#### 3.1 查看服务日志

```shell
journalctl -u <服务名>
```

#### 3.2 实时查看服务日志

```shell
journalctl -u <服务名> -f
```

#### 3.3 查看开机日志

```shell
journalctl -b
```



### 四、常见systemd的依赖与目标

---

#### 4.1 依赖关系

`systemd` 允许在 `unit` 中定义依赖关系，通常用 `Requires=` 和 `After=` 指令：

- `Requires=`: 表示服务依赖的其他服务，如果依赖的服务失败，当前服务也会失败。
- `After=`: 指定当前服务应该在某些服务之后启动（仅影响启动顺序，不表示强依赖）。

#### 4.2 目标（Targets）

目标是`systemd`中的一组`unit`，通常用于表示系统的特定状态或启动顺序。例如：

- `multi-user.target`：对应于传统的运行级别 3，多用户模式但无图形界面。
- `graphical.target`：对应于传统的运行级别 5，启用了图形界面。



### 五、调试与故障排除

---

#### 5.1 查看错误日志

如果服务无法启动，可以查看详情的日志信息。

```shell
journalctl -xe
```

#### 5.2 查看依赖关系

```shell
systemctl list-dependencies <服务名>
```
