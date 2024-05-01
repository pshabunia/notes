# Android Data Backup

## Setup 
Install Android Debug Bridge (adb): https://developer.android.com/tools/adb
Enable debug mode on device: https://developer.android.com/studio/debug/dev-options#enable

## Export data
```shell
adb start-server
# connect the phone
# confirm debug connection on the phone if asked
adb devices
adb attach <device-id>
# cd target_directory 
# move a single file or entire directory
adb pull /sdcard/DCIM/Camera/ .
# move specific files
adb shell 'find /sdcard/DCIM/Camera -name "2023*" -print0' | xargs -0 -n 1 adb pull
```
