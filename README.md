## ssh honeypot

Patch openssh from **source code** to log user-password combinations, as what ssh-honeypot does.

**ILLEGAL PURPOSE IS FORBIDDEN!**

Description
-----------
Openssh `sshd.service` doesn't record password to its log by default.
```zsh
$ systemctl status sshd.service
...
Jun 04 23:15:32 VM-X-XX-centos sshd[17196]: Failed password for invalid user XXX from XX.XX.XX.XX port 26179 ssh2
...
```
Then why not patch it to log user-password combinations for further analyses? ┐(￣ヮ￣)┌
```zsh
$ systemctl status sshd.service
...
Jun 04 23:15:31 VM-X-XX-centos sshd[17196]: ssh_honeypot: user=`Whos_there` password=`How_dare_you!` host=`XX.XX.XX.XX` port=`26179`
...
```
In CentOS7, ssh login messages are saved in `/var/log/secure`. So you can filter desserts like this:
```zsh
$ grep 'ssh_honeypot' /var/log/secure
...
Jun  4 23:29:16 VM-4-11-centos sshd[20250]: ssh_honeypot: user=`cirros` password=`cubswin:)` host=`XX.XX.XX.XX` port=`40572`
Jun  4 23:29:19 VM-4-11-centos sshd[20263]: ssh_honeypot: user=`cirros` password=`gocubsgo` host=`XX.XX.XX.XX` port=`40938`
...
```

requirements
------------
packet source codes into rpm package
- rpmbuild

Some development packages used to build rpm package:
- audit-libs-devel
- groff
- pam-devel
- tcp_wrappers-devel
- fipscheck-devel
- systemd-devel
- libedit-devel
- xauth   
- ...

Usage
-----
1) Get openssh-server source package from registory such as [pkgs.org](https://centos.pkgs.org/7/centos-x86_64/openssh-server-7.4p1-21.el7.x86_64.rpm.html).
```zsh
$ wget -c "http://vault.centos.org/7.9.2009/os/Source/SPackages/openssh-7.4p1-21.el7.src.rpm"
# Now `openssh-7.4p1-21.el7.src.rpm` is saved in current working directory.
```
2) Unzip source package.
```zsh
$ mkdir openssh-project && cd openssh-project      # Create working space for openssh

$ rpm2cpio ../openssh-7.4p1-21.el7.src.rpm | cpio -id # Source files will be extracted to `openssh` directory
...

$ ls -l   # Check
total 6296
...
```

3) Build rpmbuild working space.
```zsh
$ rpmbuild -ba ./openssh.spec
error: File /root/rpmbuild/SOURCES/pam_ssh_agent_auth-0.10.3.tar.bz2: No such file or directory
# Now rpmbuild will automatically create a working space `rpmbuild` which is told in error message beyond.

$ mv * ../rpmbuild/SOURCES/ && cd ../rpmbuild/SOURCES/
```

4) Unzip openssh-`your-version-here`.tar.gz.
```zsh
$ tar -zxvf openssh-7.4p1.tar.gz
```

5) Get ssh_honeypot.patch from this repo and patch `openssh-7.4p1/auth-passwd.c`.
```zsh
$ pwd
/root/rpmbuild/SOURCES/openssh-7.4p1

$ patch < ssh_honeypot.patch # Download ssh_honeypot.patch to current directory.
patching file auth-passwd.c
```

6) Rebuild rpm package.
```zsh
$ rpmbuild -ba ../openssh.spec
# it may take some time...
```

7) Reinstall openssh-server from new rpm package.
```zsh
$ cd ../../RPMS/x86_64 && ls -l
total 5388
-rw-r--r-- 1 root root  520952 Jun  4 18:04 openssh-7.4p1-21.el7.x86_64.rpm
...
-rw-r--r-- 1 root root  468980 Jun  4 18:04 openssh-server-7.4p1-21.el7.x86_64.rpm
...

$ rpm --reinstall ./openssh-7.4p1-21.el7.x86_64.rpm

$ rpm --reinstall ./openssh-server-7.4p1-21.el7.x86_64.rpm
```

8) Restart `sshd.service`.
```zsh
$ systemctl restart sshd.service
```
**Ooops! Everything is ready and enjoy it.**