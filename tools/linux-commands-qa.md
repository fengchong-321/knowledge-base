# Linux 命令测试工程师精通指南

> 标签: #linux #shell #运维 #面试
> 创建时间: 2026-02-26
> 来源: [Linux官方文档](https://www.kernel.org/doc/) | [Linux命令大全](https://www.runoob.com/linux/linux-command-manual.html)

## 概述

Linux 命令是测试工程师必备技能，涉及服务器部署、日志分析、性能监控、问题排查等核心工作。本文整理测试面试高频命令，按重要程度分类。

---

## 一、知识体系总览

### 掌握程度分类

| 级别 | 说明 | 面试权重 |
|------|------|----------|
| 🔴 必须掌握 | 面试必问，日常必用 | 40% |
| 🟠 重要 | 常见考点，需要熟练 | 30% |
| 🟡 常用 | 工作中频繁使用 | 20% |
| 🟢 了解 | 高级场景，知道即可 | 10% |

---

## 二、核心知识点

### 🔴 必须掌握

#### 1. 文件与目录操作

```bash
# ========== 目录导航 ==========
pwd                    # 显示当前目录
cd /path/to/dir        # 切换目录
cd ~                   # 回到 home 目录
cd ..                  # 返回上级目录
cd -                   # 返回上一次目录

# ========== 文件列表 ==========
ls                     # 列出当前目录
ls -l                  # 详细信息（权限、大小、时间）
ls -la                 # 包含隐藏文件
ls -lh                 # 人类可读大小
ls -lt                 # 按时间排序

# ========== 创建/删除 ==========
mkdir dir              # 创建目录
mkdir -p a/b/c         # 递归创建
rmdir dir              # 删除空目录
rm file                # 删除文件
rm -r dir              # 递归删除目录
rm -rf dir             # 强制递归删除（慎用！）

# ========== 复制/移动 ==========
cp file1 file2         # 复制文件
cp -r dir1 dir2        # 复制目录
mv file1 file2         # 移动/重命名
mv dir1 dir2           # 移动目录

# ========== 查看文件 ==========
cat file               # 显示全部内容
cat -n file            # 带行号
more file              # 分页查看
less file              # 可上下翻页
head -n 20 file        # 前 20 行
tail -n 20 file        # 后 20 行
tail -f app.log        # 实时追踪日志（高频！）

# ========== 查找文件 ==========
find /path -name "*.log"           # 按名称查找
find /path -type f -size +100M     # 查找大于 100M 的文件
find /path -mtime -7               # 7 天内修改的文件
find /path -user root              # 查找属于 root 的文件
```

#### 2. 文本处理

```bash
# ========== grep 搜索（面试必问！）==========
grep "error" app.log              # 搜索关键字
grep -i "error" app.log           # 忽略大小写
grep -n "error" app.log           # 显示行号
grep -c "error" app.log           # 统计匹配行数
grep -r "error" /var/log/         # 递归搜索
grep -A 5 "error" app.log         # 显示匹配行及后 5 行
grep -B 5 "error" app.log         # 显示匹配行及前 5 行
grep -C 5 "error" app.log         # 前后各 5 行
grep -E "error|warning" app.log   # 正则表达式（或）
grep -v "debug" app.log           # 排除匹配行

# ========== 常用组合 ==========
ps -ef | grep java                # 查找 Java 进程
cat app.log | grep "2024-01-01"   # 管道组合
grep -r "TODO" --include="*.py"   # 只搜索 .py 文件

# ========== awk 文本处理 ==========
awk '{print $1}' file             # 打印第一列
awk -F: '{print $1}' /etc/passwd  # 指定分隔符
awk '{sum+=$1} END {print sum}'   # 求和
awk 'NR==10' file                 # 打印第 10 行
awk 'length>80' file              # 打印超过 80 字符的行

# ========== sed 流编辑器 ==========
sed 's/old/new/g' file            # 替换所有匹配
sed -i 's/old/new/g' file         # 直接修改文件
sed -n '10,20p' file              # 打印 10-20 行
sed '/pattern/d' file             # 删除匹配行
```

#### 3. 进程管理

```bash
# ========== 查看进程 ==========
ps -ef                            # 所有进程（标准格式）
ps -aux                           # 所有进程（BSD 格式）
ps -ef | grep java                # 查找特定进程
pgrep -l java                     # 按名称查找进程 ID

# ========== 动态监控 ==========
top                               # 实时进程监控
htop                              # 增强版（需安装）
top -p 1234                       # 监控特定 PID

# ========== 杀进程 ==========
kill 1234                         # 正常终止
kill -9 1234                      # 强制终止
killall java                      # 按名称杀进程
pkill -f "python script.py"       # 按命令行匹配

# ========== 后台运行 ==========
nohup python app.py &             # 后台运行，不挂断
nohup ./start.sh > log.txt 2>&1 & # 输出重定向
jobs                              # 查看后台任务
fg %1                             # 调到前台
```

#### 4. 权限管理

```bash
# ========== 权限说明 ==========
# r=4, w=2, x=1
# 755 = rwxr-xr-x (所有者全权限，其他读执行)
# 644 = rw-r--r-- (所有者读写，其他只读)

chmod 755 script.sh               # 数字方式
chmod +x script.sh                # 添加执行权限
chmod -R 755 /path                # 递归修改

# ========== 所有权 ==========
chown user:group file             # 修改所有者
chown -R user:group dir           # 递归修改
chgrp group file                  # 修改组

# ========== 查看权限 ==========
ls -l file                        # 查看权限
stat file                         # 详细信息
```

#### 5. 系统资源监控

```bash
# ========== 内存 ==========
free -m                           # MB 单位显示
free -h                           # 人类可读

# ========== 磁盘 ==========
df -h                             # 磁盘使用情况
du -sh /path                      # 目录大小
du -sh *                          # 当前各目录大小

# ========== 网络 ==========
netstat -tlnp                     # 监听端口
netstat -anp                      # 所有连接
ss -tlnp                          # 现代替代
lsof -i :8080                     # 查看端口占用

# ========== 系统 ==========
uname -a                          # 系统信息
hostname                          # 主机名
date                              # 日期时间
uptime                            # 运行时间和负载
```

---

### 🟠 重要

#### 6. 网络操作

```bash
# ========== 连通性 ==========
ping google.com                   # 测试连通
ping -c 4 google.com              # 4 次后停止

# ========== 端口扫描 ==========
telnet 192.168.1.1 8080           # 测试端口
nc -zv 192.168.1.1 8080           # 端口扫描

# ========== 数据传输 ==========
curl http://example.com           # HTTP 请求
curl -I http://example.com        # 只获取 header
curl -X POST -d "data" url        # POST 请求
curl -H "Content-Type: json" url  # 自定义 header

wget http://example.com/file.zip  # 下载文件
wget -c url                       # 断点续传

# ========== 远程连接 ==========
ssh user@192.168.1.1              # SSH 连接
ssh -p 2222 user@host             # 指定端口
scp file user@host:/path          # 上传文件
scp user@host:/path/file .        # 下载文件
scp -r dir user@host:/path        # 上传目录

# ========== DNS ==========
nslookup google.com               # DNS 查询
dig google.com                    # 详细 DNS 信息
host google.com                   # 简单查询
```

#### 7. 压缩与解压

```bash
# ========== tar（面试常问！）==========
tar -cvf archive.tar files        # 创建 tar
tar -xvf archive.tar              # 解压 tar
tar -czvf archive.tar.gz files    # 创建 gzip 压缩
tar -xzvf archive.tar.gz          # 解压 tar.gz
tar -cjvf archive.tar.bz2 files   # 创建 bzip2
tar -xjvf archive.tar.bz2         # 解压 tar.bz2

# ========== zip ==========
zip -r archive.zip dir            # 压缩目录
unzip archive.zip                 # 解压
unzip -l archive.zip              # 查看内容

# ========== 其他 ==========
gzip file                         # 压缩为 .gz
gunzip file.gz                    # 解压
bzip2 file                        # 压缩为 .bz2
```

#### 8. 定时任务

```bash
# ========== crontab ==========
crontab -l                        # 查看定时任务
crontab -e                        # 编辑定时任务
crontab -r                        # 删除定时任务

# 格式：分 时 日 月 周
# 示例：
* * * * * command                 # 每分钟
0 * * * * command                 # 每小时
0 0 * * * command                 # 每天 0 点
0 9 * * 1-5 command               # 工作日 9 点
*/5 * * * * command               # 每 5 分钟
0 0 1 * * command                 # 每月 1 号
```

---

### 🟡 常用

#### 9. 用户管理

```bash
# ========== 用户操作 ==========
useradd username                  # 添加用户
userdel username                  # 删除用户
usermod -aG wheel user            # 添加到组
passwd username                   # 修改密码

# ========== 组操作 ==========
groupadd groupname                # 添加组
groupdel groupname                # 删除组

# ========== 切换用户 ==========
su - username                     # 切换用户
sudo command                      # 以 root 执行
```

#### 10. 软件包管理

```bash
# ========== CentOS/RHEL ==========
yum install package               # 安装
yum remove package                # 卸载
yum update                        # 更新
yum search package                # 搜索
yum list installed                # 已安装列表

# ========== Ubuntu/Debian ==========
apt install package               # 安装
apt remove package                # 卸载
apt update                        # 更新源
apt upgrade                       # 升级软件

# ========== 通用 ==========
rpm -ivh package.rpm              # 安装 rpm
dpkg -i package.deb               # 安装 deb
```

#### 11. Vim 基础

```bash
# ========== 模式 ==========
# 命令模式（默认）→ i 进入插入模式
# 插入模式 → Esc 返回命令模式
# 命令模式 → : 进入底行模式

# ========== 基本操作 ==========
vim file                          # 打开文件
i                                 # 插入模式
:w                                # 保存
:q                                # 退出
:wq                               # 保存退出
:q!                               # 强制退出

# ========== 编辑操作 ==========
dd                                # 删除行
yy                                # 复制行
p                                 # 粘贴
u                                 # 撤销
gg                                # 到文件头
G                                 # 到文件尾
/keyword                          # 搜索
:n                                # 跳到第 n 行

# ========== 格式转换 ==========
:set ff=unix                      # 转为 Unix 格式
:set ff=dos                       # 转为 DOS 格式
```

---

### 🟢 了解

#### 12. 高级命令

```bash
# ========== 磁盘 I/O ==========
iotop                             # I/O 监控
iostat -x 1                       # 详细 I/O 统计

# ========== 性能分析 ==========
strace -p PID                     # 跟踪系统调用
ltrace command                    # 跟踪库调用

# ========== 日志分析 ==========
journalctl -u service             # systemd 日志
dmesg                             # 内核日志

# ========== 文件系统 ==========
lsof | grep deleted               # 已删除但占空间的文件
fuser -v /path                    # 查看文件使用者

# ========== 高级文本 ==========
sort file | uniq -c               # 排序并统计重复
cut -d: -f1 /etc/passwd           # 按列切割
wc -l file                        # 统计行数
diff file1 file2                  # 对比文件
```

---

## 三、面试高频问题

### 基础篇

| 问题 | 答案 |
|------|------|
| 如何查看当前目录？ | `pwd` |
| 如何查看所有文件包括隐藏文件？ | `ls -la` |
| 如何查看日志文件最后 100 行？ | `tail -n 100 app.log` |
| 如何实时查看日志？ | `tail -f app.log` |
| 如何查找包含 "error" 的行？ | `grep "error" app.log` |
| 绝对路径符号是什么？ | `/` |
| 当前目录和上级目录符号？ | `.` 和 `..` |

### 进阶篇

| 问题 | 答案 |
|------|------|
| 如何查看 Java 进程？ | `ps -ef \| grep java` |
| 如何查看端口占用？ | `netstat -tlnp` 或 `lsof -i :port` |
| 如何查看磁盘空间？ | `df -h` |
| 如何查看内存使用？ | `free -h` |
| 如何杀掉进程？ | `kill -9 PID` |
| chmod 755 是什么意思？ | rwxr-xr-x |
| 如何解压 tar.gz？ | `tar -xzvf file.tar.gz` |
| 如何在后台运行程序？ | `nohup command &` |

### 高级篇

| 问题 | 答案 |
|------|------|
| 如何统计日志中 error 出现次数？ | `grep -c "error" app.log` |
| 如何查找 7 天前的日志文件并删除？ | `find /logs -name "*.log" -mtime +7 -delete` |
| 如何查看最占内存的 10 个进程？ | `ps -aux --sort=-%mem \| head -11` |
| 如何实现日志分析脚本？ | 结合 grep/awk/sed |
| Windows 换行和 Linux 换行的区别？ | CRLF vs LF，用 `dos2unix` 转换 |

---

## 四、实战场景

### 场景1：查找日志中的错误

```bash
# 查找今天的 error 日志
grep "$(date +%Y-%m-%d)" app.log | grep -i "error"

# 统计各错误类型数量
grep "error" app.log | awk -F: '{print $1}' | sort | uniq -c | sort -rn

# 查找并显示上下文
grep -C 10 "NullPointerException" app.log
```

### 场景2：监控服务状态

```bash
# 检查服务是否运行
ps -ef | grep tomcat | grep -v grep

# 检查端口是否监听
netstat -tlnp | grep 8080

# 检查磁盘空间是否充足
df -h | awk '$5 > 80 {print $0}'
```

### 场景3：清理日志文件

```bash
# 删除 30 天前的日志
find /var/log -name "*.log" -mtime +30 -delete

# 压缩 7 天前的日志
find /var/log -name "*.log" -mtime +7 -exec gzip {} \;

# 清空大日志文件（不删除）
> app.log
```

---

## 五、命令速查表

### 文件操作

| 命令 | 说明 |
|------|------|
| `ls -la` | 列出所有文件 |
| `cd` | 切换目录 |
| `pwd` | 当前目录 |
| `mkdir -p` | 递归创建目录 |
| `rm -rf` | 强制删除 |
| `cp -r` | 递归复制 |
| `mv` | 移动/重命名 |

### 查看文件

| 命令 | 说明 |
|------|------|
| `cat` | 显示全部 |
| `more` | 分页查看 |
| `less` | 可翻页 |
| `head -n` | 前 n 行 |
| `tail -f` | 实时追踪 |
| `wc -l` | 统计行数 |

### 搜索查找

| 命令 | 说明 |
|------|------|
| `grep` | 文本搜索 |
| `find` | 文件查找 |
| `which` | 命令位置 |
| `whereis` | 程序位置 |

### 进程管理

| 命令 | 说明 |
|------|------|
| `ps -ef` | 所有进程 |
| `top` | 实时监控 |
| `kill -9` | 强制终止 |
| `nohup &` | 后台运行 |

### 系统监控

| 命令 | 说明 |
|------|------|
| `free -h` | 内存使用 |
| `df -h` | 磁盘使用 |
| `du -sh` | 目录大小 |
| `netstat -tlnp` | 端口监听 |

---

## 相关知识点

- [[Docker 常用命令速查]]
- [[Pytest 面试完全指南]]
- [[Playwright 测试框架精通指南]]

---
*采集自 Claude Code 对话*

**Sources:**
- [测试面试常问的linux命令 - WorkTile](https://worktile.com/kb/ask/441726.html)
- [Linux面试必背20个命令 - WorkTile](https://worktile.com/kb/ask/444673.html)
- [Linux命令大全 - 菜鸟教程](https://www.runoob.com/linux/linux-command-manual.html)
