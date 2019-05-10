# Create
-1. Disable NetworkManager
```
sudo apt-get purge -y network-manager
```
0. Create bridge network interface
```
# netplan.yml
network
  renderer: networkd
  ...
  bridges:
    br0:
      interfaces:
        - enp0s31f6
      addresses: [192.168.1.3/24]
      dhcp4: false
      parameters:
        stp: false
        forward-delay: 0
  ...
```
1. Create network
```
docker network create --driver=bridge --ip-range=192.168.1.192/27 --subnet=192.168.1.0/24 --gateway=192.168.1.3 --attachable -o "com.docker.network.bridge.name=br0" --aux-address="DefaultGatewayIPv4=192.168.1.1" br0
```
2. Create test container
```
docker run --rm -it --network=br0 --ip=192.168.1.200 nginx
```
3. Create gitlab container
```
docker run --detach \
  --name gitlab \
  --hostname gitlab.zag \
  --network br0 \
  --ip 192.168.1.200 \
  --publish 22:22 \
  --publish 80:80 \
  --publish 443:443 \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'http://gitlab.zag/';" \
  gitlab/gitlab-ce:latest
```
4. ???
5. Profit


# Backup & Restore
```
# backup
docker exec -t gitlab gitlab-rake gitlab:backup:create
docker cp gitlab:/var/opt/gitlab/backups/<backup>.tar .

# restore
docker cp <backup>.tar gitlab:/var/opt/gitlab/backups/
docker exec -it gitlab gitlab-rake gitlab:backup:restore
```

