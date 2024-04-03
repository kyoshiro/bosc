# Cheatsheet

List of commands for several tools

## Commands for Linux in general

### LV resizeing / extending free space

`fdisk /dev/<pvname>`

`partprobe`

`pvresize /dev/<pvname>`

`lvresize /dev/mapper/<lvname> -l +100%FREE -r`

## Commands for Gentoo Linux

### Mount and chroot into gentoo box
`mount /dev/mapper/vg_gentoo-root /mnt/gentoo`

`mount /dev/sda2 /mnt/gentoo/boot`

`mount /dev/sda1 /mnt/gentoo/boot/efi`

`mount -t proc /proc /mnt/gentoo/proc`

`mount -rbind /dev /mnt/gentoo/dev`

`mount -rbind /sys /mnt/gentoo/sys`

`chroot /mnt/gentoo /bin/bash`

`env-update && source /etc/profile`

## Commands for Docker

### docker-compose up / down
`docker-compose up -d`

`docker-compose down`

### docker list running containers
`docker ps`

### docker list all containers also stopped ones
`docker ps -a`

### docker remove dangling images
`docker image prune`

### docker remove all unused images not only dangling ones
`docker image prune -a`

### docker remove image from registry via curl

```
registry='<registry host>'
name='<image name>'
auth='-u <username>:<password>'
tag='<tag>'
curl $auth -X DELETE -sI -k "https://${registry}/v2/${name}/manifests/$(
  curl $auth -sI -k \
    -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
    "https://${registry}/v2/${name}/manifests/${tag}" \
    | tr -d '\r' | sed -En 's/^Docker-Content-Digest: (.*)/\1/pi'
)"
```

## Commands for Kubernetes

### kubectl apply / delete / diff
`kubectl --kubeconfig ~/.kube/<kubeconfig> apply|delete|diff -f <config.yaml>`

### kubectl copy local to remote
`kubectl --kubeconfig ~/.kube/<kubeconfig> cp ./<localfile> pod-name:/<remotefile>`

### kubectl copy remote to local
`kubectl --kubeconfig ~/.kube/kubeconfig cp pod-name:/<remotefile> ./<localfile>`

## Commands for Helm

### helm install
`helm --kubeconfig ~/.kube/<kubeconfig> install <release-name> -n <namespace> -f ./values.yaml ./<chart-directory> --dry-run`

`helm --kubeconfig ~/.kube/<kubeconfig> install <release-name> -n <namespace> -f ./values.yaml ./<chart-directory>`

### helm update
`helm --kubeconfig ~/.kube/<kubeconfig> upgrade --install <release-name> -n <namespace> -f ./values.yaml ./<chart-directory> --dry-run`

`helm --kubeconfig ~/.kube/<kubeconfig> upgrade --install <release-name> -n <namespace> -f ./values.yaml ./<chart-directory>`

### helm uninstall
`helm --kubeconfig ~/.kube/<kubeconfig> uninstall <release-name> -n <namespace> --dry-run`

`helm --kubeconfig ~/.kube/<kubeconfig> uninstall <release-name> -n <namespace>`


## Commands for Terraform

### Init
`terraform init`

### Plan and Apply optional with target module
`terraform plan [--target='module.<modulename>'] --out plan[.<modulename>].out`

`terraform apply plan[.<modulename>].out`


## Commands for Git

### Revert branch to specific commit

`git reset --hard <commit-hash>`

`git push -f origin master`
### Change commit message
`git commit --amend`
### Change commits author name / email address
`git commit --amend --author "Your Name <your_email@address.com>" --no-edit`

`git rebase --continue`

### Change / Sqash commit messages

`git rebase --interactive <commit-hash>`

replace all commits from `pick` to `sqash` to remove the messages

add a new commit message

### Change History username / email on bulk

replace  NEW_NAME, NEW_EMAIL_ADDRESS, OLD_NAME, OLD_EMAIL_ADDRESS

`git filter-branch -f --env-filter "GIT_AUTHOR_NAME='NEW_NAME'; GIT_AUTHOR_EMAIL='NEW_EMAIL_ADDRESS'; GIT_COMMITTER_NAME='OLD_NAME'; GIT_COMMITTER_EMAIL='OLD_EMAIL_ADDRESS';" HEAD`

