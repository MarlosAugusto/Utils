## [install WSL 2](https://docs.microsoft.com/pt-br/windows/wsl/install-win10) - *if necessary*


## [install Android Studio](https://developer.android.com/studio) and set the environment variables - *if necessary*


## install JDK - *on WSL2*
- `sudo add-apt-repository ppa:openjdk-r/ppa`
- `sudo apt-get update`
- `sudo apt-get install openjdk-8-jdk`

- *o validate run `java -version`*


## install React Native Cli
- `npm install -g react-native-cli`
- *to validate run `react-native -v`*

## [download command line tools for linux](https://developer.android.com/studio) and extract in home dir - *on WSL2*
- `mkdir -p ~/Android/cmdline-tools`
- `sudo apt install unzip`- *if necessary*
- `unzip commandlinetools-linux-XXXXX_latest -d ~/Android/cmdline-tools`

## set environment variables in config file: home/*(.bashrc | .bash_profile | .profile | .zshrc)* - *on WSL2*
```
JAVA_HOME=$(dirname $( readlink -f $(which java) ))
JAVA_HOME=$(realpath "$JAVA_HOME"/../)
export JAVA_HOME

export ANDROID_HOME=$HOME/Android
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/tools:$PATH
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/tools/bin:$PATH
export PATH=$PATH:$ANDROID_HOME/platform-tools:$PATH
```
- restart the terminal or update the `.zshrc` file
- run `sdkmanager "platform-tools"` to init configs

## Enable adb server access from WSL2
- in config file:
```
export WSL_HOST=$(tail -1 /etc/resolv.conf | cut -d' ' -f2)
export ADB_SERVER_SOCKET=tcp:$WSL_HOST:5037
```
- or if it doesn't work with that:
install (`sudo apt-get install socat`)
and run `socat -d -d TCP-LISTEN:5037,reuseaddr,fork TCP:$(cat /etc/resolv.conf | tail -n1 | cut -d " " -f 2):5037`

## Enable metro bundler access on Windows - *in PowerShell(admin mode)*
```
iex "netsh interface portproxy delete v4tov4 listenport=8081 listenaddress=127.0.0.1" | out-null;
$WSL_CLIENT = bash.exe -c "ifconfig eth0 | grep 'inet '";
$WSL_CLIENT -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';
$WSL_CLIENT = $matches[0];
iex "netsh interface portproxy add v4tov4 listenport=8081 listenaddress=127.0.0.1 connectport=8081 connectaddress=$WSL_CLIENT"
```

## start server in Windows
- PowerShell:
```
adb kill-server
adb -a nodaemon server start
```
- or WSL2:
```
adb.exe kill-server
adb.exe -a nodaemon server start
```

## run - in WSL2
- create react native project or navigate to exists project:
```
npx react-native init <projectName>` | `cd <projectName>
```
- start the emulator (if not exists create on AndroidStudio):
```
emulator.exe @EMULATOR_NAME
```
- start metro server:
```
npx react-native start
```
- run project(to find out the device id run `adb devices`)
```
npx react-native run-android --deviceId emulator-5554
```

## references
- [Configurando projeto React Native usando o Windows WSL2](https://medium.com/@rafaelnogueira/configurando-projeto-react-native-usando-o-windows-wsl2-1ce9efec02c1)
- [Building a react native app in WSL2](https://gist.github.com/bergmannjg/461958db03c6ae41a66d264ae6504ade#enable-access-to-adb-server-from-wsl2)
