BOX='generic/ubuntu2004'
EGWINT = ENV.fetch('EGWINT', 'enp113s0f0')
VAULT_PASSWORD_FILE = '.ansible-vault-password'
SHELL_PROVISION_SCRIPT = <<-SHELL
  # remove the vagrant default route so ansible figures out the correct default interface
  ip route | grep --quiet "^default via 192.168.121.1" && ip route delete default via 192.168.121.1 || true
SHELL

Vagrant.configure('2') do |config|
  config.vm.box = BOX

  config.vm.provider :libvirt do |lv|
    lv.cpus = 2
    lv.memory = 4096
  end

  config.vm.synced_folder './', '/vagrant', type: 'nfs'

  config.vm.define 'egw' do |egw|
    egw.vm.hostname = 'egw'
    egw.vm.network :public_network,
                   dev: EGWINT,
                   mode: 'passthrough',
                   mac: '20:ca:b0:70:00:02',
                   trust_guest_rx_filters: true
    egw.vm.provision :shell, inline: SHELL_PROVISION_SCRIPT
    egw.vm.provision 'ansible' do |ansible|
      ansible.playbook = 'master.yml'
      ansible.compatibility_mode = '2.0'
      ansible.host_vars = {
        egw: {
          bridge_name: 'multus0',
          isGateway: true,
          subnet: '192.168.128.0/24',
          rangeStart: '192.168.128.200',
          rangeEnd: '192.168.128.216',
          gateway: '192.168.128.1',
          pod_cidr: '10.128.0.0/16',
        }
      }
      ansible.verbose = true
      ansible.vault_password_file = VAULT_PASSWORD_FILE
    end
    egw.trigger.after :destroy do |trigger|
      trigger.run = {inline: 'rm -f egw-admin.conf master.retry'}
    end

  end


  config.trigger.before [:up, :resume, :reload] do |trigger|
    trigger.info = "Setting EGWINT #{EGWINT} down"
    trigger.run = { inline: "sudo ip link set dev #{EGWINT} down" }
  end


  config.trigger.after [:destroy, :halt, :suspend] do |trigger|
    trigger.info = "setting EGWINT #{EGWINT} down"
    trigger.run = { inline: "sudo ip link set dev #{EGWINT} down" }
  end

end