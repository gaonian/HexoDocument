## 用户组
在linux中的每个用户必须属于一个组，不能独立于组外。在linux中每个文件有所有者、所在组、其它组的概念

- 所有者
- 所在组
- 其它组
- 改变用户所在的组


### 所有者

一般为文件的创建者，谁创建了该文件，就天然的成为该文件的所有者

用ls ‐ahl命令可以看到文件的所有者

也可以使用chown 用户名 文件名来修改文件的所有者

 
 

### 文件所在组

当某个用户创建了一个文件后，这个文件的所在组就是该用户所在的组

用ls ‐ahl命令可以看到文件的所有组

也可以使用chgrp 组名 文件名来修改文件所在的组

 

### 其它组

除开文件的所有者和所在组的用户外，系统的其它用户都是文件的其它组









类型 | 举例 
---|---
用户 | `who, whoami, su, useradd, userdel, passwd, usermod, /etc/passwd`
组 | `groupadd, groupdel, groupmod, /etc/group`
文件 | `chmod, chown, chgrp`
其他 | `sudo, exit`





## 1. 用户
- 已登录的用户信息: `who`

```
显示当前所有已登录到系统的用户信息。

参数:
    -H    显示各列的标题（表头）。
    -u    显示闲置时间。如果用户在前1分钟内有操作, 显示“.”; 超过24小时没有操作, 显示“old”字符串。
    -m    显示当前会话的用户的信息, 相当于: who am i
    -q    显示当前所有已登录的 账户名称 和 总人数。
    -T    显示用户的 状态信息

例子:
    who         // 显示当前所有已登录的用户
    who -q      // 显示当前所有已登录的账号名称和总人数
    who am i    // 显示当前会话的用户的信息
    who -m      // 显示当前会话的用户的信息
    who -uTH    // 显示当前所有已登录的用户信息（包括 闲置时间, 用户状态, 各列标题）
```

- 显示当前会话的用户的名称: `whoami`
 
```
显示当前会话的用户的名称, 格式: whoami
```

- 切换用户: `su`
 
```
select user, 切换用户。su 后面加上 “-” 表示切换用户后同时把工作目录切换到该用户的 home 目录。

例子:
    su username         // 切换到指定用户
    su - username       // 切换到指定用户, 并同时把工作目录切换到用户的home目录
    su                  // 没有指定用户名, 则切换到 root 用户
    su -                // 切换到 root 用户, 并把工作目录切换到 /root
    su root             // 切换到 root 用户
    su - root           // 切换到 root 用户, 并把工作目录切换到 /root
```

- 添加用户: `useradd`

```
格式: useradd [-potions] username

参数:
    -d    指定新建用户的主目录, 如果不指定, 则系统自动在 /home 目录创建
          一个和用户名相同的文件夹作为新建用户的主目录（/home/username）
          
    -m    自动创建主目录文件夹
    
    -g    指定新建用户所在组名称, 如果不指定, 则系统自动创建一个和用户名
          相同的组作为新建用户所属组

例子:
    useradd -m user01                   // 新建用户 user01, 自动创建主目录 /home/user01, 自动创建组 user01
    useradd -md /home/u2 user02         // 新建用户 user02, 自动创建主目录 /home/u2, 自动创建组 user02
    useradd -md /home/u3 -g g3 user03   // 新建用户 user03, 自动创建主目录 /home/u3, 指定组为 g3（指定的组必须已存在）

注意:
    新创建的用户默认不能使用 sudo 命令, 需要另外添加该命令的权限, 实际上 sudo 是一个组的名称, 
    把 组sudo 追加到用户的附加组列表, 该用户便能在输入用户自己密码的情况下执行 sudo 命令。
    
    例如让用户 user01 拥有 sudo 命令权限, 命令格式为: usermod -aG sudo user01
    
    具体参考后面的 usermod 命令介绍。
    ```

- 删除用户: `userdel`

