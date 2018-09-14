>1.拷贝gitlab-ci-multi-runner-1.0.1-1.x86_64.rpm到目标主机

    sudo yum install -y gitlab-ci-multi-runner-1.0.1-1.x86_64.rpm

>2.配置用户：

    sudo gitlab-ci-multi-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner

>3.修改/etc/hosts

    vi /etc/hosts
    132.37.5.190    gitlab.woego.cn

>4.注册：

    gitlab-ci-multi-runner register
    Please enter the gitlab-ci coordinator URL (e.g. http://gitlab-ci.org:3000/):
    http://gitlab.woego.cn/ci
    Please enter the gitlab-ci token for this runner:
    wdyEBWaDVPLNTjnXvQHw
    Please enter the gitlab-ci description for this runner:
    [gxuweg3tst1]: 132.37.5.190
    INFO[0028] a8b4cd26 Registering runner... succeeded
    Please enter the executor: ssh, shell, parallels, docker, docker-ssh:
    [shell]:shell
    INFO[0031] Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!

>5.指定安装目录并启动

    [root@gxuweg3tst1 bin]# gitlab-ci-multi-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
    [root@gxuweg3tst1 bin]# gitlab-ci-multi-runner start

>6.查看进程

    ps -ef |grep git
