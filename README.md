# Practices for installing and running an Android emulator in headless mode on Ubuntu servers

### REFERENCES:
- https://gist.github.com/Abyss-W4tcher/f1833623c975193446315d48c106750e
- https://gist.github.com/atyachin/2f7c6054c4cd6945397165a23623987d
- https://gist.github.com/nhtua/2d294f276dc1e110a7ac14d69c37904f

# AArch64 Android emulation and kernel cross-compilation

The following assumes you are using a AArch64 host.

## Android SDK installation

Setup SDK and emulator :
```sh
# https://developer.android.com/studio/index.html#command-line-tools-only
sudo apt-get install unzip openjdk-17-jdk gradle -y
mkdir -p Android/cmdline-tools
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip -O commandlinetools-linux-11076708_latest.zip
unzip commandlinetools-linux-11076708_latest.zip -d Android/cmdline-tools
mv Android/cmdline-tools/cmdline-tools Android/cmdline-tools/latest
export ANDROID_HOME=$(pwd)/Android
export PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/cmdline-tools/latest:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin:$ANDROID_HOME/platform-tools:$PATH"

# Get emulator download link from https://ci.android.com/builds/branches/aosp-emu-master-dev/grid (needs a manual download, or get the link from your navigator downloads and paste it in the following wget parameter)
# e.g. https://ci.android.com/builds/submitted/8632828/emulator-linux_aarch64/latest/sdk-repo-linux_aarch64-emulator-8632828.zip
wget '' -O emulator.zip # 31.3.8
unzip -qq emulator.zip -d Android/

# Edit the XML <revision> tag at the end with your emulator version
curl 'https://chromium.googlesource.com/android_tools/+/refs/heads/main/sdk/emulator/package.xml?format=TEXT' | base64 -d > Android/emulator/package.xml
```

Setup dependencies :
```sh
yes | sdkmanager --licenses
sdkmanager "build-tools;34.0.0" "platform-tools" "platforms;android-34" "tools"
```

### Ressources 

- https://gist.github.com/jason-s-yu/30375db45c1f71c1259e042d216e4bd3
- https://gist.github.com/atyachin/2f7c6054c4cd6945397165a23623987d

## Emulation

Fetch avd system image :
```sh
# sdkmanager --list # List available packages
TARGET_IMAGE='system-images;android-27;default;arm64-v8a'
sdkmanager $TARGET_IMAGE
```

Create avd :
```sh
# If you encounter any java error when launching avdmanager, please follow https://stackoverflow.com/a/62610046
yes '' | avdmanager create avd -n my_avd -k $TARGET_IMAGE
```

Start emulation :
```sh
# Headless
emulator -avd my_avd -no-snapshot -no-window

# Graphical
emulator -avd my_avd -no-snapshot -gpu swiftshader_indirect 
```

Delete avd : 
```sh
avdmanager delete avd -n my_avd
```

# Run a Headless Android Device on Ubuntu server (no GUI) 
```sh
#!/bin/bash -i
#using shebang with -i to enable interactive mode (auto load .bashrc)

set -e #stop immediately if any error happens

# Install Open SDK
apt update
apt install openjdk-8-jdk -y
update-java-alternatives --set java-1.8.0-openjdk-amd64
java -version

# Install SDK Manager
# you can find this file at https://developer.android.com/studio/index.html#downloads - section command line only
cd ~ && wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip
ANDROID_HOME=/opt/androidsdk
mkdir -p $ANDROID_HOME
apt install unzip -y && unzip sdk-tools-linux-4333796.zip -d $ANDROID_HOME

echo "export ANDROID_HOME=$ANDROID_HOME" >> ~/.bashrc
echo 'export SDK=$ANDROID_HOME' >> ~/.bashrc
echo 'export PATH=$SDK/emulator:$SDK/tools:$SDK/tools/bin:$SDK/platform-tools:$PATH' >> ~/.bashrc
source ~/.bashrc

# Install Android Image version 28
yes | sdkmanager "platform-tools" "platforms;android-28" "emulator"
yes | sdkmanager "system-images;android-28;google_apis;x86_64"
emulator -version

echo "INSTALL ANDROID SDK DONE!"
echo "run 01.emulator-up.sh [new device name] to start emulator"

#!/bin/bash -i
#using shebang with -i to enable interactive mode (auto load .bashrc)
#this script was inspired from https://docs.travis-ci.com/user/languages/android/

set -e #stop immediately if any error happens

avd_name=$1

if [[ -z "$avd_name" ]]; then
  avd_name="avd28"
fi

#check if emulator work well
emulator -version

# create virtual device, default using Android 9 Pie image (API Level 28)
echo no | avdmanager create avd -n avd28 -k "system-images;android-28;google_apis;x86_64"

# start the emulator
emulator -avd avd28 -no-audio -no-window &

# show connected virtual device
adb devices
```
