# Apline学习笔记


CentOS离场，把玩debian。

<!--more-->

## Docker

### 开启远程管理

Alpine管理服务是用RC的组件，如果开启远程管理，需要修改 `/etc/init.d/docker`，在`command_args`参数后 增加：`-H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock`， 完整的文件如下：

```bash
#!/sbin/openrc-run
# Copyright 1999-2013 Gentoo Foundation
# Distributed under the terms of the GNU General Public License v2

command="${DOCKERD_BINARY:-/usr/bin/dockerd}"
pidfile="${DOCKER_PIDFILE:-/run/${RC_SVCNAME}.pid}"
command_args="-p \"${pidfile}\" ${DOCKER_OPTS} -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock"
DOCKER_LOGFILE="${DOCKER_LOGFILE:-/var/log/${RC_SVCNAME}.log}"
DOCKER_ERRFILE="${DOCKER_ERRFILE:-${DOCKER_LOGFILE}}"
DOCKER_OUTFILE="${DOCKER_OUTFILE:-${DOCKER_LOGFILE}}"
start_stop_daemon_args="--background \
    --stderr \"${DOCKER_ERRFILE}\" --stdout \"${DOCKER_OUTFILE}\""

extra_started_commands="reload"

rc_ulimit="${DOCKER_ULIMIT:--c unlimited -n 1048576 -u unlimited}"

retry="${DOCKER_RETRY:-TERM/60/KILL/10}"

depend() {
    need sysfs cgroups
}

start_pre() {
    checkpath -f -m 0644 -o root:docker "$DOCKER_LOGFILE"
}

reload() {
        ebegin "Reloading ${RC_SVCNAME}"
        start-stop-daemon --signal HUP --pidfile "${pidfile}"
        eend $? "Failed to stop ${RC_SVCNAME}"
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
cat > /root/.smb.auth
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

