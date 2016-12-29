# -*- mode: ruby -*-
# vi: set ft=ruby :


$num_instances = 3

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  
  # Create k8s-master
  config.vm.define "master"  do |config|
      config.vm.hostname = "master"
      config.vm.network "private_network", ip: "192.168.50.130"

      config.vm.provision "file", source: "master-etcd.conf", destination: "etcd.conf"
      config.vm.provision "file", source: "master-apiserver", destination: "apiserver"

      config.vm.provision "shell", privileged: true, env: {"HOSTNAME" => config.vm.hostname}, inline: <<-SHELL
        systemctl stop firewalld
        systemctl disable firewalld
        yum -y install ntp
        systemctl start ntpd
        systemctl enable ntpd
        yum -y install etcd kubernetes;
        mv etcd.conf /etc/etcd/etcd.conf
        mv apiserver /etc/kubernetes/apiserver
        for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
          systemctl restart $SERVICES
          systemctl enable $SERVICES
          systemctl status $SERVICES 
        done
        etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}'
      SHELL
     end
     

  # Create k8s nodes 
  (1..$num_instances).each do |i|
    config.vm.define "minion#{i}"  do |config|
        config.vm.hostname = "minion#{i}"
        
        config.vm.network "private_network", ip: "192.168.50.#{130 + i}"

        config.vm.provision "file", source: "minion-kubelet", destination: "minion-kubelet"
        config.vm.provision "file", source: "minion-flanneld", destination: "minion-flanneld"
        
        kubelet_config_line = "KUBELET_HOSTNAME=\"--hostname_override=192.168.50.#{130 + i}\""

        config.vm.provision "shell", privileged: true, env: {"KUBELET_CONFIG_LINE" => kubelet_config_line}, inline: <<-SHELL
          systemctl stop firewalld
          systemctl disable firewalld
          yum -y install ntp
          systemctl start ntpd
          systemctl enable ntpd
          
          yum -y install flannel kubernetes
          
          head -n -1 /etc/kubernetes/config > temp.txt ; mv -f temp.txt /etc/kubernetes/config
          echo 'KUBE_MASTER="--master=http://192.168.50.130:8080"' >> /etc/kubernetes/config
          
          echo $KUBELET_CONFIG_LINE
          echo $KUBELET_CONFIG_LINE >> minion-kubelet
          mv minion-kubelet  /etc/kubernetes/kubelet
          mv minion-flanneld  /etc/sysconfig/flanneld
          
          for SERVICES in kube-proxy kubelet docker flanneld; do
              systemctl restart $SERVICES
              systemctl enable $SERVICES
              systemctl status $SERVICES 
          done

        SHELL
       end


  end
end
