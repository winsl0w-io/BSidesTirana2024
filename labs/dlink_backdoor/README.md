# DLink firmware hardcoded creds

1.  Extract the firmware and navigate into it

```
> binwalk -e Dlink_firmware.bin

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
48            0x30            Unix path: /dev/mtdblock/2
96            0x60            uImage header, header size: 64 bytes, header CRC: 0x7FE9E826, created: 2010-11-23 11:58:41, image size: 878029 bytes, Data Address: 0x80000000, Entry Point: 0x802B5000, data CRC: 0x7C3CAE85, OS: Linux, CPU: MIPS, image type: OS Kernel Image, compression type: lzma, image name: "Linux Kernel Image"
160           0xA0            LZMA compressed data, properties: 0x5D, dictionary size: 33554432 bytes, uncompressed size: 2956312 bytes
917600        0xE0060         PackImg section delimiter tag, little endian size: 7348736 bytes; big endian size: 2256896 bytes
917632        0xE0080         Squashfs filesystem, little endian, non-standard signature, version 3.0, size: 2256151 bytes, 1119 inodes, blocksize: 65536 bytes, created: 2010-11-23 11:58:47


> cd _Dlink_firmware.bin.extracted/squashfs-root/

```

2. Now we will be looking into resources searching for credentials

```
> grep -iRn 'telnet' .

grep: ./usr/lib/tc/q_netem.so: binary file matches
grep: ./usr/sbin/telnetd: binary file matches
./etc/scripts/misc/telnetd.sh:3:TELNETD=`rgdb -g /sys/telnetd`
./etc/scripts/misc/telnetd.sh:4:if [ "$TELNETD" = "true" ]; then
./etc/scripts/misc/telnetd.sh:5:    echo "Start telnetd ..." > /dev/console
./etc/scripts/misc/telnetd.sh:8:        telnetd -l "/usr/sbin/login" -u Alphanetworks:$image_sign -i $lf &
./etc/scripts/misc/telnetd.sh:10:       telnetd &
./etc/scripts/system.sh:26: # start telnet daemon
./etc/scripts/system.sh:27: /etc/scripts/misc/telnetd.sh    > /dev/console
./etc/defnodes/S11setnodes.php:39:set("/sys/telnetd",           "true");
./www/__adv_port.php:22:                    <option value='Telnet'>Telnet</option>


```
3. Look into these candidates and try figuring out the creds!

```
#!/bin/sh
image_sign=`cat /etc/config/image_sign`
TELNETD=`rgdb -g /sys/telnetd`
if [ "$TELNETD" = "true" ]; then
	echo "Start telnetd ..." > /dev/console
	if [ -f "/usr/sbin/login" ]; then
		lf=`rgdb -i -g /runtime/layout/lanif`
		telnetd -l "/usr/sbin/login" -u Alphanetworks:$image_sign -i $lf &
	else
		telnetd &
	fi
fi

```

As you can see, one of the lines starts telnet with the `-u` argument and providing the username `Alphanetworks` along with the `$image_sign` variable as a password.

The variable definition can be traced back to the beginning of the file, which indicates that the value is taken from the content of the file located at `/etc/config/image_sign`

So we have telnet running using hardcoded credentials.
