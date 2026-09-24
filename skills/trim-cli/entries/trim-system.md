---
name: trim-system
description: 查询静态系统信息、机器类型、系统版本、身份或执行电源操作时使用
---

# trim-system

## 什么时候看这个 skill

- 需要确认 fnOS 版本、运行环境类型、系统标识
- 要查看当前机器的型号、机器 ID、卷信息、硬件信息、电源计划等静态细节
- 要重启或关闭 NAS
- 需要梳理系统配置与固件信息供后续调查

## 先看哪里

- 系统信息 reference：`../reference/sysinfo.md`
- 电源操作 reference：`../reference/power.md`
- 需要同时查硬件、卷、电源计划等信息时逐条调用对应 `system` 子命令

## 核心提醒

- 这里只介绍静态系统信息，不包括 CPU 或 memory 运行态指标，后者由 `trim-monitor` 处理
- `system` 固定子命令均为只读查询；`system request` 是泛化入口，默认需要确认或 `--yes`
- `power reboot` 和 `power poweroff` 是高影响写操作，会中断设备服务，必须确认目标设备和授权
- `system request` 仅允许 `appcgi.sysinfo.*` 端点，JSON 中不要写 `req` 或 `reqid`
- `system request` 使用已保存 session 认证；先登录目标设备再调用配置类端点
- 机器类型、版本等字段会随 CLI 或 NAS 版本变化，优先以 `system info` 当前输出为准
- 遇到异常或字段为空，说明当前 CLI 与目标版本字段不一致，请以输出结果为可信来源

## 常用命令

```bash
trim-cli system info
trim-cli system trim-version
trim-cli system kernel-version
trim-cli system host-name
trim-cli system machine-id
trim-cli system uptime
trim-cli system all-vols
trim-cli system vols
trim-cli system docker
trim-cli system unix-time
trim-cli system boot-on-power
trim-cli system hardware
trim-cli system reserved-partition
trim-cli system is-trim-machine
trim-cli system trim-feature
trim-cli system fan-mode
trim-cli system trim-disk
trim-cli system time-setting --lang zh-CN
trim-cli system timezone-list --lang zh-CN
trim-cli system power-plan
trim-cli system next-power-off-time
trim-cli system power-plan-status
trim-cli system power-on-seconds
trim-cli system inited-timestamp
trim-cli system request appcgi.sysinfo.getTrimVersion --yes
```

## 电源操作

```bash
trim-cli power reboot --yes
trim-cli power poweroff --yes
```