## Commands for MySQL

### Processlist with dump to file for further analysis
`mysql -u sqladmin -h dtor22-s1-mysqldb.mysql.database.azure.com -p -e "SHOW PROCESSLIST" | tee process_list.txt`
`mysql -u sqladmin -h dtor22-s1-mysqldb.mysql.database.azure.com -p -e "select count(host), user, command from information_schema.processlist group by user, command;" | tee process_list.txt`

### Processlist with watch
`watch 'mysql -e "show full processlist;"'`

### Processlist with watch when password is needed
`watch -n 1 'mysql -p<password-here> -e "show full processlist;"'`

### List processes with specific execution time (e.g. >30 sec.)
`SELECT user,db,command,time,info FROM information_schema.processlist WHERE time > 30;`

### Galera status, e.g. for donor/sync, cluster size
`show status like '%wsrep%';`

### Show status of engine InnoDB, e.g. for deadlock Info:
`show engine innodb status`

### Sed Operations, e.g. replace in all files:
`find . -exec sed -i -e 's/querytext/replacetext/g' {} \;`

### Oneliner for git glab operations:
`for dir in (ls); cd $dir && git st | grep "v1" && git co main && git co . && glab release create 2.0.0 -N "- [+] Migration from shared modules" && cd .. || cd ..; end`
`for dir in (ls); echo $dir && cd $dir && glab release create 1.0.0 -N "- [+] Migration from shared modules" && cd .. || cd ..; end`

## Clear DNS cache on Mac

`sudo dscacheutil -flushcache`

## Filter Accesslog and show IPs:
`grep "<somefilter_e.g._date>" access.log | awk '{print $1}' | sort -n | uniq -c | sort -n | tail -3`

## IP Filtering with iptables:
Insert: `iptables -I INPUT -s 123.45.6.7 -j DROP`
Delete: `iptables -D INPUT -s 123.45.6.7 -j DROP`

### Calculate Subnets:

cidrsubnet(prefix, newbits, netnum)

newbits -> extend subnetmask, e.g. /16 + newbits of 4, -> /20

netnum -> factor for subnet starting ip, e.g.:
base net: 10.0.0.0/16
newbits: 4
subnet calculated: 10.0.0.0/20
netnum: 4
new subnet position calculated: 10.0.64.0/20, because each /20 has a subnet range of 4096 ips:
first (0): 10.0.0.0/20
second (1): 10.0.16.0/20
third (2): 10.0.32.0/20
fourth (3): 10.0.48.0/20
fifth (4): 10.0.64.0/20

## RAID configuration:

### Create SSD/HHD raid configuration with sda3 (ssd) as read prefered and sdb3 (hdd) write behind for redundancy
`mdadm --create /dev/md0 -level=1 -raid-disks=2 --bitmap=internal /dev/sda3 --write-behind --write-mostly /dev/sdb3`

### Create raid from existing drive with one drive missing at beginning

`mdadm --create /dev/md3 --level=1 --raid-disks=2 /dev/sdb3 missing`

### Examine raid status

`cat /proc/mdstat`
`mdadm --detail /dev/md0`
`mdadm –examine –scan`
`lsblk`

### Remove old raid superblock from raid drive/partition

`mdadm –zero-superblock /dev/sdb3`

### Add drive to raid for resync

`mdadm –add /dev/md0 /dev/sdb3`

### Change write-behind and write-mostly status of raid disk

`echo writemostly > /sys/block/md0/md/dev-sdb3/state`
`echo writebehind > /sys/block/md0/md/dev-sdb3/state`

`echo -writemostly > /sys/block/md0/md/dev-sdb3/state`
`echo -writebehind > /sys/block/md0/md/dev-sdb3/state`

## NixOS commands

### List generations of user
`nix-env  --list-generations`

### Remove all generations of user except current
`nix-collect-garbage -d`

### Update Channel
`nix-channel --update; nix-env -iA nixpkgs.nix`

