```sh
sudo pacman  -S adb

sudo pacman  -S nodejs
# npx stands for `node package executor`

sudo pacman -S android-studio
# sudo pacman  -S jdk-openjdk
```

```sh
sudo pacman -S jdk17-openjdk
sudo archlinux-java set java-17-openjdk
```
#### set env `bashrc` or `zshrc`

```sh
export ANDROID_HOME=/home/ohm/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

or may be 
```sh
export ANDROID_HOME=/opt/android-sdk
export ANDROID_SDK_ROOT=$ANDROID_HOME
export PATH="$PATH:$ANDROID_HOME/platform-tools"
export PATH="$PATH:$ANDROID_HOME/emulator"
export PATH="$PATH:$ANDROID_HOME/cmdline-tools/latest/bin"
```


> [!Note]
> if path is not properly set then `android/local.properties`
```js
sdk.dir=/home/ohm/Android/Sdk
```

✅ 1) Install required Android SDK packages (Command‑Line Tools + platform‑tools)
```sh
sudo pacman -S android-sdk-platform-tools android-sdk-cmdline-tools-latest
```

```sh
yes | sdkmanager --licenses
```