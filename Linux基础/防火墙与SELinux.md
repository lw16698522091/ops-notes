# 防火墙与 SELinux

## 适用场景

- 服务已启动、端口已监听，但外部仍然连不上

- 端口无法访问，需要确认是被防火墙拦截还是 SELinux 限制

- 新增服务端口后，需要放行防火墙规则

## 1. 防火墙状态查看
| 命令 | 作用 |
|------|------|
| systemctl status firewalld | 查看防火墙服务是否在运行；如果显示 inactive，说明防火墙没开，通常不是拦截原因 |
| firewall-cmd --state | 简化输出防火墙运行状态，直接显示 running 或 not running |

## 2. 查看当前防火墙规则
| 命令 | 作用 |
|------|------|
| firewall-cmd --list-all | 查看当前区域的所有规则，包括放行的端口、服务、网卡等；可以查看 80、8080 等端口是否已经放行 |

## 3. 放行端口
| 命令 | 作用 |
|------|------|
| firewall-cmd --add-port=80/tcp --permanent | 放行 80 端口，--permanent 表示永久生效 |
| firewall-cmd --reload | 重新加载防火墙规则，让永久配置生效 |

**注意**：执行完 --add-port 后必须 reload，否则不生效

## 4. 移除端口
| 命令 | 作用 |
|------|------|
| firewall-cmd --remove-port=8080/tcp --permanent | 移除已放行的端口 |

  执行后同样需要 firewall-cmd --reload

## 5. 端口访问异常时防火墙排查思路

当外部无法访问某个端口时，按以下步骤排查：

1\. 先确认服务在监听：ss -lntp | grep 端口号

   - 如果没有任何输出，说明服务没启动，不是防火墙问题

2\. 再确认防火墙状态：systemctl status firewalld

   - 如果防火墙没运行，可以直接排除防火墙

3\. 再查看规则：firewall-cmd --list-all

   - 如果端口没有出现在放行列表中，执行 add-port 并 reload

4\. 注意临时规则与永久规则

   - 不加 --permanent 的规则立即生效但重启后丢失

   - 加了 --permanent 的规则必须 reload 才生效

## 6. SELinux 基本概念

- SELinux 是系统内核级的安全模块，会限制进程对文件的访问

- 即使权限、防火墙都正确，SELinux 仍可能拦截访问

- getenforce

  查看当前 SELinux 状态，输出 Enforcing（强制开启）或 Disabled（关闭）

- setenforce 0

  临时将 SELinux 切换为 Permissive（允许但不阻断），仅内存生效，重启后失效

  注意：这不是根治方案，只是临时确认问题是否由 SELinux 引起

- 配置文件：/etc/selinux/config

  持久修改需要改这个文件，将 SELINUX=enforcing 改为 SELINUX=disabled

  修改后需重启系统生效

## 7. 完整排障顺序

外部访问不通时，严格按照以下顺序检查：

1\. 服务本身是否启动、端口是否监听：systemctl status 服务名、ss -lntp

2\. 防火墙是否放行：firewall-cmd --list-all

3\. SELinux 是否拦截：getenforce，临时 setenforce 0 测试

4\. 网络是否连通：ping 虚拟机 IP、宿主机访问测试

## 关联内容

- 端口与连接状态判断：01\_Linux基础/端口与进程.md

- 服务管理：01\_Linux基础/服务管理.md

## 学习说明

- 防火墙的放行端口和 reload 操作在环境搭建时实际用过，基础命令熟悉

- 不同服务使用不同端口时，能自己判断需要放行哪些端口

- SELinux 目前只理解基本概念和临时关闭方法，未掌握完整规则配置，属于后续需要深入学习的内容
