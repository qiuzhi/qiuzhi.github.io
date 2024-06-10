# Apline学习笔记


CentOS离场，把玩debian。

&lt;!--more--&gt;

## Docker

### 开启远程管理

Alpine管理服务是用RC的组件，如果开启远程管理，需要修改 `/etc/init.d/docker`，在`command_args`参数后 增加：`-H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock`， 完整的文件如下：

```bash
#!/sbin/openrc-run
# Copyright 1999-2013 Gentoo Foundation
# Distributed under the terms of the GNU General Public License v2

command=&#34;${DOCKERD_BINARY:-/usr/bin/dockerd}&#34;
pidfile=&#34;${DOCKER_PIDFILE:-/run/${RC_SVCNAME}.pid}&#34;
command_args=&#34;-p \&#34;${pidfile}\&#34; ${DOCKER_OPTS} -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock&#34;
DOCKER_LOGFILE=&#34;${DOCKER_LOGFILE:-/var/log/${RC_SVCNAME}.log}&#34;
DOCKER_ERRFILE=&#34;${DOCKER_ERRFILE:-${DOCKER_LOGFILE}}&#34;
DOCKER_OUTFILE=&#34;${DOCKER_OUTFILE:-${DOCKER_LOGFILE}}&#34;
start_stop_daemon_args=&#34;--background \
    --stderr \&#34;${DOCKER_ERRFILE}\&#34; --stdout \&#34;${DOCKER_OUTFILE}\&#34;&#34;

extra_started_commands=&#34;reload&#34;

rc_ulimit=&#34;${DOCKER_ULIMIT:--c unlimited -n 1048576 -u unlimited}&#34;

retry=&#34;${DOCKER_RETRY:-TERM/60/KILL/10}&#34;

depend() {
    need sysfs cgroups
}

start_pre() {
    checkpath -f -m 0644 -o root:docker &#34;$DOCKER_LOGFILE&#34;
}

reload() {
        ebegin &#34;Reloading ${RC_SVCNAME}&#34;
        start-stop-daemon --signal HUP --pidfile &#34;${pidfile}&#34;
        eend $? &#34;Failed to stop ${RC_SVCNAME}&#34;
}
```

然后重启 Docker 服务：

```bash
service docker restart
```

## 挂载

### cifs

```bash
# 必要的依赖组件
apk add cifs-utils openrc
sudo rc-update add netmount

//192.168.9.23/Disk_sataa5 /mnt/zidoo cifs  _netdev,gid=0,uid=0,username=guest,password=guest,dynperm,exec,noacl,nobrl,nofail,nounix,rw,serverino,setuids,sfu 0 0

# 在 /etc/fstab 文件末位增加一行，用于配置 SAMBA/SMB/CIFS 服务挂载项
//192.168.9.23/Disk_sataa5 /mnt/zidoo cifs  _netdev,credentials=/root/.smb.auth,gid=1000,uid=1000,dynperm,exec,noacl,nobrl,nofail,nounix,rw,serverino,setuids,sfu 0 0

# credentials file content - 建议以下认证信息不要包含特殊符号，以免无法认证
# /root/.smb.auth
cat &gt; /root/.smb.auth
username=username
password=password
domain=doaminname

chmod 600 /root/.smb.auth

# 初次手动挂载(免重启)
mount -a
```

参考

1. [Alpine Linux 中 Docker 开启远程管理](https://www.cnblogs.com/alexyangchina/p/12973131.html)
2. [SAMBA/SMB/CIFS 开机自动挂载 for Alpine Linux](https://blog.gazer.win/essay/cifs-netmount-on-alpine-linux.html)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/study-apline/  