```
删除用户时, 如果用户所属组是创建用户时自动创建的和用户名称同名的组, 并且该组内没有其他用户, 则该组也会被删掉。

格式: userdel [-options] username

参数:
    -f    强制删除用户, 即使用户当前已登录
    -r    删除用户, 同时删除用户主目录

例子:
    userdel user01          // 删除用户 user01, 保留用户主目录
    userdel -r user02       // 删除用户 user02, 并同时删除用户主目录
    ```

- 更改用户密码:`passwd`

```
设置或修改用户密码, 普通用户可以根据原密码修改自己的密码, 超级用户可以直接重置所有用户（包括 root 用户）的密码。

格式: passwd [-options] [username]

参数:
    -a    all, 此选项只能和 -S 一起使用, 来显示所有用户的状态
    -d    delete, 删除用户密码（把密码置为空）
    -l    lock, 锁定指定账户（将密码更改为一个不可能与加密值匹配的值来禁用）
    -u    unlock, 解锁指定账户
    -S    status, 显示账户状态信息

例子:
    passwd              // 更改自己的密码, 输入 原密码, 新密码, 确认新密码
    passwd user01       // 重置用户的密码, 输入 新密码, 确认新密码
    passwd -aS          // 查看所有用户的状态
    passwd -S user01    // 查看用户 user01 的状态
    ```

- 修改用户信息: `usermod`

```
usermode 命令用于修改用户的基本信息。不允许修改正在线上的用户的名称, 不允许修改正在系统上执行程序的用户的ID。

格式: usermod [-options] username

参数:
    -c    comment, 设置帐户注释
    -d    home_dir, 设置用户主目录
    -e    expiredate, 设置帐户过期日期
    -g    group, 强制修改用户的新主组
    -G    groups, 修改用户新的附加组列表
    -a    append group, 将用户追加至上边 -G 中提到的附加组中, 并不从其它组中删除此用户
    -u    uid, 修改账户的用户ID
    -l    login, 设置用户的登录名称
    -s    shell, 修改用户登录后使用的 shell
    -L    lock, 锁定用户帐户（等同于 passwd -l 命令锁定账户）
    -U    unlock, 解锁用户帐户（等同于 passwd -u 命令解账户）

例子:
    usermod -g group01 user01   // 把用户 user01 的主组修改为 group01 组
    usermod -u 9999 user01      // 把用户 user01 的 ID 修改为 9999
    usermod -l u1 user01        // 把用户 user01 的名称修改为 u1
    
    usermod -aG sudo user01     // 把 组sudo 添加到用户 user01 的附加组列表, 使该用户拥有执行 sudo 命令的权限
    ```

- 用户账户信息: /etc/passwd

```
文件 /etc/passwd 中存放了所有用户的账户信息, 每一行为一个用户, 每个用户的账户信息分为 7 个字段, 用 “:” 分隔。

格式:
    Username : Password : User ID : Group ID : Comment : Home Diretory : Login Command
    用户名    : 密码      : 用户ID   : 组ID     : 用户注解 : 主目录         : 登录后执行的命令

其中 密码字段 只显示一个特殊字符“x”或“*”, 加密后的密码存放在 /etc/shadow 文件中, 只有超级用户才能访问。
```


## 2. 组
- 添加组: `groupadd`

```
格式: groupadd [-options] groupname

参数:
    -g    指定新建组的ID

例子:
    groupadd group01            // 新建名称为 group01 的组
    groupadd -g 1001 group02    // 新建名称为 group02 的组,  并给指定组的 ID 为 1001
    ```

- 删除组: `groupdel`

```
删除组, 如果组内还包含有用户, 需要想将用户删除或移出该组后才能删除。

格式: groupdel groupname
```

- 改变组信息: `groupmod`

```
格式: groupmod [-options] groupname

参数:
    -n    修改组的名称

例子:
    groupmod -n NewGroupName OldGroupName
    ```

