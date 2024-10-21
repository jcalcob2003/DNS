Vagrant.configure("2") do |config|
    # Configuración del servidor maestro
    config.vm.define "dns_master" do |master|
      master.vm.box = "debian/bullseye64"
      master.vm.hostname = "tierra.sistema.test"
      master.vm.network "private_network", ip: "192.168.57.103"
      master.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y bind9 bind9utils bind9-doc
        # Configuración adicional para maestro aquí
      SHELL
    end
    
    # Configuración del servidor esclavo
    config.vm.define "dns_slave" do |slave|
      slave.vm.box = "debian/bullseye64"
      slave.vm.hostname = "venus.sistema.test"
      slave.vm.network "private_network", ip: "192.168.57.102"
      slave.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y bind9 bind9utils bind9-doc
        # Configuración adicional para esclavo aquí
      SHELL
    end
  end