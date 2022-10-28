**server configuration**

| name  | ip address    |
| ----- | ------------- |
| node1 | 192.168.0.104 |
| node2 | 192.168.0.105 |

**node1 and node2 make an /etc/hosts entry**

```bash
cat <<EOF >> /etc/hosts
192.168.0.105 node2
192.168.0.104 node1
EOF
hostnamectl set-hostname node2
hostnamectl set-hostname node1
```

**preparation of the pcs clusters on node1 and node2**

```bash
yum update -y
# install packages like httpd, drbd, pcs
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum install -y kmod-drbd84 drbd84-utils -y
yum install pacemaker pcs psmisc policycoreutils-python httpd -y vim
# disble the selinux
setenforce 0
# disable the firewalld
systemctl stop firewalld
systemctl disable firewalld
# Enable the server status for httpd server
cat <<EOF >> /etc/httpd/conf.d/status.conf
<Location /server-status>
    SetHandler server-status
    Require local
</Location>
EOF
```

**prepare the drbd configuration both node1 and node2**

```bash
cat <<EOF >>  /etc/drbd.d/clusterdb.res
