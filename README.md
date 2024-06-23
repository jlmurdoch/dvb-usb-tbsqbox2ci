# Kernel module for TurboSight (TBS) QBox2 CI DVB-S2 

Just add the TBS QBox2 CI USB to kernel source.

Taken from the github.com/tbsdtv sources in June 2024 after failing to get the full package to work.

The QBox2 CI uses code pre-existing in the kernel source:
* STMicroelectronics STB6100 DVB-S2 tuner [CONFIG_DVB_STB6100]
* STMicroelectronics STV0903 DVB-S2 demodulator [CONFIG_DVB_STV090X]

LED functionality has been commented out to avoid changes to other source code.

This means that inside the drivers/media/usb/dvb-usb directory, the following code is created/modified:
* Created:
  * tbs-qbox2ci.c
  * tbs-qbox2ci.h
* Modified:
  * Kconfig
  * Makefile

Install guide for Debian 12:
```
# Install linux kernel source on Debian 12
apt update
apt upgrade
apt install linux-source
cd /usr/src
tar xf linux-source-*.tar.xz

# Patch TBS5980 into dvb-usb
cd /usr/src/linux-source-*
patch -p1 << /path/to/dvb-usb-tbsqbox2ci.patch

# Configure the kernel and enable the module
cp /boot/config-$(uname -r) .config
echo CONFIG_DVB_USB_TBSQBOX2CI=m >> .config

# Compile in-tree dvb-usb modules
make prepare
make modules_prepare
make M=drivers/media/usb/dvb-usb modules

# Install
cp drivers/media/usb/dvb-usb/dvb-usb-tbsqbox2ci.ko /lib/modules/$(uname -r)/kernel/drivers/media/usb/dvb-usb/
depmod -a
cp /path/to/dvb-usb-tbsqbox-id5980.fw /lib/firmware/
```

Quickstart testing / investigation / tuning:
```
# Use w_scan for scanning, dvblast for tuning / CI testing
apt install w-scan dvblast

# Scan the current LNB in its existing position
w_scan -f s -s S19E2

# Tune in on QVC Germany on 19.2E (vertical on 13V, default)
dvblast -f 12551500 -s 22000000 

# Tune in on QVC HD Germany on 19.2E (horizontal on 18V)
dvblast -f 10788000 -s 22000000 -v 18
```
