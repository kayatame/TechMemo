## 1. CentOS7ミラーファイル調整
### 1-1. 該当エラー
```
# yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

<details>
<summary>エラー内容</summary>

```
Loaded plugins: fastestmirror
remi-release-7.rpm                                                                                                                                                               |  28 kB  00:00:00
Examining /var/tmp/yum-root-Ve07MM/remi-release-7.rpm: remi-release-7.9-6.el7.remi.noarch
Marking /var/tmp/yum-root-Ve07MM/remi-release-7.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package remi-release.noarch 0:7.9-6.el7.remi will be installed
--> Processing Dependency: epel-release = 7 for package: remi-release-7.9-6.el7.remi.noarch
Loading mirror speeds from cached hostfile
Could not retrieve mirrorlist http://mirrorlist.centos.org/?release=7&arch=x86_64&repo=os&infra=genclo error was
14: curl#6 - "Could not resolve host: mirrorlist.centos.org; Unknown error"


 One of the configured repositories failed (Unknown),
 and yum doesn't have enough cached data to continue. At this point the only
 safe thing yum can do is fail. There are a few ways to work "fix" this:

     1. Contact the upstream for the repository and get them to fix the problem.

     2. Reconfigure the baseurl/etc. for the repository, to point to a working
        upstream. This is most often useful if you are using a newer
        distribution release than is supported by the repository (and the
        packages for the previous distribution release still work).

     3. Run the command with the repository temporarily disabled
            yum --disablerepo=<repoid> ...

     4. Disable the repository permanently, so yum won't use it by default. Yum
        will then just ignore the repository until you permanently enable it
        again or use --enablerepo for temporary usage:

            yum-config-manager --disable <repoid>
        or
            subscription-manager repos --disable=<repoid>

     5. Configure the failing repository to be skipped, if it is unavailable.
        Note that yum will try to contact the repo. when it runs most commands,
        so will have to try and fail each time (and thus. yum will be be much
        slower). If it is a very temporary problem though, this is often a nice
        compromise:

            yum-config-manager --save --setopt=<repoid>.skip_if_unavailable=true

Cannot find a valid baseurl for repo: base/7/x86_64
```
</details>

### 1-2. リポジトリ設定調整
```
# cp -ip /etc/yum.repos.d/CentOS-Base.repo{,.$(date +%Y%m%d)}
# vi /etc/yum.repos.d/CentOS-Base.repo
-----------------------------------------------------------------------------
[base]
name=CentOS-$releasever - Base
baseurl=http://vault.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-$releasever - Updates
baseurl=http://vault.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-$releasever - Extras
baseurl=http://vault.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[centosplus]
name=CentOS-$releasever - Plus
baseurl=http://vault.centos.org/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
-----------------------------------------------------------------------------
```