# Ansible Role: Slurm集群部署

这个Ansible role用于部署完整的Slurm集群，包括控制节点、计算节点、slurmdbd数据库节点以及其他必要组件。

## 角色组件

本角色包含以下主要组件的部署和配置：

- **控制节点(slurmctld)**: 负责作业调度和资源分配
- **计算节点(slurmd)**: 执行作业的工作节点
- **数据库节点(slurmdbd)**: 存储作业历史和账户信息
- **Munge认证**: 用于节点间安全通信
- **时间同步**: 基于chrony的时间同步设置
- **环境配置**: Slurm环境变量设置

## 目录结构

```
./
├── defaults/           # 默认变量
│   └── main.yml        # 主要配置变量
├── files/              # 离线安装包
│   ├── slurm-24.11.4.tar.gz
│   ├── pmix-3.2.5.tar.gz
│   ├── pmix-4.2.9.tar.gz
│   └── pmix-5.0.7.tar.gz
├── handlers/           # 处理服务重启等通知
├── tasks/              # 主要任务文件
│   ├── main.yml        # 主任务入口
│   ├── common.yml      # 所有节点通用配置
│   ├── controller.yml  # 控制节点配置
│   ├── compute.yml     # 计算节点配置
│   ├── database.yml    # 数据库节点配置
│   ├── munge.yml       # Munge认证配置
│   ├── environment.yml # 环境变量配置
│   └── time_sync.yml   # 时间同步配置
└── templates/          # Jinja2模板
    ├── slurm.conf.j2           # Slurm主配置
    ├── slurmdbd.conf.j2        # SlurmdDB配置
    ├── cgroup.conf.j2          # Cgroup配置
    ├── slurmdbd.service.j2     # SlurmdDB服务
    ├── slurmctld.service.j2    # Slurmctld服务
    ├── slurmd.service.j2       # Slurmd服务
    ├── slurm.sh.j2             # Slurm环境变量脚本
    ├── chrony_master.conf.j2   # 主节点时间同步配置
    └── chrony_node.conf.j2     # 计算节点时间同步配置
```

## 使用方法

1. 直接使用此角色
2. 确保所有需要的安装包已放入`files/`目录

### Playbook示例

```yaml
- hosts: slurm_cluster
  vars:
    slurm_roles:
      - "{{ 'master' if inventory_hostname == 'master' else 'node' }}"
      - "{{ 'dbd' if inventory_hostname == 'master' else '' }}"
  roles:
    - slurm
```

## 支持的角色

在`slurm_roles`变量中指定节点角色:
- `master`: 部署控制节点服务(slurmctld)
- `node`: 部署计算节点服务(slurmd)
- `dbd`: 部署数据库节点服务(slurmdbd)

## 主要配置变量

配置文件: `defaults/main.yml`

| 变量名 | 描述 | 默认值 |
|--------|------|--------|
| slurm_version | Slurm版本 | 24.11.4 |
| top_dir | 安装根目录 | /opt/hpc |
| slurm_cluster_name | 集群名称 | cluster |
| slurm_control_machine | 控制节点主机名 | master |
| slurm_compute_nodes | 计算节点配置列表 | compute[1-10] |
| slurm_partitions | 集群分区配置 | debug |
| mariadb_root_password | MariaDB根密码 | 123456 |
| slurm_db_name | Slurm数据库名称 | slurm_acct_db |
| slurm_db_user | Slurm数据库用户 | slurm |
| slurm_db_pass | Slurm数据库密码 | 12345678 |

## 功能特性

- 自动配置时间同步服务
- 自动创建和分发Munge密钥
- 自动设置Slurm环境变量
- 支持多版本PMIX库

## 前提条件

- 目标服务器已安装基本的Linux系统
- 节点间可以通过SSH互相访问
- Ansible控制节点可以访问所有目标节点
- Ansible主机清单需要配置为主机名
- 主机名已正确配置，且可以互相解析

## 注意事项

- 部署前确保已正确配置节点的主机名和IP地址
- 离线部署时确保所有依赖包已下载到`files/`目录
- 建议在测试环境验证配置后再在生产环境部署