- 查看所有组: `/etc/group`

```
文件 /etc/group 中存放了所有组的信息, 每一行表示一个组, 每个组的信息分为 4 个字段, 用 “:” 分隔。

格式:
    Group Name : Password : Group ID : User List
    ```

## 3. 文件
- Linux的文件权限用 3组 每组3位 共9位 字符表示

```
终端输入命令ls -al查看当前目录下的所有文件的权限:

-r--r--r--
-rw-------
drw-r--r--
-rw-rw-rw-
drwx------
-rwxr--r--
lrwxr-xr-x
-rwxrwxrwx
```

上面每行列出了10个字符，其中第1位表示文件的类型: d表示文件夹, l表示链接文件, -表示文件。

剩余9位分为3组，每组3位，分别表示: 文件所有者、同组用户、其他用户 对该文件的权限。

每组的3位字母分别表示对该文件的读(r)、写(w)、执行(x)权限，显示字母表示有该权限，显示-表示没有该权限。

- 权限的数字解析

```
每组权限可分别由3位二进制数分别表示 r、w、x 的权限开关，1表示有该权限，0表示没有该权限，3位二进制数转换为十进制的一个整数即可用于表示读、写、执行三个权限的开关。如下案例:

// 可总结为: r=4, w=2, x=1
rwx         r-x         rw-         ---
111 == 7    101 == 5    110 == 6    000 = 0

// 上面表示是单组权限, 使用过程中一般三组权限一起使用
rwxrwxrwx           r--r--r--           rw-rw-rw-           rw-r-----
111111111 == 777    100100100 == 444    110110110 == 666    110100000 == 640

PS: 上面的777, 444等权限数字, 应该 拆分解读, 即 3位数字 分别表示 三组用户(所有者, 同组用户, 其他用户)对文件的权限。
```

- 修改文件/目录的 权限: `chmod`

```
chmod修改文件权限有两种形式: 字母形式 和 数字形式

字母形式:

    格式: 
        chmod [u/g/o/a][+/-/=][r/w/x] file/dir

    参数:
        u/g/o/a: 修改哪个组的权限, 分别表示 所有者(user), 所在组(group), 其他组(other), 所有(all)
        +/-/=: 增加(+), 撤销(-) 或 重置(=) 权限
        r/w/x: 需要操作的权限, 读(r)、写(w)、执行(x)

    例子:
        chmod u+x aa.sh             // 给文件所有者添加执行权限
        chmod a+x aa.sh             // 给所有用户添加执行权限
        chmod a=rwx aa.sh           // 给所有用户添加读写和执行权限
        chomd a= aa.sh              // 撤销所有用户的所有权限, 相当于 a-rwx
        chmod o-w aa.txt            // 给其他组的用户撤销写权限
        chmod u=rw,g=r,o= aa.sh     // 可以用逗号分隔分别给不同权限组修改权限

数字形式:

    权限有3位数字组成（注意: 每一位单独使用, 分别表示 所有者、所在组、其他组 的权限）,
    读(r) = 4, 写(w) = 2, 执行(x) = 1, 没有(-) = 0
    
    格式: 
        chmod [3位数字权限] file/dir
    
    例子:
        chmod 777 aa.sh             // 给所有用户添加读写和执行权限
        chmod 640 aa.sh             // 所有者有读写权限, 所在组有读权限, 其他组没有任何权限
        ```

- 修改文件/目录的 所有者: `chown`

```
修改文件所有者(change owner)

格式: chown [-options] username file/dir

参数: 
    -R      递归处理, 如果修改的是目录, 则递归修改目录下的所有文件和子目录

例子:
    chonw user01 aa.txt         // 把文件 aa.txt 的所有者修改为 用户user01
    chonw -R user01 bbDir       // 递归修改 目录bbDir 的所有者为 用户user01
    ```

