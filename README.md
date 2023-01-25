# AGENT SMITH

## SET UP VM ON VBOX
OVA_NAME=flatcar.ova   #fedora-coreos-37.20221225.3.0-virtualbox.x86_64.ova
VM_NAME=my-instance
IGN_PATH="./config_online.ign"
VBoxManage import --vsys 0 --vmname "$VM_NAME" "$OVA_NAME" && \
VBoxManage guestproperty set "$VM_NAME" /Ignition/Config "$(cat $IGN_PATH)" && \
VBoxManage modifyvm "$VM_NAME" --natpf1 "guestssh,tcp,,2222,,22" && \
VBoxManage startvm "$VM_NAME"

ssh -o "StrictHostKeyChecking=no" core@localhost -p 2222 -i ~/.ssh/id_ecdsa_fedora

# SETUP SERVICES ON DOCKER CONTAINERS
docker run \
-v /home/core/sftp/id_ecdsa:/etc/ssh/ssh_host_ed25519_key \
-v /home/core/sftp/input:/home/foo/share \
-p 2222:22 -d atmoz/sftp \
foo::1001

## SOURCES

https://coreos.github.io/ignition/configuration-v3_4_experimental/

https://coreos.github.io/ignition/examples/

https://docs.fedoraproject.org/en-US/fedora-coreos/provisioning-virtualbox/

## SELINUX NOT NICE
with SE_linux set as enforcing the volumes are mounted but not accessible and there is a permission error which is very misleading.

```bash
docker run \
-v /root/sshkey/ssh_host_ed25519_key:/etc/ssh/ssh_host_ed25519_key \
-v /root/sshkey/ssh_host_rsa_key:/etc/ssh/ssh_host_rsa_key \
-v /root/sshkey/ssh_host_ed25519_key.pub:/home/foo/.ssh/keys/ssh_host_ed25519_key.pub:ro \
-v /root/sshkey/ssh_host_rsa_key.pub:/home/foo/.ssh/keys/ssh_host_rsa_key.pub:ro \
-v /root/sshkey/share:/home/foo/share \
-p 2222:22 -d atmoz/sftp \
foo::1001
```

## Docker from systemd

```
[Unit]
Description=YouTrack Service
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker exec %n stop
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull jetbrains/youtrack:<version>
ExecStart=/usr/bin/docker run --rm --name %n \
    -v <path to data directory>:/opt/youtrack/data \
    -v <path to conf directory>:/opt/youtrack/conf \
    -v <path to logs directory>:/opt/youtrack/logs \
    -v <path to backups directory>:/opt/youtrack/backups \
    -p <port on host>:8080 \
    jetbrains/youtrack:<version>

[Install]
WantedBy=default.target
```

## Connect SFTP
```bash
sftp -o "StrictHostKeyChecking no" -i /root/sshkey/ssh_host_ed25519_key -P 2222 foo@localhost
```