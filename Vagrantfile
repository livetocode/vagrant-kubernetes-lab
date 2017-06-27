# -*- mode: ruby -*-
# vi: set ft=ruby :



Vagrant.require_version ">= 1.6.0"

boxes = [
    {
        :name => "k8smaster",
        :eth1 => "192.168.8.10",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "k8sworker",
       :eth1 => "192.168.8.11",
        :mem => "2048",
        :cpu => "2"
    },
]

Vagrant.configure(2) do |config|

  config.vm.box = "ubuntu/xenial64"

  boxes.each do |opts|
      config.vm.define opts[:name] do |config|
        config.vm.hostname = opts[:name]

        config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", opts[:mem]]
          v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
        end

        config.vm.network :private_network, ip: opts[:eth1]
      end
  end

  config.vm.define "k8smaster" do |k8smaster|
    k8smaster.ssh.forward_agent = true
    k8smaster.vm.provision "shell", inline: <<-SHELL
      set -e
      set -x
      sed 's/127\.0\.0\.1.*k8s.*/192\.168\.8\.10 k8smaster/' -i /etc/hosts
      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
      echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list
      mv ~/kubernetes.list /etc/apt/sources.list.d
      apt-get update
      apt-get upgrade -y
      apt-get install -y docker.io
      apt-get install -y kubelet kubeadm kubectl kubernetes-cni
      apt-get install -y nfs-common
      wget --no-verbose -O /usr/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.7.0-beta.2/bin/linux/amd64/kubectl
      chmod a+x /usr/bin/kubectl
      wget --no-verbose -O /usr/bin/kubeadm https://storage.googleapis.com/kubernetes-release/release/v1.7.0-beta.2/bin/linux/amd64/kubeadm
      chmod a+x /usr/bin/kubeadm
      systemctl stop kubelet.service
      wget --no-verbose -O /usr/bin/kubelet https://storage.googleapis.com/kubernetes-release/release/v1.7.0-beta.2/bin/linux/amd64/kubelet
      chmod a+x /usr/bin/kubelet
      systemctl start kubelet.service
      sleep 5
      kubeadm init --apiserver-advertise-address 192.168.8.10 --pod-network-cidr 10.244.0.0/16 --kubernetes-version v1.6.6 --token 54c315.78a320e33baaf27d
      echo "export KUBERNETES_SERVICE_HOST=192.168.8.10" > /etc/profile.d/kubernetes.sh
      echo "export KUBERNETES_SERVICE_PORT=6443" >> /etc/profile.d/kubernetes.sh
      echo "export KUBECONFIG=/vagrant/kubeconfig/admin.conf" >> /etc/profile.d/kubernetes.sh
      cp -rf  /etc/kubernetes/admin.conf /vagrant/kubeconfig/      
      export KUBECONFIG=/etc/kubernetes/admin.conf
      kubectl patch daemonset kube-proxy -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/command/2", "value":"--proxy-mode=userspace"}]'
      kubectl patch daemonset kube-proxy -n kube-system --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/securityContext", "value": {"privileged": true}}]'
      sleep 60
      kubectl create -f /vagrant/networking/kube-weave.yml
      sleep 15
    SHELL
  end

  config.vm.define "k8sworker" do |k8sworker|
    k8sworker.ssh.forward_agent = true
    k8sworker.vm.provision "shell", inline: <<-SHELL
      set -e
      set -x
      sed 's/127\.0\.0\.1.*k8s.*/192\.168\.8\.11 k8sworker/' -i /etc/hosts
      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
      echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list
      mv ~/kubernetes.list /etc/apt/sources.list.d
      apt-get update
      apt-get upgrade -y
      apt-get install -y docker.io
      apt-get install -y kubelet kubeadm kubectl kubernetes-cni
      apt-get install -y nfs-common
      wget --no-verbose -O /usr/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.7.0-beta.2/bin/linux/amd64/kubectl
      chmod a+x /usr/bin/kubectl
      wget --no-verbose -O /usr/bin/kubeadm https://storage.googleapis.com/kubernetes-release/release/v1.7.0-beta.2/bin/linux/amd64/kubeadm
      chmod a+x /usr/bin/kubeadm
      systemctl stop kubelet.service
      wget --no-verbose -O /usr/bin/kubelet https://storage.googleapis.com/kubernetes-release/release/v1.7.0-beta.2/bin/linux/amd64/kubelet
      chmod a+x /usr/bin/kubelet
      systemctl start kubelet.service
      sleep 5
      kubeadm join --token=54c315.78a320e33baaf27d 192.168.8.10:6443
      sleep 120
      echo "export KUBERNETES_SERVICE_HOST=192.168.8.10" > /etc/profile.d/kubernetes.sh
      echo "export KUBERNETES_SERVICE_PORT=6443" >> /etc/profile.d/kubernetes.sh
      echo "export KUBECONFIG=/vagrant/kubeconfig/admin.conf" >> /etc/profile.d/kubernetes.sh
      export KUBECONFIG=/vagrant/kubeconfig/admin.conf
      kubectl create -f /vagrant/monitoring/kube-heapster.yml
      kubectl create -f /vagrant/dashboard/kube-dashboard.yml
    SHELL
  end
end
