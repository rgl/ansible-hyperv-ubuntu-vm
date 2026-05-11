# About

[![Build status](https://github.com/rgl/ansible-hyperv-ubuntu-vm/actions/workflows/build.yml/badge.svg)](https://github.com/rgl/ansible-hyperv-ubuntu-vm/actions/workflows/build.yml)

This is an example Ansible project that creates a Ubuntu Hyper-V Virtual Machine.

For a more complete Ansible Playbook see the [rgl/my-ubuntu-ansible-playbooks repository](https://github.com/rgl/my-ubuntu-ansible-playbooks).

# Usage

Install a Windows machine with Hyper-V, [`wasmtime`](https://github.com/bytecodealliance/wasmtime) and [`hadris-iso-cli-wasm`](https://github.com/rgl/hadris-iso-cli-wasm), like in [rgl/my-windows-ansible-playbooks](https://github.com/rgl/my-windows-ansible-playbooks), to serve as the  Hyper-V environment.

Configure an External Virtual Switch named `Bridge` in your Hyper-V environment.

Install the [test/templates/ubuntu-24.04-amd64-vsphere virtual machine template](https://github.com/rgl/ubuntu-vagrant) in your Hyper-V environment.

Execute the following procedure in a Ubuntu machine.

Install Docker.

Open the [inventory file](inventory.yml) and modify the virtual machines details to fit your environment.

Set your Hyper-V details:

```bash
cat >secrets.sh <<'EOF'
export VM_HYPERV_HOSTNAME='192.168.8.22'
export VM_HYPERV_USERNAME='Administrator'
export VM_HYPERV_PASSWORD='vagrant'
export VM_HYPERV_TEMPLATE='C:/Users/Administrator/.vagrant.d/boxes/ubuntu-24.04-amd64/0.0.0/hyperv/Virtual Hard Disks/packer-ubuntu-amd64.vhdx'
export VM_HYPERV_STORAGE='C:/ProgramData/Microsoft/Windows/Hyper-V/Virtual Machines'
export VM_SWITCH='Bridge'
#export VM_VLAN_MODE='Access'
#export VM_VLAN_ID='1'
export VM_GATEWAY='192.168.8.1'
export VM_FIRST_IP='192.168.8.200'
EOF
source secrets.sh
```

Lint the playbooks:

```bash
./ansible-lint.sh --offline --parseable example.yml || echo 'ERROR linting'
./ansible-lint.sh --offline --parseable example-destroy.yml || echo 'ERROR linting'
```

List the inventory:

```bash
./ansible-inventory.sh --list --yaml
```

See the facts about the `hv` machine (the Hyper-V Host):

```bash
./ansible.sh hv -m ansible.builtin.setup
```

Run an ad-hoc command in the `hv` machine (the Hyper-V Host):

```bash
./ansible.sh hv -m win_command -a 'whoami /all'
./ansible.sh hv -m win_command -a 'cmd /c winrm enumerate winrm/config/listener'
./ansible.sh hv -m win_command -a 'cmd /c winrm get winrm/config'
./ansible.sh hv -m win_shell -a 'Get-PSSessionConfiguration'
```

Create and configure the `vm1` machine (the Hyper-V Guest) using the [`example.yml` playbook](example.yml):

```bash
./ansible-playbook.sh --limit=vm1 example.yml | tee ansible-example.log
```

Access the `vm1` machine (the Hyper-V Guest):

```bash
vm1_vars="$(ANSIBLE_CALLBACK_RESULT_FORMAT=json ANSIBLE_CALLBACK_FORMAT_PRETTY=false \
    ./ansible.sh vm1 -m debug -a 'var=hostvars[inventory_hostname]' \
    | grep -oP '(?<=SUCCESS => )\{.*\}')"
vm1_user="$(jq -r '.["hostvars[inventory_hostname]"].ansible_user' <<<"$vm1_vars")"
vm1_host="$(jq -r '.["hostvars[inventory_hostname]"].ansible_host' <<<"$vm1_vars")"
ssh "$vm1_user@$vm1_host"
id
uname -a
ip addr
systemctl status
systemctl status hv-kvp-daemon.service
networkctl status
timedatectl status
ps -efww --forest
exit
```

Access the `vm1` machine (the Hyper-V Guest) from within the `hv` machine (the Hyper-V Host) using the host Hyper-V VMBus Socket (aka vsock):

```bash
hv_vars="$(ANSIBLE_CALLBACK_RESULT_FORMAT=json ANSIBLE_CALLBACK_FORMAT_PRETTY=false \
    ./ansible.sh hv -m debug -a 'var=hostvars[inventory_hostname]' \
    | grep -oP '(?<=SUCCESS => )\{.*\}')"
hv_user="$(jq -r '.["hostvars[inventory_hostname]"].ansible_user' <<<"$hv_vars")"
hv_host="$(jq -r '.["hostvars[inventory_hostname]"].ansible_host' <<<"$hv_vars")"
ssh "$hv_user@$hv_host"
pwsh
hvc list
# connect using hvc ssh.
# NB hvc ssh uses a hyper-v vmbus socket which does not use the guest network.
# NB this is equivalent of using ssh -o ProxyCommand='hvc nc %h %p' vagrant@vm1.
# NB hvc cannot use an ssh path with spaces.
$env:HV_SSH_COMMAND='C:\PROGRA~1\OpenSSH\ssh.exe'
hvc ssh vagrant@vm1
cat /etc/os-release
ps -efww --forest
sudo ss -a --vsock --processes
exit # exit hvc ssh.
exit # exit hv pwsh
exit # exit hv ssh
```

Destroy the `vm1` machine (the Hyper-V Guest) using the [`example-destroy.yml` playbook](example-destroy.yml):

```bash
./ansible-playbook.sh --limit=vm1 example-destroy.yml | tee ansible-example-destroy.log
```

# References

* [microsoft.hyperv Ansible Collection](https://galaxy.ansible.com/ui/repo/published/microsoft/hyperv)
* [microsoft.hyperv Ansible Collection repository](https://github.com/ansible-collections/microsoft.hyperv)
* [HVTools](https://github.com/michaelmsonne/HVTools)
