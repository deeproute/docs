# Linux General Commands

## Host Management

### Check uptime all worker nodes
```
MY_CLUSTER=mycluster
MY_TYPE=w
MY_SSH_TIMEOUT=1
for node in {01..28} ; \
do MY_HOST="${MY_CLUSTER}${MY_TYPE}${node}" ; \
echo "$MY_HOST" ; \
ssh -o ConnectTimeout=$MY_SSH_TIMEOUT -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" "core@$MY_HOST" uptime ; \
done
```

### Shutdown all nodes
```
MY_CLUSTER=mycluster
MY_TYPE=w
MY_SSH_TIMEOUT=1
for node in {01..28} ; \
do MY_HOST="${MY_CLUSTER}${MY_TYPE}${node}" ; \
echo "$MY_HOST" ; \
ssh -o ConnectTimeout=$MY_SSH_TIMEOUT -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" "core@$MY_HOST" sudo shutdown now ; \
done

```
### How much time the machine was on
```
$ uptime

 08:25:51 up 1 min,  2 users,  load average: 3.42, 0.95, 0.33
```

## File Management

### Copy files excluding ".folder" names

```
rsync -av --exclude=".*" src dest
```

### Grep for word "keyword" and show the 5 lines before and 2 lines after. Also highlight it with a color.
```
grep -B 5 -A 2 --color 'keyword' /path/to/file.log
```

## User Management
### Check the current user ids
```
id
uid=0(root) gid=0(root) groups=0(root),1337
```