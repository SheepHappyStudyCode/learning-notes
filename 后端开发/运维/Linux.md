﻿﻿﻿# Linux
## 1 Linux基础知识
### 1.1 虚拟机快照
作用：通过快照将当前虚拟机的状态保存下来，以后可以通过快照将虚拟机恢复到之前的状态。
## 2 Linux基础命令
1. 命令的**基础格式**：`command [-options] [parameter]`
2. 展示目录下的文件信息：`ls [-a -l -h] [Linux路径]`
3. 输出当前所在目录：`pwd`
4. 切换目录：`cd Linux路径`
5. 创建文件夹：`mkdir [-p] Linux路径`
	- p选项功能：创建多级目录
6. 文件操作
	- 创建文件：`touch 文件路径`
	- 查看文件内容：
		1. `cat 文件路径`：查看文件的所有内容
		2. `more 文件路径`：分页查看文件内容
7. 复制文件：`cp [-r] 起始路径 终点路径`
	复制文件夹要用r选项
8. 移动文件：`mv 起始路径 终点路径`
9. 删除文件夹：`rm [-r] [-f] 参数1，参数2...参数N`
	注意点：可以用*进行模糊匹配 
10. 查找命令：
	- 查找命令所在文件位置：`which command`
	- 通过文件名查找文件：`find 起始位置 -name "文件名"`
	- 通过文件大小查找文件：`find 起始位置 -size + | -n[kMG]` 
11. 过滤文件内容：`grep [-n] "关键字“ 文件路径`
12. 统计文件的行数，单词数量等：`wc [-c -m -l -w] 文件路径`
13. **管道符**：管道符左边的结果作为管道符右边命令的输入
14. 在命令行输出指定内容：`echo 输出的内容`
	通过反引号可以输出命令的结果
15. 重定向符：
	- 将左边的结果以覆盖的方式写到右边的文件:`>`
	- 将左边的结果以追加的方式写到右边的文件:`>>`
16. 查看文件尾部的内容：`tail [-f -num] 路径`
17. **vi编辑器**
	- 命令模式：是其他模式的中转站，有许多快捷键。
	- 输入模式：
	- 底线命令模式：负责对整体文件进行操作，例如保存、退出。
## 3 Linux用户和权限
1. 切换用户：`su [-] [用户名]`
	-  `-`表示切换用户后加载环境变量，一般建议带上
2. 临时获得root权限：`sudo 其他命令`
3. 用户和用户组管理命令：`groupadd groupdel useradd userdel id usermod getent`
4. 文件权限：
	- `r`：可读
	- `w`：可写
	- `x`：可执行，若是文件夹则是可以cd。
5. 修改权限控制：`chmod [-R] 权限 文件或文件夹`
	- `-R`：对文件夹内的文件也执行同样的操作
	- 示例：`chmod u=rwx, g=rx, o=x test.txt`
	- 示例：`chmod 751 hello.txt`
6. 修改文件、文件夹的所属用户、用户组：`chown [-R] [用户][:][用户组] 文件或文件夹`
	- **只有root用户才能执行此命令**
## 4 Linux实用操作
1. 各种快捷键：
	- 登出：ctrl + d
	- 搜索历史命令：ctrl + r
2. 软件安装
	- yum命令安装软件：`yum [-y] [install | remove | search] 软件名称`（需要root权限）
	- ubuntu安装程序：`apt [-y] [install | remove | search] 软件名称`
3. 通过systemctl管理软件：`systemctl start | stop | status | enable | disable 服务名`
4. ln命令创建软链接：`ln -s 参数1 参数2`
	- 参数1：被链接的文件或文件夹
	- 参数2：要链接去的目的地
5. 日期和时区
	- 显示日期：`date [-d] [+格式化字符串]`
