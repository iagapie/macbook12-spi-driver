Very simple, work in progress input driver for the SPI keyboard / trackpad found on 12" MacBooks (2015 and later) and MacBook Pros 13,* (late 2016) and 14,* (mid 2018).

As of kernel 5.3 this driver is part of the standard kernel; a couple additional fixes have been included in 5.12.

Using it:
---------
If you're on any MacBook or MacBook Pro other than MacBook8,1 (2015), and you're running a kernel before 4.11, then you'll need to boot the kernel with `intremap=nosid`. In all cases make sure you don't have `noapic` in your kernel options.

On the 2015 MacBook you need to (re)compile your kernel with `CONFIG_X86_INTEL_LPSS=n` if running a kernel before 4.14. And on all kernels you need ensure the `spi_pxa2xx_platform` and `spi_pxa2xx_pci` modules are loaded too (if you don't have those module, rebuild your kernel with `CONFIG_SPI_PXA2XX=m` and `CONFIG_SPI_PXA2XX_PCI=m`).

On all other MacBook's and MacBook Pros you need to instead make sure both the `spi_pxa2xx_platform` and `intel_lpss_pci` modules are loaded (if these don't exist, you need to (re)compile your kernel with `CONFIG_SPI_PXA2XX=m` and `CONFIG_MFD_INTEL_LPSS_PCI=m`).

For best results everywhere, make sure all three modules (this `applespi` driver plus the two core ones mentioned above) are present in your initramfs/initrd so that the keyboard is functional by the time the prompt for the disk password appears. Also, having them loaded early also appears to remove the need for the `irqpoll` kernel parameter on MacBook8,1's.


DKMS module (Debian & co):
--------------------------
As root, do the following (all MacBook's and MacBook Pro's except MacBook8,1 (2015)):
```
echo -e "\n# applespi\napplespi\nspi_pxa2xx_platform\nintel_lpss_pci" >> /etc/initramfs-tools/modules

apt install dkms
git clone https://github.com/cb22/macbook12-spi-driver.git /usr/src/applespi-0.1
dkms install -m applespi -v 0.1
```

If you're on a MacBook8,1 (2015):
```
echo -e "\n# applespi\napplespi\nspi_pxa2xx_platform\nspi_pxa2xx_pci" >> /etc/initramfs-tools/modules

apt install dkms
git clone https://github.com/roadrunner2/macbook12-spi-driver.git /usr/src/applespi-0.1
dkms install -m applespi -v 0.1
```

Akmods module (RPM Fusion / Red Hat & co):
------------------------------------------
You can build the akmod package from this repository:

https://pagure.io/fedora-macbook12-spi-driver-kmod

Or use this [copr repository](https://copr.fedorainfracloud.org/coprs/meeuw/macbook12-spi-driver-kmod/):
```
$ dnf copr enable meeuw/macbook12-spi-driver-kmod

$ dnf install macbook12-spi-driver-kmod
```

What doesn't work:
------------------
* Autodetection of ISO layout
* Resume on MacBook8,1
 
Debugging:
----------
Packet tracing is exposed via the kernel tracepoints framework. Tracing of individual packet types can be enabled with something like the following:
```
echo 1 | sudo tee /sys/kernel/debug/tracing/events/applespi/applespi_keyboard_data/enable
```
The packets are then visible in `/sys/kernel/debug/tracing/trace`

Trackpad dimensions logging can be enabled with
```
echo 1 | sudo tee /sys/kernel/debug/applespi/enable_tp_dim
```
and then viewed with something like
```
sudo watch /sys/kernel/debug/applespi/tp_dim
```

Some useful threads:
--------------------
* https://bugzilla.kernel.org/show_bug.cgi?id=108331
* https://bugzilla.kernel.org/show_bug.cgi?id=99891
