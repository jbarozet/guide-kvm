# Part1 - Installing KVM

<br>

## Step 1: Check for Virtualization Support

To check whether virtualization is enabled on your PC, issue the following command:

- `LC_ALL=C lscpu | grep Virtualization`

The hardware specs to run KVM is VT-x for Intel processors and AMD-V for AMD processors. As such, if your system has the hardware to create virtual machines, you will see the following within the text you are displayed:

**Virtualization: VT-x** or **Virtualization: AMD-V**

Alternatively:

- `egrep -c '(vmx|svm)' --color=always /proc/cpuinfo`

If you get back results with `vmx`, then you have an Intel processor. If you get back results with `svm`, then you have an AMD processor. If you receive a `null` return, then your processor is not built for hardware supported full virtualization. The xen approach, used in the CentOS 5 series, supports para virtualization.

<br>

## Step 2: Search for Kernel Module

To see whether your system has the kernel module to run KVM, enter the following command:

`zgrep CONFIG_KVM /proc/config.gz`

Example:

```bash
zgrep CONFIG_KVM /proc/config.gz
CONFIG_KVM_GUEST=y
CONFIG_KVM_MMIO=y
CONFIG_KVM_ASYNC_PF=y
CONFIG_KVM_VFIO=y
CONFIG_KVM_GENERIC_DIRTYLOG_READ_PROTECT=y
CONFIG_KVM_COMPAT=y
CONFIG_KVM_XFER_TO_GUEST_WORK=y
CONFIG_KVM=m
CONFIG_KVM_INTEL=m
CONFIG_KVM_AMD=m
CONFIG_KVM_AMD_SEV=y
CONFIG_KVM_XEN=y
CONFIG_KVM_MMU_AUDIT=y
```

<br>

## Step 3: Install KVM on Arch

Install packages for KVM
- `sudo pacman -S virt-manager qemu qemu-arch-extra ovmf vde2 dnsmasq bridge-utils openbsd-netcat`

<br>

## Step 4: Activate and Launch KVM

Issue the following command to activate KVM:
- `sudo systemctl enable libvirtd.service`

Next, enter the following:
- `sudo systemctl start libvirtd.service`

<br>


## References

[Install KVM on Arch](https://www.youtube.com/watch?v=itZf5FpDcV0)

