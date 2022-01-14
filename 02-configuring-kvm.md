# Part2 - Configuring KVM

To continue using KVM with your standard Linux account, you can do so by modifying the libvirtd.conf file. Access the file by entering the following:
- `sudo vim /etc/libvirt/libvirtd.conf`

Between line 80-90, there should be the term “unix_sock_group.” You will change this to libvirt.
- `unix_sock_group = "libvirt"`

Jump to the lines between 100-110 and change the unix_sock_rw_perms to = 0770
- `unix_sock_rw_perms = "0770"`

Then, issue the following code to include your standard Linux account with libvirt.
- `sudo usermod -a -G libvirt $(whoami)`
- `sudo usermod -a -G kvm $(whoami)`
- `newgrp libvirt`


Reboot the libvirt service to apply changes. To do so, issue the following code:
- `sudo systemctl restart libvirtd.service`

You can now use your Linux account to use KVM.
If you use a Window Manager like dwm, spectrwm, awesome, etc, check also polkit on Arch. You will need one of them with WM (lxsession). With Desktop environment like Gnome/KDE this is ok.


