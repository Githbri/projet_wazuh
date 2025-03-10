Vagrant.configure(2) do |config|

  # Utilise Ubuntu 22.04 comme base
  config.vm.box = "ubuntu/jammy64"

  # Définit une seule machine nommée "serverbri"
  config.vm.define "serverbri" do |cfg|
    # Nom de la machine
    cfg.vm.hostname = "serverbri"
    
    # Configure une IP réseau privée
    cfg.vm.network "private_network", ip: "10.0.0.101"

    # Configuration des ressources VirtualBox
    cfg.vm.provider "virtualbox" do |v|
      v.memory = 4096       # 4 Go de RAM
      v.cpus = 4            # 4 processeurs
    end

    # Ajoute le script pour le logo
    cfg.vm.provision :shell, path: "install_sabri.sh"
  end

end

