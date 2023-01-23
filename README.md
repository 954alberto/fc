# AGENT SMITH

## SET UP VM ON VBOX
VM_NAME=my-instance
IGN_PATH="./config_online.ign"
VBoxManage import --vsys 0 --vmname "$VM_NAME" fedora-coreos-37.20221225.3.0-virtualbox.x86_64.ova
VBoxManage guestproperty set "$VM_NAME" /Ignition/Config "$(cat $IGN_PATH)"
VBoxManage modifyvm "$VM_NAME" --natpf1 "guestssh,tcp,,2222,,22"
VBoxManage startvm "$VM_NAME"

ssh -o "StrictHostKeyChecking=no" core@localhost -p 2222 -i ~/.ssh/id_ecdsa_fedora

# SETUP SERVICES ON DOCKER CONTAINERS
docker run \
-v /home/core/sftp/id_ecdsa:/etc/ssh/ssh_host_ed25519_key \
-v /home/core/sftp/input:/home/foo/share \
-p 2222:22 -d atmoz/sftp \
foo::1001