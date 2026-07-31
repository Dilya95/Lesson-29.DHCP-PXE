Vagrant.configure("2") do |config|
  
  # PXE Server
  config.vm.define "pxeserver" do |server|
    server.vm.box = 'bento/ubuntu-26.04'
    server.vm.host_name = 'pxeserver'
    
    server.vm.network "forwarded_port", guest: 80, host: 8080
    server.vm.network :private_network,
      ip: "10.0.0.20",
      virtualbox__intnet: 'pxenet'
    server.vm.network :private_network, 
      ip: "192.168.50.10", 
      adapter: 3
    
    server.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
    
    #server.vm.provision "ansible" do |ansible|
    #  ansible.playbook = "ansible/provision.yml"
    #  ansible.inventory_path = "ansible/hosts"
    #  ansible.host_key_checking = "false"
    #  ansible.limit = "all"
    #end
  end
  
  # PXE Client
  config.vm.define "pxeclient" do |pxeclient|
    pxeclient.vm.box = 'bento/ubuntu-26.04'
    # pxeclient.vm.box = 'seskion/ubuntu-20.04-efi'  # Альтернативный вариант с EFI
    pxeclient.vm.host_name = 'pxeclient'
    
    pxeclient.vm.network :private_network, 
      ip: "10.0.0.21"
    
    pxeclient.vm.provider :virtualbox do |vb|
      vb.memory = "6144"

    vb.customize ["modifyvm", :id, "--firmware", "bios"]
      
      # Настройка сетевых интерфейсов и порядка загрузки
      vb.customize [
        'modifyvm', :id,
        '--nic1', 'intnet',
        '--intnet1', 'pxenet',
        '--nic2', 'none',
        '--boot1', 'net',
        '--boot2', 'none',
        '--boot3', 'none',
        '--boot4', 'none'
      ]
      
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
  end
  
end
