# Autowire PoC

Quick PoC to test Autowire

## Usage

### Boot up cluster
```sh
vagrant up
```

### List instances
```sh
vagrant status
```

### Access Consul UI
 * http://172.20.20.10:8500

### Check WG status
Must be logged in onto one instance
```sh
vagrant ssh n0
wg show
```

## References
 * https://github.com/geniousphp/autowire
 * https://www.wireguard.com/
 * https://www.consul.io
 * https://www.vagrantup.com/
