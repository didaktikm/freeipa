# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']

MACHINES = {
  :ipaclient => {
        :box_name => "centos/7",
        :ip_addr => '172.20.10.51',
        :memory => "256"
  },       
  :ipaserver => {
        :box_name => "centos/7",
        :ip_addr => '172.20.10.50',
        :memory => "2048"
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          # box.vm.provider :virtualbox do |vb|
          # vb.customize ["modifyvm", :id, "--memory", "2048"]
        #   vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
          # vb.name = boxname.to_s

        #   boxconfig[:disks].each do |dname, dconf|
        #       unless File.exist?(dconf[:dfile])
        #         vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
        #       end
        #       vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
        # end
        # end
      box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
      SHELL

      case boxname.to_s
      when "ipaserver"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          cp /vagrant/id_rsa /home/vagrant/.ssh/
          cp /vagrant/id_rsa /root/.ssh/
          chown vagrant:vagrant /home/vagrant/.ssh/id_rsa 
          chown root:root /root/.ssh/id_rsa
          chmod 0600 /home/vagrant/.ssh/id_rsa
          chmod 0600 /root/.ssh/id_rsa
          yum install epel-release ansible vim -y
          yum install ipa-server ipa-server-dns -y
          echo "172.20.10.50  pol.otus.local pol" > /etc/hosts
          echo "172.20.10.51  client" >> /etc/hosts
          echo "[ipaclients]\nclient" >> /etc/ansible/hosts
          sed -i 's/#host_key_checking = False/host_key_checking = False/g' /etc/ansible/ansible.cfg
          ipa-server-install --hostname=pol.otus.local --domain=otus.local --realm=OTUS.LOCAL --ds-password=qwerty19 --admin-password=qwerty19 --mkhomedir --setup-dns --forwarder=8.8.8.8 --auto-reverse --unattended
          ansible-playbook /vagrant/playbook-client.yml
          SHELL
      when "ipaclient"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
        echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCW+VHI6di+7jZZhnYiCUciVO3oCSJ1xkV+8TINsNy1Itek0BUnorH+Mh6wC5eHoFVsid39v5A5ypzYZvJWhjwu4LNBJFroNhPnpmSBoA7Xk9U+slDI1A6pImop3qQbncMbYMdeyK5yoQO9bgJKDoQG7ak99qp24C4koFHGXO9Bejhenkkct2j0iTQreRyv2y3oSeOvsvQcBFuYS3H0FPhTUII8dx+/tjOTYFaxiA+EkWhuyXfhnrUd60BN5+ajqEgtv4CYZm2MBzDWu3Sor142Ms3R/FbwF1MJKd7JHOzJcTARfnpBqBZi+Or+l9+Pdl8yzxbxO0+9yaj7MGP9eyVT" >> /home/vagrant/.ssh/authorized_keys
        SHELL
      end
      end
   end
end
