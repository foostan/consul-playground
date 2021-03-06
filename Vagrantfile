# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

CONSUL_SERVER_NUM = 3
CONSUL_CLIENT_NUM = 1

IP_BASE = "192.168.33."

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"

  (1..CONSUL_SERVER_NUM).each do |i|
    node_name = "consul-server-#{i}"
    config.vm.define vm_name = node_name do |server|
      ip = "#{IP_BASE}#{i+10}"
      if i == 1
        setup(server, node_name, ip, true, true)
      else
        setup(server, node_name, ip, true, false)
      end
    end
  end

  (1..CONSUL_CLIENT_NUM).each do |i|
    node_name = "consul-client-#{i}"
    config.vm.define vm_name = node_name do |client|
      ip = "#{IP_BASE}#{i+20}"
      setup(client, node_name, ip, false, false)
    end
  end
end

def setup(config, hostname, ip, server, bootstrap)
  config.vm.hostname = hostname
  config.vm.network :private_network, ip: ip

  consul_opt_server = server ? '-server' : ''
  consul_opt_bootstrap = bootstrap ? '-bootstrap-expect 1' : ''

  config.vm.provision "docker" do |d|
    d.run "progrium/consul",
      cmd: "#{consul_opt_server} #{consul_opt_bootstrap} -advertise #{ip} -join #{IP_BASE}11",
      args: "-h consul-#{hostname} -p 8300:8300 -p 8301:8301/tcp -p 8301:8301/udp -p 8302:8302/tcp -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 8600:53/udp"

    d.run "progrium/registrator",
      cmd: "consul://#{ip}:8500",
      args: "-h registrator-#{hostname} -v /var/run/docker.sock:/tmp/docker.sock"

    d.run "foostan/docker-metrics-to-consul",
      args: "-h metrics-#{hostname} -v '/var/run/docker.sock:/var/run/docker.sock' -v '/sys/fs/cgroup/:/sys/fs/cgroup:ro' -e 'CONSUL_URI=http://#{ip}:8500' -e 'KV_PREFIX=docker_metrics'"

    d.run "foostan/haproxy-with-consul",
      args: "-h proxy-#{hostname} -p 80 -p 8080 -e 'CONSUL_URI=#{ip}:8500'"

    d.run "foostan/tinyweb",
      args: "-h blue-tinyweb-#{hostname} -p 80 -e 'SERVICE_TAGS=blue' -e 'SERVICE_NAME=webapp'"

    d.run "foostan/tinyweb",
      args: "-h green-tinyweb-#{hostname} -p 80 -e 'SERVICE_TAGS=green' -e 'SERVICE_NAME=webapp'"
  end
end
