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

Create and configure the `vm1` machine using the [`example.yml` playbook](example.yml): 

```bash
./ansible-playbook.sh --limit=vm1 example.yml | tee ansible-example.log
```

Destroy the `example1` machine using the [`example-destroy.yml` playbook](example-destroy.yml): 

```bash
./ansible-playbook.sh --limit=vm1 example-destroy.yml | tee ansible-example-destroy.log
```

# References

* [microsoft.hyperv Ansible Collection](https://galaxy.ansible.com/ui/repo/published/microsoft/hyperv)
* [microsoft.hyperv Ansible Collection repository](https://github.com/ansible-collections/microsoft.hyperv)
* [HVTools](https://github.com/michaelmsonne/HVTools)
