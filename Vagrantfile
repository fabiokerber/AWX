Vagrant.configure('2') do |config|

  $script_awx = <<-SCRIPT
    sudo bash -c 'echo "LC_ALL=en_US.utf8" >> /etc/default/locale
    sudo bash -c 'echo "LANG=en_US.utf8" >> /etc/default/locale
    sudo bash -c 'echo "LC_ALL=en_US.utf8" >> /etc/environment
    sudo bash -c 'echo "LANG=en_US.utf8" >> /etc/environment
    sudo bash -c 'echo "locales locales/default_environment_locale select en_US.UTF-8" | debconf-set-selections'
    sudo bash -c 'echo "locales locales/locales_to_be_generated multiselect en_US.UTF-8 UTF-8" | debconf-set-selections'
    sudo bash -c 'rm "/etc/locale.gen"'
    sudo bash -c 'dpkg-reconfigure --frontend noninteractive locales'
    sudo bash -c 'echo "let g:skip_defaults_vim = 1" >> /etc/vim/vimrc'
  SCRIPT

  config.env.enable
  config.vm.boot_timeout = 1200

  if Vagrant.has_plugin?("vagrant-vbguest")
      config.vbguest.auto_update = false
  end

  config.vm.define 'master' do |master|

    master.vm.provider 'virtualbox' do |vb|
      vb.memory = "4096"
      vb.cpus = 4
      vb.name = 'awx-master'
      vb.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
    end

    master.disksize.size = '60GB'
    master.vm.box = 'debian/bullseye64'
    master.vm.hostname = 'awx-master'
    master.vm.network 'private_network', ip: ENV['MASTER_IP']

    master.vm.provision 'shell', inline: 'echo "$IP awx-agent" >> /etc/hosts', env: {"IP" => ENV['AGENT_IP']}
    master.vm.provision 'shell', inline: $script_awx
    master.vm.provision :reload

    master.vm.provision 'shell', inline: 'sudo timedatectl set-timezone $TZ', env: {"TZ" => ENV['TIMEZONE']}
    master.vm.provision 'shell', inline: 'sudo apt update'
    master.vm.provision 'shell', inline: 'sudo apt install -y ca-certificates curl gnupg lsb-release git curl vim'

    master.vm.provision 'shell', inline: 'sudo mkdir -m 0755 -p /etc/apt/keyrings'
    master.vm.provision 'shell', inline: 'curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg'
    master.vm.provision 'shell', inline: 'echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null'
    master.vm.provision 'shell', inline: 'sudo apt update'
    master.vm.provision 'shell', inline: 'sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin'
    master.vm.provision 'shell', inline: 'sudo systemctl enable docker'
    master.vm.provision 'shell', inline: 'sudo usermod -aG docker vagrant'

    master.vm.provision 'shell', inline: 'curl -sfL https://get.k3s.io | sh -s - --docker'
    master.vm.provision 'shell', inline: 'wget https://get.helm.sh/$VHELM -P /tmp/', env: {"VHELM" => ENV['KUBERNETES_HELM']}
    master.vm.provision 'shell', inline: 'tar -xf /tmp/$VHELM -C /tmp/', env: {"VHELM" => ENV['KUBERNETES_HELM']}
    master.vm.provision 'shell', inline: 'sudo mv /tmp/linux-386/helm /usr/local/bin/helm'
    master.vm.provision 'shell', inline: 'helm repo add stable https://charts.helm.sh/stable'
    master.vm.provision 'shell', inline: 'helm repo update'
    
    master.vm.provision 'shell', inline: 'curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash'
    master.vm.provision 'shell', inline: 'chmod 755 kustomize'
    master.vm.provision 'shell', inline: 'sudo mv kustomize /usr/local/bin'
    master.vm.provision 'shell', inline: 'cp /vagrant/files/* /tmp'
    master.vm.provision 'shell', inline: 'sed -i "s/op_tag/$OP_TAG/g" /tmp/kustomization.yaml ', env: {"OP_TAG" => ENV['AWX_OPERATOR']}
    master.vm.provision 'shell', inline: 'cd /tmp && sudo kustomize build . | sudo kubectl apply -f -'
    master.vm.provision 'shell', inline: 'sed -i "6 i \  \- awx-local.yaml" /tmp/kustomization.yaml'
    master.vm.provision 'shell', inline: 'cd /tmp && sudo kustomize build . | sudo kubectl apply -f -'

  end

  config.vm.define 'agent' do |agent|

    agent.vm.provider 'virtualbox' do |vb|
      vb.memory = 1024
      vb.cpus = 2
      vb.name = 'jenkins-agent'
      vb.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']

    end

    agent.vm.box = 'debian/bullseye64'
    agent.vm.hostname = 'awx-agent'
    agent.vm.network 'private_network', ip: ENV['AGENT_IP']
    agent.vm.provision 'shell', inline: 'echo "$IP jenkins-master" >> /etc/hosts', env: {"IP" => ENV['MASTER_IP']}
    agent.vm.provision 'shell', inline: 'sudo timedatectl set-timezone $TZ', env: {"TZ" => ENV['TIMEZONE']}
    agent.vm.provision 'shell', inline: 'sudo apt update'
    agent.vm.provision 'shell', inline: 'sudo mkdir /root/.ssh'
    agent.vm.provision 'shell', inline: 'sudo cp /vagrant/keys/id_rsa.pub /root/.ssh/authorized_keys'

  end

end