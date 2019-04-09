0. Create bridge network interface
```
# netplan.yml
network
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
3. ???
4. Profit
