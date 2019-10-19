# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT

echo "Installing dependencies ..."
echo "deb http://deb.debian.org/debian/ unstable main" | sudo tee /etc/apt/sources.list.d/unstable.list
printf 'Package: *\nPin: release a=unstable\nPin-Priority: 90\n' | sudo tee /etc/apt/preferences.d/limit-unstable
sudo apt-get update
sudo apt-get install -y unzip curl jq dnsutils vim htop net-tools wireguard linux-headers-$(uname -r)

echo "Determining Consul version to install ..."
CHECKPOINT_URL="https://checkpoint-api.hashicorp.com/v1/check"
if [ -z "$CONSUL_DEMO_VERSION" ]; then
    CONSUL_DEMO_VERSION=$(curl -s "${CHECKPOINT_URL}"/consul | jq .current_version | tr -d '"')
fi

echo "Fetching Consul version ${CONSUL_DEMO_VERSION} ..."
cd /tmp/
curl -s https://releases.hashicorp.com/consul/${CONSUL_DEMO_VERSION}/consul_${CONSUL_DEMO_VERSION}_linux_amd64.zip -o consul.zip

echo "Installing Consul version ${CONSUL_DEMO_VERSION} ..."
unzip consul.zip
sudo chmod +x consul
sudo mv consul /usr/bin/consul

sudo mkdir -p /etc/consul.d
sudo chmod a+w /etc/consul.d

sudo useradd consul | true

curl -sLO https://github.com/geniousphp/autowire/releases/download/v0.1.2/autowire-v0.1.2-linux-x86_64.zip
unzip autowire-v0.1.2-linux-x86_64.zip
rm autowire-v0.1.2-linux-x86_64.zip
mv autowire-v0.1.2-linux-x86_64 /usr/local/bin/autowire

cat <<EOT > /etc/systemd/system/autowire.service
[Unit]
Description=Autowire
After=network.target consul.service
Requires=network.target

[Service]
ExecStart=/usr/local/bin/autowire -if-name eth1
ExecStop=/usr/bin/wg-quick down wg0
Restart=always
RestartSec=10
User=root

[Install]
WantedBy=multi-user.target
EOT

systemctl daemon-reload
systemctl enable autowire
systemctl restart autowire

SCRIPT

$consul_systemd = <<CONSUL_SYSTEMD

cat <<EOT > /etc/systemd/system/consul.service
[Unit]
Description=Consul
After=network.target
Requires=network.target

[Service]
Environment=CONSUL_UI_BETA=true
ExecStartPre=/bin/mkdir -p /var/lib/consul
ExecStartPre=/bin/chown -R consul. /var/lib/consul
ExecStart=/usr/bin/consul agent -server -retry-join ${CONSUL_BOOTSTRAP_HOST} -client 0.0.0.0 -bootstrap-expect 3 -ui -raft-protocol=3 -datacenter=${DC} -advertise=${HOSTIP} -data-dir=/var/lib/consul -node=${HOSTNAME}
ExecStop=/usr/bin/consul leave
ExecReload=/usr/bin/consul reload
PermissionsStartOnly=true
Restart=always
RestartSec=1
User=consul

[Install]
WantedBy=multi-user.target
EOT

systemctl daemon-reload
systemctl enable consul
systemctl restart consul

CONSUL_SYSTEMD

# Specify a Consul version
CONSUL_DEMO_VERSION = ENV['CONSUL_DEMO_VERSION']

# Specify a custom Vagrant box for the demo
DEMO_BOX_NAME = ENV['DEMO_BOX_NAME'] || "debian/contrib-buster64"

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = DEMO_BOX_NAME

  config.vm.provision "shell", inline: $script, env: {'CONSUL_DEMO_VERSION' => CONSUL_DEMO_VERSION}

  (0..2).each do |i|
    config.vm.define "n#{i}" do |node|
        node.vm.hostname = "n#{i}"
        hostip = "172.20.20.#{10+i}"
        #CONSUL_BOOTSTRAP_HOST ||= hostip
        dc = "nnn"
        CONSUL_BOOTSTRAP_HOST ||= hostip
        consul_bootstrap_host = CONSUL_BOOTSTRAP_HOST
        node.vm.network "private_network", ip: hostip
        node.vm.provision "shell", inline: $consul_systemd, env: {"CONSUL_BOOTSTRAP_HOST": consul_bootstrap_host,"HOSTIP" => hostip, "DC" => dc}
    end
  end

end
