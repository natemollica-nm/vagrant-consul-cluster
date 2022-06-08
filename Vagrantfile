# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

CONSUL_VERSION="1.11.6+ent"
ENVOY_VERSION="1.20.2"
LAN_IP_DC1="20.0.0"
LAN_IP_DC2="20.1.0"
WAN_IP="192.168.0"
MAC_NETWORK_BRIDGE="en8: AX88179A"

$router = <<-ROUTER
echo "[+] Configuring ipv4 forwarding for .1 IPs (/etc/sysctl.conf)...."
echo -e "net.ipv4.ip_forward=1" | sudo tee --append /etc/sysctl.conf
ROUTER

$node = <<-NODE
echo "[+] Disabling ICMP ipv4 routing (/etc/sysctl.conf)...."
echo -e "net.ipv4.conf.all.send_redirects=0" | sudo tee --append /etc/sysctl.conf
NODE

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "hashicorp/bionic64" # Ubuntu 18.04 LTS 64-bit Box
  config.vm.provider "virtualbox" do |v|
    v.cpus = 6
    v.memory = "2048"
   end

    config.vm.define "consul-cluster-router" do |router|
      router.vm.network "public_network", ip: "#{LAN_IP_DC1}.1", bridge: "#{MAC_NETWORK_BRIDGE}"
      router.vm.network "public_network", ip: "#{LAN_IP_DC2}.1", bridge: "#{MAC_NETWORK_BRIDGE}"
      router.vm.provision "shell", inline: $router, run: "once"
      router.vm.provision "shell", path: "scripts/vagrant-routing/ubuntu-router-configure.sh", run: "always"
      router.vm.provision :reload
    end

    config.vm.define "consul-external-test" do |external|
      external.vm.hostname = "consul-external-test"
      external.vm.network "public_network", ip: "#{LAN_IP_DC1}.5", bridge: "#{MAC_NETWORK_BRIDGE}"
      external.vm.network "public_network", ip: "#{WAN_IP}.5", bridge: "#{MAC_NETWORK_BRIDGE}"
      external.vm.provision "shell", inline: $node, run: "once"
      external.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
        args: "'--local-lan' #{LAN_IP_DC1} '--remote-lan' #{LAN_IP_DC2} '--wan-net' #{WAN_IP}"
      external.vm.provision "file", source: "consul/services/counting-service_linux_amd64", destination: "$HOME/counting-service_linux_amd64"
      external.vm.provision :reload
    end

    config.vm.define "consul-dc1-server-0" do |consul0|
        consul0.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc1' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
        consul0.vm.hostname = "consul-dc1-server-0"
        consul0.vm.network "public_network", ip: "#{LAN_IP_DC1}.10", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul0.vm.network "public_network", ip: "#{WAN_IP}.100", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul0.vm.provision "shell", inline: $node, run: "once"
        consul0.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        consul0.vm.network "forwarded_port", guest: 8500, host: 8500, protocol: "tcp"
        consul0.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC1} '--remote-lan' #{LAN_IP_DC2} '--wan-net' #{WAN_IP}"
        consul0.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        consul0.vm.provision :reload
    end

    config.vm.define "consul-dc1-server-1" do |consul1|
        consul1.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc1' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
        consul1.vm.hostname = "consul-dc1-server-1"
        consul1.vm.network "public_network", ip: "#{LAN_IP_DC1}.20", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul1.vm.network "public_network", ip: "#{WAN_IP}.150", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul1.vm.provision "shell", inline: $node, run: "once"
        consul1.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        consul1.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC1} '--remote-lan' #{LAN_IP_DC2} '--wan-net' #{WAN_IP}"
        consul1.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        consul1.vm.provision :reload
    end

    config.vm.define "consul-dc1-server-2" do |consul2|
        consul2.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc1' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
        consul2.vm.hostname = "consul-dc1-server-2"
        consul2.vm.network "public_network", ip: "#{LAN_IP_DC1}.30", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul2.vm.network "public_network", ip: "#{WAN_IP}.160", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul2.vm.provision "shell", inline: $node, run: "once"
        consul2.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        consul2.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC1} '--remote-lan' #{LAN_IP_DC2} '--wan-net' #{WAN_IP}"
        consul2.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        consul2.vm.provision :reload
    end

    config.vm.define "consul-dc1-server-3" do |consul3|
        consul3.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc1' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
        consul3.vm.hostname = "consul-dc1-server-3"
        consul3.vm.network "public_network", ip: "#{LAN_IP_DC1}.40", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul3.vm.network "public_network", ip: "#{WAN_IP}.165", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul3.vm.provision "shell", inline: $node, run: "once"
        consul3.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        consul3.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC1} '--remote-lan' #{LAN_IP_DC2} '--wan-net' #{WAN_IP}"
        consul3.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        consul3.vm.provision :reload
    end

    config.vm.define "consul-dc1-mesh-gw" do |primary_mesh|
        primary_mesh.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc1' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
        primary_mesh.vm.hostname = "consul-dc1-mesh-gw"
        primary_mesh.vm.network "public_network", ip: "#{LAN_IP_DC1}.55", bridge: "#{MAC_NETWORK_BRIDGE}"
        primary_mesh.vm.network "public_network", ip: "#{WAN_IP}.155", bridge: "#{MAC_NETWORK_BRIDGE}"
        primary_mesh.vm.provision "shell", inline: $node, run: "once"
        primary_mesh.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        primary_mesh.vm.network "forwarded_port", guest: 19000, host: 19001, protocol: "tcp"
        primary_mesh.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC1} '--remote-lan' #{LAN_IP_DC2} '--wan-net' #{WAN_IP}"
        primary_mesh.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        primary_mesh.vm.provision :reload
    end

    config.vm.define "consul-dc1-client-0" do |client0|
        client0.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc1' '--enable-acls' '--set-gossip-encryption' '--set-rpc-encryption'"
        client0.vm.hostname = "consul-dc1-client-0"
        client0.vm.network "public_network", ip: "#{LAN_IP_DC1}.2", bridge: "#{MAC_NETWORK_BRIDGE}"
        client0.vm.network "public_network", ip: "#{WAN_IP}.200", bridge: "#{MAC_NETWORK_BRIDGE}"
        client0.vm.provision "shell", inline: $node, run: "once"
        client0.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        client0.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC1} '--remote-lan' #{LAN_IP_DC2} '--wan-net' #{WAN_IP}"
        client0.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        client0.vm.provision :reload
    end

    config.vm.define "consul-dc1-client-1" do |client1|
        client1.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc1' '--enable-acls' '--set-gossip-encryption' '--set-rpc-encryption'"
        client1.vm.hostname = "consul-dc1-client-1"
        client1.vm.network "public_network", ip: "#{LAN_IP_DC1}.3", bridge: "#{MAC_NETWORK_BRIDGE}"
        client1.vm.network "public_network", ip: "#{WAN_IP}.201", bridge: "#{MAC_NETWORK_BRIDGE}"
        client1.vm.provision "shell", inline: $node, run: "once"
        client1.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        client1.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC1} '--remote-lan' #{LAN_IP_DC2} '--wan-net' #{WAN_IP}"
        client1.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        client1.vm.provision :reload
    end

    config.vm.define "consul-dc2-server-0" do |consul20|
        consul20.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc2' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
        consul20.vm.hostname = "consul-dc2-server-0"
        consul20.vm.network "public_network", ip: "#{LAN_IP_DC2}.10", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul20.vm.network "public_network", ip: "#{WAN_IP}.170", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul20.vm.provision "shell", inline: $node, run: "once"
        consul20.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        consul20.vm.network "forwarded_port", guest: 8501, host: 8501, protocol: "tcp"
        consul20.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC2} '--remote-lan' #{LAN_IP_DC1} '--wan-net' #{WAN_IP}"
        consul20.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        consul20.vm.provision :reload
    end

    config.vm.define "consul-dc2-server-1" do |consul21|
        consul21.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc2' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
        consul21.vm.hostname = "consul-dc2-server-1"
        consul21.vm.network "public_network", ip: "#{LAN_IP_DC2}.20", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul21.vm.network "public_network", ip: "#{WAN_IP}.180", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul21.vm.provision "shell", inline: $node, run: "once"
        consul21.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        consul21.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC2} '--remote-lan' #{LAN_IP_DC1} '--wan-net' #{WAN_IP}"
        consul21.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        consul21.vm.provision :reload
    end

    config.vm.define "consul-dc2-server-2" do |consul22|
        consul22.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc2' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
        consul22.vm.hostname = "consul-dc2-server-2"
        consul22.vm.network "public_network", ip: "#{LAN_IP_DC2}.30", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul22.vm.network "public_network", ip: "#{WAN_IP}.190", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul22.vm.provision "shell", inline: $node, run: "once"
        consul22.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        consul22.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC2} '--remote-lan' #{LAN_IP_DC1} '--wan-net' #{WAN_IP}"
        consul22.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        consul22.vm.provision :reload
    end

    config.vm.define "consul-dc2-server-3" do |consul23|
        consul23.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc2' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
        consul23.vm.hostname = "consul-dc2-server-3"
        consul23.vm.network "public_network", ip: "#{LAN_IP_DC2}.40", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul23.vm.network "public_network", ip: "#{WAN_IP}.195", bridge: "#{MAC_NETWORK_BRIDGE}"
        consul23.vm.provision "shell", inline: $node, run: "once"
        consul23.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        consul23.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC2} '--remote-lan' #{LAN_IP_DC1} '--wan-net' #{WAN_IP}"
        consul23.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        consul23.vm.provision :reload
    end

    config.vm.define "consul-dc2-mesh-gw" do |secondary_mesh|
      secondary_mesh.vm.provision "shell", path: "prov/install-consul", run: "once",
          args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc2' '--enable-acls' '--enable-consul-connect' '--envoy-version' #{ENVOY_VERSION} '--set-gossip-encryption' '--set-rpc-encryption'"
      secondary_mesh.vm.hostname = "consul-dc2-mesh-gw"
      secondary_mesh.vm.network "public_network", ip: "#{LAN_IP_DC2}.55", bridge: "#{MAC_NETWORK_BRIDGE}"
      secondary_mesh.vm.network "public_network", ip: "#{WAN_IP}.185", bridge: "#{MAC_NETWORK_BRIDGE}"
      secondary_mesh.vm.provision "shell", inline: $node, run: "once"
      secondary_mesh.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
      secondary_mesh.vm.network "forwarded_port", guest: 19000, host: 19002, protocol: "tcp"
      secondary_mesh.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
        args: "'--local-lan' #{LAN_IP_DC2} '--remote-lan' #{LAN_IP_DC1} '--wan-net' #{WAN_IP}"
      secondary_mesh.vm.provision "file", source: "consul/services/", destination: "$HOME/"
      secondary_mesh.vm.provision :reload
    end

    config.vm.define "consul-dc2-client-0" do |client20|
        client20.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc2' '--enable-acls' '--set-gossip-encryption' '--set-rpc-encryption'"
        client20.vm.hostname = "consul-dc2-client-0"
        client20.vm.network "public_network", ip: "#{LAN_IP_DC2}.2", bridge: "#{MAC_NETWORK_BRIDGE}"
        client20.vm.network "public_network", ip: "#{WAN_IP}.202", bridge: "#{MAC_NETWORK_BRIDGE}"
        client20.vm.provision "shell", inline: $node, run: "once"
        client20.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        client20.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC2} '--remote-lan' #{LAN_IP_DC1} '--wan-net' #{WAN_IP}"
        client20.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        client20.vm.provision :reload
    end

    config.vm.define "consul-dc2-client-1" do |client21|
        client21.vm.provision "shell", path: "prov/install-consul", run: "once",
            args: "'--version' #{CONSUL_VERSION} '--datacenter' 'dc2' '--enable-acls' '--set-gossip-encryption' '--set-rpc-encryption'"
        client21.vm.hostname = "consul-dc2-client-1"
        client21.vm.network "public_network", ip: "#{LAN_IP_DC2}.3", bridge: "#{MAC_NETWORK_BRIDGE}"
        client21.vm.network "public_network", ip: "#{WAN_IP}.203", bridge: "#{MAC_NETWORK_BRIDGE}"
        client21.vm.provision "shell", inline: $node, run: "once"
        client21.vm.provision "shell", path: "scripts/install-dnsmasq/install-dnsmasq", run: "once"
        client21.vm.provision "shell", path: "scripts/vagrant-routing/consul-node-routing.sh", run: "always",
            args: "'--local-lan' #{LAN_IP_DC2} '--remote-lan' #{LAN_IP_DC1} '--wan-net' #{WAN_IP}"
        client21.vm.provision "file", source: "consul/services/", destination: "$HOME/"
        client21.vm.provision :reload
    end
 end
