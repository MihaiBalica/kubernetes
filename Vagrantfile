BOX_IMAGE = "generic/ubuntu1804"
#KUBEADM_VERSION = "1.13.5-00"

Vagrant.configure("2") do |config|
  config.vm.provider :libvirt do |v|
    v.memory = 1024
    v.cpus = 2
  end

  config.vm.provision :shell, privileged: true, inline: $install_common_tools

  config.vm.define :master do |master|
    master.vm.box = BOX_IMAGE
    master.vm.hostname = "master"
    master.vm.network :private_network, ip: "10.0.0.10"
    master.vm.provider :libvirt do |vb|
      vb.cpus = 2
    end
    master.vm.network "forwarded_port", guest: 80, host: 8080
    master.vm.network "forwarded_port", guest: 6443, host: 6443
    master.vm.provision "shell", inline: $init_master
  end

  %w{worker1 worker2}.each_with_index do |name, i|
    config.vm.define name do |worker|
      worker.vm.box = BOX_IMAGE
      worker.vm.hostname = name
      worker.vm.provider :libvirt do |vb|
        # vb.customize ["modifyvm", :id, "--memory", "2048"]
        vb.memory = 2048
      end
      worker.vm.network :private_network, ip: "10.0.0.#{i + 11}"
      worker.vm.provision :shell, privileged: false, inline: <<-SHELL

SHELL
    end
  end

  config.vm.provision "shell", inline: $install_multicast
end


$install_common_tools = <<-SCRIPT
# disable swap
swapoff -a
sed -i '/swap/d' /etc/fstab

# Install kubeadm, kubectl and kubelet
export DEBIAN_FRONTEND=noninteractive
apt-get -qq install ebtables ethtool
apt-get -qq update
apt-get -qq install -y docker.io apt-transport-https curl
systemctl daemon-reload
systemctl restart restart docker.service
systemctl enable docker.service
sudo usermod -aG docker vagrant
sudo sysctl net.bridge.bridge-nf-call-iptables=1
apt-key adv --fetch-keys https://packages.cloud.google.com/apt/doc/apt-key.gpg
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get -qq update
apt-get -qq install -y \
  kubelet \
  kubeadm \
  kubectl
apt-mark hold kubelet kubeadm kubectl
SCRIPT

$install_multicast = <<-SHELL
apt-get -qq install -y avahi-daemon libnss-mdns
SHELL

#kubeadm token create --print-join-command on master to get the join command for slaves
$init_master = <<-SHELL
kubeadm init --pod-network-cidr=10.244.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
SHELL


