# How to deploy a NixOS VM via virt-installer

## Launching the VM instance

1. Launch a new machine via virt-install, using the Ignition file with your customizations.

    ```bash
    source .env
    virt-install --connect="qemu:///session" --name="${VM_NAME}" --vcpus="${VCPUS}" --memory="${RAM_MB}" \
        --os-variant="nixos-21.05" --import --graphics=none \
        --disk="size=${DISK_GB},path=$HOME/.local/share/libvirt/images/${VM_NAME}-disk1.qcow2" \
        --network="model=virtio,bridge=virbr0,mac=${MAC_ADDR}" \
        --cdrom=${ISO} --livecd
    ```

2. Select boot option for serial console

    You may not see the Boot menu. If this is the case press ESC ENTER or ESC and enter 'boot-serial' to directly boot.

    ```text
    ┌──────────────────────────────────────────────────────────────────────────────┐
    │                                    NixOS                                     │
    ├──────────────────────────────────────────────────────────────────────────────┤
    │ NixOS 21.11.336824.ccb90fb9e11 Installer                                     │
    │ NixOS 21.11.336824.ccb90fb9e11 Installer (nomodeset)                         │
    │ NixOS 21.11.336824.ccb90fb9e11 Installer (copytoram)                         │
    │ NixOS 21.11.336824.ccb90fb9e11 Installer (debug)                             │
    │*NixOS 21.11.336824.ccb90fb9e11 Installer (serial console=ttyS0,115200n8)*    │
    │ Memtest86+                                                                   │
    └──────────────────────────────────────────────────────────────────────────────┘
    ```

3. enable ssh access

    ```bash
    # generate ss keys
    ssh-keygen -t ed25519 -a 100 -f /home/nixos/.ssh/id_ed25519 -P ''

    # install some tools
    nix-env -f '<nixpkgs>' -iA vim

    # print ip address
    ip addr | grep 'state UP' --after-context=2 | sed -r -n -e 's/.*inet ([^/]*).*/\1/p'

    # set nixos password
    passwd
    ```

4. connect via ssh to VM

    replace <IP-ADDR> with the actual ip address.

    ```bash
    ssh nixos@<IP-ADDR>
    ```

## Installing NixOS from liveCD

1. Partitioning and formatting (MBR)

    ```bash
    # create root partition and 4 GiB swap space
    sudo parted /dev/vda -- mklabel msdos
    sudo parted /dev/vda -- mkpart primary 1MiB -4GiB
    sudo parted /dev/vda -- mkpart primary linux-swap -4GiB 100%

    # format partitions
    sudo mkfs.ext4 -L nixos /dev/vda1
    sudo mkswap -L swap /dev/vda2
    ```

2. Installing

    ```bash
    # activate swap device
    sudo swapon /dev/sda2

    # mount rootfs
    sudo mount /dev/disk/by-label/nixos /mnt

    # generate nixos config
    sudo nixos-generate-config --root /mnt

    # edit config file
    sudo vim /mnt/etc/nixos/configuration.nix

    # do the installation
    sudo nixos-install --no-root-password

    # reboot
    sudo reboot
    ```

## Further readings

- [NixOS Manual](https://nixos.org/manual/nixos/stable)
- [Nix Expression Language](https://nixos.wiki/wiki/Nix%20Expression%20Language)
- [Creating guests with virt-install
](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-guest_virtual_machine_installation_overview-creating_guests_with_virt_install)
