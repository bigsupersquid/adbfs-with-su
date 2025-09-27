This variant of adbfs is a workaround for root access to the device filesystem using su, for devices with no "adb root" due to a user build of ROM.

WARNING ACHTUNG ETC

How to Not Nuke Your Device With This (Mostly)

#### 1. **Mount Read-Only by Default**
When testing, mount your `adbfs-with-su` variant with `-o ro`:
```bash
./adbfs -o ro /mnt/phone
```
Only switch to `-o rw` when you *know* what you’re doing.

#### 2. **Use a Wrapper Script**
```bash
#!/bin/bash
echo "WARNING: This is RW. Ctrl+C in 3 seconds..."
sleep 3
./adbfs -o rw,allow_other /mnt/phone
```

#### 3. **Backup Key Dirs First**
Before poking around:
```bash
adb pull /data/system ~/_backup-data-system
adb pull /data/misc/wifi ~/_backup-wifi
adb pull /data/adb/magisk.db ~/_backup-magisk
```

#### 4. **Test in `/data/local/tmp` First**
It’s your **sandbox**. Write, break, delete — no harm done.

#### 5. **Don’t `chown -R 0:0 /data`**
I don’t know why you’d do it.
But someone, somewhere, will.
And their phone will never boot again.



Actual Instructions:
=============

## Ubuntu

You will need `libfuse-dev` and `adb`. You will also need `build-essential`, `git`, and `pkg-config`.

    sudo apt-get install libfuse-dev android-tools-adb
    sudo apt-get install build-essential git pkg-config

Clone the repository:

    git clone git@github.com:spion/adbfs-rootless.git
    cd adbfs-rootless    

Build:

    make

Optional: If you have a separate copy of android-sdk and would
like to use that adb, copy the binary adbfs to the `android-sdk/platform-tools`
directory. If platform-tools is in your $PATH you can skip this step.

Create a mount point if needed (e.g. in your home directory):

    mkdir ~/droid

You can now mount your device (also from the platform-tools dir):

    ./adbfs ~/droid

If you want to trigger a media rescan after every operation, use the option `-o rescan`:

    ./adbfs -o rescan ~/droid

Have fun!

## MacOS

Install adb and fuse

    brew install --cask android-platform-tools
    brew install --cask macfuse

Check access to phone through adb

    adb devices

Clone the repository:

    git clone https://github.com/spion/adbfs-rootless.git
    cd adbfs-rootless

Build:

    make

Create a mount point if needed (e.g. in your home directory):

    mkdir ~/droid

Mount your device (You will be asked and have to allow fuse extension):

    ./adbfs ~/droid

Have fun!

## Troubleshooting

### Error: device not found

When running you get the following error:

```
--*-- exec_command: adb shell ls
error: device not found
```

Solution: Make sure that [USB Debugging is enabled][enable-usb-debug].

Then `fusermount -u /media/mount/path` before trying again. Note that if for any reason `fusermount` is not available in your system, you can use `sudo umount /media/mount/path` instead.

### Error: device offline

When running you get the following error:

```
--*-- exec_command: adb shell ls
error: device offline
```

Solution: Make sure that

1. Your android-sdk-tools are up to date. Newer versions
   of Android also require newer versions of adb. For more info, see 
   [this Stack Overflow post][error-device-offline].

2. You answer `Yes` when your phone asks whether it should allow the 
   computer with the specified RSA key to access the device.

Then `killall -9 adb; fusermount -u /media/mount/path` before trying again.


[enable-usb-debug]: http://www.droidviews.com/how-to-enable-developer-optionsusb-debugging-mode-on-devices-with-android-4-2-jelly-bean/
[error-device-offline]: http://stackoverflow.com/questions/10680417/error-device-offline
