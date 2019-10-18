# wlan-extcap
Wireshark extcap interface for remote wireless captures using a Linux device.

This extcap interface is basically a wrapper for the `sshdump` extcap interface that includes additional options to customize the capture. For example, if capturing Wi-Fi traffic, you can choose the Wi-Fi channel to capture on. It also simplifies the configuration of the extcap interface so that the user doesn't have to deal with complex remote capture commands, etc.

The `wlandump` extcap interface currently provides two capture interfaces: __Wi-Fi__ and __Zigbee__, each with its own set of options. The `wifidump` interface allows you to perform remote Wi-Fi captures on a specific channel and channel width using a  Linux device with a Wi-Fi adapter that can be put into monitor mode. The `zbdump` interface allows you to perform remote Zigbee captures using Linux device with a [TI CC2531 USB](http://www.ti.com/tool/CC2531EMK) dongle.

## Installation

### If you're running Wireshark on Windows:

1. Install [Python](https://www.python.org/downloads/).
2. The `wlandump` extcap interface requires the `sshdump` extcap interface, which is not installed by default on Windows. When installing Wireshark on Windows, select __SSHdump__ as one of the components to install:

<p align="center">
<img src="/images/wireshark-installer-sshdump.png" alt="Wireshark Installer SSHdumpr" height="400px">
</p>

2. Copy `wlandump` to `C:\Program Files\Wireshark\extcap\`
3. Create a file called `wlandump.bat` in the same `C:\Program Files\Wireshark\extcap\` directory with the following content: 

```bat
@echo off
<PATH_TO_PYTHON_INTERPRETER> <PATH_TO_WLANDUMP> %*
```

Where `<PATH_TO_PYTHON_INTERPRETER>` is the path to the Python executable and `<PATH_TO_WLANPIDUMP>` is the path to the `wlandump` extcap interface script. For example:

```bat
@echo off
"C:\Program Files (x86)\Python37-32\python.exe" "C:\Program Files\Wireshark\extcap\wlandump" %*
```

### If you're running Wireshark on macOS:
1. Copy `wlandump` to `/Applications/Wireshark.app/Contents/MacOS/extcap/`
2. Make sure it has execution permissions:

```sh
$ chmod +x /Applications/Wireshark.app/Contents/MacOS/extcap/wlandump
```

### If you're running Wireshark on Linux
The steps are the same as the ones above for macOS, the only difference is the path to copy `wlandump` to. To find the correct path:
1. On Wireshark, go to _`Help -> About Wireshark`_;
2. Change to tab __Folders__;
3. Use the path indicated by __Extcap path__.

<p align="center">
<img src="/images/wireshark-extcap-path.png" alt="Extcap path" height="300px">
</p>

Launch Wireshark and verify that the capture interfaces provided by the `wlandump` extcap interface are listed:

![WLAN Extcap Interface](/images/wlandump-interfaces.png "WLAN Extcap Interface")

> __Note__: You will have to reinstall the `wlandump` extcap interface on your computer each time you update Wireshark. The Wireshark installer doesn't preserve 3rd-party extcap interfaces added to the _extcap_ folder.

## Remote Wi-Fi Captures

The `wifidump` capture interface allows you to perform remote Wi-Fi captures on a specific channel and channel width using a Linux device with a Wi-Fi adapter that can be put into monitor mode.

### Setup

The `wifidump` capture interface uses `tcpdump` as the remote tool for Wi-Fi captures. Make sure `tcpdump` can be run remotely by the SSH user and without the need of root privileges. For example:
```sh
$ sudo groupadd pcap
$ sudo usermod -a -G pcap USERNAME
$ sudo chgrp pcap /usr/sbin/tcpdump
$ sudo chmod 750 /usr/sbin/tcpdump
$ sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
```

where `USERNAME` is the SSH user for connecting remotely.

The interface also requires of the `ip` and `iw` command line utilities to put the Wi-Fi adapter in monitor mode and set the desired channel and channel width. Make sure these two utilities are installed and then create the file `/etc/sudoers.d/wifidump` with the following content:

```sh
USERNAME ALL = (root) NOPASSWD: /sbin/ip, /usr/sbin/iw
```

where `USERNAME` is, again, the SSH user for connecting remotely.

### Usage

1. Click the _gear_ icon next to "Wi-Fi remote capture" to display the interface options, then choose the interface name, channel, and channel width you want to capture on:

<p align="center">
<img src="/images/wifidiump-interface-options.png" alt="Wi-Fi Interface Options" height="200px">
</p>

> __Note:__ All 802.11 channels are listed, however, the Wi-Fi adapter on the remote device may support only a subset of them. If you choose a channel that is not supported by the Wi-Fi adapter or a channel width that doesn't apply to the selected channel, the capture will fail.
2. Go to the _Server_ tab and enter the remote SSH server address, e.g. 192.168.42.1.

<p align="center">
<img src="/images/wifidump-interface-options-server.png" alt="Wi-Fi Extcap Interface Options - Server" height="200px">
</p>

3. Go to the _Authentication_ tab and enter the username and password.

<p align="center">
<img src="/images/wifidump-interface-options-auth.png" alt="Wi-Fi Extcap Interface Options - Auth" height="200px">
</p>

> __Note:__ The password is not saved, so to avoid having to enter the password each time you start a capture, I would recommend you setup passwordless SSH authentication.

4. Click the __Start__ button to start the capture.

![Wi-Fi Capture](/images/wifi-capture.png "Wi-Fi Capture")

## Remote Zigbee Captures

The `zbdump` capture interface uses `whsniff` as the remote tool for Zigbee captures using the TI CC2531 USB dongle. To install `whsniff` in the remote Linux device:

1. Install `libusb-1.0-0-dev`:
```sh
$ sudo apt-get install libusb-1.0-0-dev
```
2. Download [the latest release](https://github.com/homewsn/whsniff/releases) in tarball from github and untar it. Then build and install whsniff.
```sh
$ curl -L https://github.com/homewsn/whsniff/archive/v1.1.tar.gz | tar zx
$ cd whsniff-1.1
$ make
$ sudo make install
```

Then create the file `/etc/sudoers.d/zbdump` with the following content:

```sh
USERNAME ALL = (root) NOPASSWD: /usr/local/bin/whsniff, /usr/bin/killall /usr/local/bin/whsniff
```

where `USERNAME` is the SSH user for connecting remotely.

### Usage

1. Click the _gear_ icon next to "Zigbee remote capture" to display the interface options, then choose the Zigbee channel you want to capture on:

<p align="center">
<img src="/images/zbdump-interface-options.png" alt="Zigbee Interface Options" height="200px">
</p>

2. Go to the _Server_ tab and enter the remote SSH server address, e.g. 192.168.42.1.

<p align="center">
<img src="/images/zbdump-interface-options-server.png" alt="Zigbee Interface Options - Server" height="200px">
</p>

3. Go to the _Authentication_ tab and enter the username and password.

<p align="center">
<img src="/images/zbdump-interface-options-auth.png" alt="Zigbee Interface Options - Auth" height="200px">
</p>

> __Note:__ The password is not saved, so to avoid having to enter the password each time you start a capture, I would recommend you setup passwordless SSH authentication.

4. Click the __Start__ button to start the capture.

![Zigbee Capture](/images/zigbee-capture.png "Zigbee Capture")
