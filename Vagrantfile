Vagrant.configure("2") do |config|
  servers=[
    {
      :hostname => "ansible-control",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.56.200",
      :ssh_port => '2200'
    },
    {
      :hostname => "db01",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.56.201",
      :ssh_port => '2201'
    },
    {
      :hostname => "web01",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.56.202",
      :ssh_port => '2202'
    },
    {
      :hostname => "web02",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.56.203",
      :ssh_port => '2203'
    },
    {
      :hostname => "loadbalancer",
      :box => "bento/ubuntu-22.04",
      :ip => "192.168.56.204",
      :ssh_port => '2204'
    }

  ]

  servers.each do |machine|

    config.vm.define machine[:hostname] do |node|
      node.vm.box = machine[:box]
      node.vm.hostname = machine[:hostname]
    
      node.vm.network :private_network, ip: machine[:ip]
      node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"

      node.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", 1024]
        v.customize ["modifyvm", :id, "--name", machine[:hostname]]
      end
    end
  end

end