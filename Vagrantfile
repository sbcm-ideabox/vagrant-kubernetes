require 'ipaddr'

begin
  vboxVersion = `vboxmanage --version`

  if Gem::Version.new(vboxVersion) < Gem::Version.new('5.1')
    throw "VirtualBox version too old. Please upgrade to 5.1"
  end
rescue
  print "Could not determine Virtual Box version. Continuing"
end

Vagrant.configure(2) do |config|
  if ENV['NET_CIDR']
    NET_CIDR = ENV['NET_CIDR']
  else
    NET_CIDR = '10.10.0.0/24'
  end

  if ENV['PORTAL_CIDR']
    PORTAL_CIDR = ENV['PORTAL_CIDR']
  else
    PORTAL_CIDR = '10.0.0.0/24'
  end

  NETMASK = IPAddr.new('255.255.255.255').mask(NET_CIDR.split('/')[1]).to_s

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # This is the docker network
  config.vm.network "private_network", ip: IPAddr.new(NET_CIDR.split('/')[0]).mask(NETMASK).succ.succ.to_s, netmask: NETMASK, auto_config: false

  config.vm.post_up_message = "Vagrant Kubernetes Box based on Debian. Check http://10.0.0.3 for the dashboard"

  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "/etc/provision/playbook.yml"
    ansible.become = true
    ansible.compatibility_mode = "2.0"
    ansible.raw_arguments = '--extra-vars "NET_CIDR=' + NET_CIDR + ' PORTAL_CIDR=' + PORTAL_CIDR + '"'
  end

  config.vm.provider :virtualbox do |vb|
    # Set the vboxnet interface to promiscous mode so that the docker veth
    # interfaces are reachable
    vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    # Otherwise we get really slow DNS lookup on OSX (Changed DNS inside the machine)
    # vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]

    # Doesn't really work needs to be set in original Vagrantfile
    # config.ssh.forward_agent = true
  end
end