6. 检查网络服务器是否可联通：`ping [-c num] ip或主机名`
7. 在命令行下载网络文件：`wget [-b] url`
8. 发送网络请求：`curl [-O] url`
9. 查看端口占用：
	- `nmap IP地址`：查看指定IP对外暴露的端口
	- `netstat -anp | grep 端口号`：查看本机指定端口额占用情况 
10. 进程管理
	- 查看进程信息：`ps [-e -f]``
	- 关闭进程：`kill [-9] 进程id`，`-9`表示强制关闭。
11. 主机状态
	- 系统资源监控：`top`，有很多选项和交互快捷键
	- 磁盘信息监控：
	`df [-h]`
	`iostat [-x] [num1] [num2]`
	- 网络监控：`sar -n DEV [num1] [num2]`
12. 环境变量
	- 显示环境变量：`env`
	- 取到环境变量的值：`echo $环境变量名`
	- 临时设置环境变量：`export 变量=值`
	- 永久设置环境变量：修改对应的配置文件
13. 上传、下载
	- 鼠标拖拽：速度更快
	- 命令
14. 压缩和解压
	- `tar [-c -v -x -f -z -C] 参数1，参数2...参数n`
	- `c`：创建压缩文件
	- `v`：过程可视化
	- `x`：解压模式
	- `f`：压缩的文件或者解压的文件，该选项必须放在**最后面**
	- `z`：gzip模式
	- `C`：选择解压的目的地
	- `zip [-r] 参数1，参数2...参数n`
	- `unzip [-d] 参数`
	- `d`：指定解压的位置

## 5 firewall 常见命令整理

## 6 常见命令整理

### 查看本地端口

Linux系统作为**客户端发起连接**时可自动分配的本地端口范围

```linux
sysctl -a | grep ip_local_port_range
```

### AWK

`awk`：一个强大的文本处理命令

语法：`awk [选项参数] 'script' 文件名`

使用示例：

```shell
# 统计nginx日志的流量、请求数、各状态码数量

BEGIN {
  print "统计nginx日志数据"
}{
  total_size += $10
  req++
  status_code[$9]++
}END{
  printf "Total Requests: %d\nTotal Size: %.2f MB\n", req, total_size/1024/1024
  print "[Status Code]  Count"
  for(code in status_code){
    printf "     %-9s  %d\n", code, status_code[code]
  }
}

================================================
上面是 awk 脚本内容，下面是命令行
================================================
awk -f nginx_monitor.awk access.log
```

### 查看内存的命令

使用 GUI 界面查看

查看`/proc/meminfo`这个虚拟文件

```shell
$ cat /proc/meminfo
MemTotal:        1798504 kB
MemFree:          113748 kB
MemAvailable:     666276 kB
Buffers:          120648 kB
Cached:           527024 kB
SwapCached:        17392 kB
Active:           718292 kB
Inactive:         790892 kB
Active(anon):     388508 kB
Inactive(anon):   473628 kB
Active(file):     329784 kB
Inactive(file):   317264 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       1049596 kB
SwapFree:         975000 kB
Dirty:                28 kB
Writeback:             0 kB
AnonPages:        859260 kB
Mapped:            65936 kB
Shmem:               624 kB
Slab:              97008 kB
SReclaimable:      72656 kB
SUnreclaim:        24352 kB
KernelStack:        4736 kB
PageTables:        12780 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     1948848 kB
Committed_AS:    1971280 kB
VmallocTotal:   34359738367 kB
VmallocUsed:        9636 kB
VmallocChunk:   34359700716 kB
Percpu:              960 kB
HardwareCorrupted:     0 kB
AnonHugePages:     55296 kB
CmaTotal:              0 kB
CmaFree:               0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:      101048 kB
DirectMap2M:     1912832 kB
DirectMap1G:           0 kB
```

`free`命令

```shell
$ free
              total        used        free      shared  buff/cache   available
Mem:        1798504      964892      113236         624      720376      665812
Swap:       1049596       74596      975000
```

`vmstat`