- 修改文件/目录的 所属组: `chgrp`

```
修改文件所在组(change group)

格式: chgrp [-options] group_name file/dir

参数: 
    -R      递归处理, 如果修改的是目录, 则递归修改目录下的所有文件和子目录

例子:
    chgrp group01 aa.txt         // 把文件 aa.txt 的所在组修改为 组group01
    chgrp -R group01 bbDir       // 递归修改 目录bbDir 的所在组为 组group01
    ```

## 4. 其他
- 以其他身份来执行命令: `sudo`

```
sodu, 以其他身份来执行命令, 默认预设的身份为 root 用户。普通用户身份可能没有权限执行某些命令或查看某些
文件/文件夹, 此时需要切换到 root 用户, 如此比较麻烦, 也会暴露 root 用户密码给更多的普通用户, 因此可以
为用户添加 sudo 权限（详见上面的 usermod 命令介绍）, 让该用户可以临时以 root 身份来执行相关命令。
执行该命令需要输入当前登录用户的密码。

格式: sudo [-options] original_commands

参数:
    -H      将HOME环境变量设为新身份的HOME环境变量。
    -u      以指定用户的身份来执行命令, 默认为 root 用户。

例子:
    sudo ls /root       // 以 root 身份查看 /root 目录
    ```

- 退出shell: `exit`

```
退出shell, 或者退出当前登录的用户, 并返回给定值, 返回0表示执行成功正常退出, 返回非0表示执行失败异常退出。

例子: 
    exit        // 退出shell, 正常退出, 默认返回0
    exit 1      // 退出shell, 异常退出, 返回给定值
```




```
* /etc/passwd：     用户账户的详细信息在此文件中更新。
* /etc/shadow：     用户账户密码在此文件中更新。
* /etc/group：      新用户群组的详细信息在此文件中更新。
* /etc/gshadow：    新用户群组密码在此文件中更新。
```

- /etc/passwd

 `/etc/passwd` 是一个文本文件，其中包含了登录 Linux 系统所必需的每个用户的信息。它保存用户的有用信息，如用户名、密码、用户 ID、群组 ID、用户 ID 信息、用户的家目录和 Shell 。
 
 `/etc/passwd` 文件将每个用户的详细信息写为一行，其中包含七个字段，每个字段之间用冒号 : 分隔：


```
# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
2gadmin:x:500:10::/home/viadmin:/bin/bash
apache:x:48:48:Apache:/var/www:/sbin/nologin
zabbix:x:498:499:Zabbix Monitoring System:/var/lib/zabbix:/sbin/nologin
mysql:x:497:502::/home/mysql:/bin/bash
zend:x:502:503::/u01/zend/zend/gui/lighttpd:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/cache/rpcbind:/sbin/nologin
2daygeek:x:503:504::/home/2daygeek:/bin/bash
named:x:25:25:Named:/var/named:/sbin/nologin
mageshm:x:506:507:2g Admin - Magesh M:/home/mageshm:/bin/bash
```
7 个字段的详细信息如下。

1. 用户名 （magesh）： 已创建用户的用户名，字符长度 1 个到 12 个字符。
2. 密码（x）：代表加密密码保存在 `/etc/shadow` 文件中。
3. 用户 ID（506）：代表用户的 ID 号，每个用户都要有一个唯一的 ID 。UID 号为 0 的是为 root 用户保留的，UID 号 1 到 99 是为系统用户保留的，UID 号 100-999 是为系统账户和群组保留的。
4. 群组 ID （507）：代表群组的 ID 号，每个群组都要有一个唯一的 GID ，保存在 /etc/group文件中。
5. 用户信息（2g Admin - Magesh M）：代表描述字段，可以用来描述用户的信息（LCTT 译注：此处原文疑有误）。
6. 家目录（/home/mageshm）：代表用户的家目录。
7. Shell（/bin/bash）：代表用户使用的 shell 类型。
