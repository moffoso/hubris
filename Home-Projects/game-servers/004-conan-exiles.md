install dependancies
```bash
sudo add-apt-repository multiverse
sudo dpkg --add-architecture i386
sudo apt update && sudo apt upgrade
sudo apt install software-properties-common lib32gcc-s1 steamcmd
sudo apt install wine xvfb 
```

adding steam user
```bash
sudo useradd -m -s /bin/bash steam
sudo -iu steam
# change to the home directory
cd $HOME
mkdir conan_exiles
cd conan_exiles
```

if you get the error "steamcmd not found"
```bash
export PATH="/usr/games/:$PATH"
```

install conan-exiles-dedicated
```bash
+force_install_dir /home/steam/conan_exiles +login anonymous +app_update 443030 +quit
```

configure wine
```bash
export WINEARCH=win64
export WINEPREFIX=/home/steam/.wine64 
# run the server to create the config files
xvfb-run --auto-servernum --server-args='-screen 0 640x480x24:32' wine /home/steam/conan_exiles/ConanSandboxServer.exe -log
```

update the server
```bash
export PATH="/usr/games/:$PATH" 
steamcmd +@sSteamCmdForcePlatformType windows 
login anonymous 
app_update 443030
```

adding mods
```bash
# download thralls are alive
steamcmd +@sSteamCmdForcePlatformType windows +login anonymous +workshop_download_item 440900 2791028919
# create mod directory in /home/steam/conan-exiles
mkdir Mods
# create modlist.txt in Mods/
nano modlist.txt
# add line with path to mod
/home/steam/Steam/steamapps/workshop/content/440900/2791028919/SerenaBadlands.pak
```

create service to autostart conan-exiles on startup
```bash
[Unit]
Description=Conan Exiles Server
After=network.target

[Service]
# If you define the environment variables here you can remove them from your .profile
# but I don't think it'll do any harm to have them defined twice, as long as the definitions
# are the same
Environment=WINEARCH=win64
Environment=WINEPREFIX=/home/steam/.wine64
# Set the user, group and relative directory
WorkingDirectory=/home/steam/conan_exiles/
User=steam
Group=steam
# Run xvfb-run to start the server
ExecStart=/usr/bin/xvfb-run --auto-servernum --server-args='-screen 0 640x480x24:32' wine /home/steam/conan_exiles/ConanSandboxServer.exe -log

[Install]
WantedBy=multi-user.target
Alias=conan.service
```

### Revamped Conan server and Steamcmd
create and configure steam user as root
```bash
sudo useradd -m steam
sudo passwd steam
# add steam to sudoers file
sudo usermod -aG sudo steam 
```

change to steam user and enter home directory
```bash
sudo -u steam -s
cd /home/steam
```

install steam with native package management
```bash
sudo add-apt-repository multiverse
sudo dpkg --add-architecture i386
sudo apt update && sudo apt upgrade
sudo apt install software-properties-common lib32gcc-s1 steamcmd
sudo apt install wine xvfb 
```

create directory to install server
```bash
mkdir conan-exiles
```
start steamcmd and let it update 
```bash
cd /usr/games
./steamcmd
```

install conan-exiles dedicated server
```bash
force_install_dir /home/steam/conan_exiles +login anonymous +app_update 443030 +quit  
```






___

### conan-exiles dedicated server
create and configure steam user as root
```bash
sudo useradd -m steam
sudo passwd steam
# add steam to sudoers file
sudo usermod -aG sudo steam 
```

change to steam user and enter home directory
```bash
sudo -u steam -s
cd /home/steam
```

update & upgrade
```bash
sudo apt update && sudo apt upgrade
```

install steam with native package management
```bash
sudo add-apt-repository multiverse 
sudo dpkg --add-architecture i386 
sudo apt update && sudo apt upgrade 
sudo apt install software-properties-common lib32gcc-s1 steamcmd 
sudo apt install wine xvfb 
```

temporarily set variables to modify steamcmd to run commands
```bash
export PATH="/usr/games/:$PATH"
export WINEARCH=win64
export WINEPREFIX=/home/steam/.wine64
```

log into the steam account and run this command
```bash
steamcmd +@sSteamCmdForcePlatformType windows +force_install_dir /home/steam/conan_exiles +login anonymous +app_update 443030 +quit
```

stop the server with CTRL+C and modify server ini files under home/steam/conan_exiles/ConanSandbox/Saved/Config/WindowsServer

create a service to run this from outside of the steam user
with user dan
```bash
nano /etc/systemd/system/conanexiles.service
# following lines:
[Unit]
Description=Conan Exiles Server
After=network.target

[Service]
# If you define the environment variables here you can remove them from your .profile
# but I don't think it'll do any harm to have them defined twice, as long as the definitions
# are the same
Environment=WINEARCH=win64
Environment=WINEPREFIX=/home/steam/.wine64
# Set the user, group and relative directory
WorkingDirectory=/home/steam/conan_exiles/
User=steam
Group=steam
# Run xvfb-run to start the server
ExecStart=/usr/bin/xvfb-run --auto-servernum --server-args='-screen 0 640x480x24:32' wine /home/steam/conan_exiles/ConanSandboxServer.exe -log

[Install]
WantedBy=multi-user.target
Alias=conan.service
```

start and enable service
```bash
sudo systemctl start conanexiles
sudo systemctl enable conanexiles
```

```bash
xvfb -run --auto-servernum --server-args='-screen 0 640x480x24:32'wine /home/steam/conan_exiles/ConanSandboxServer.exe -log
```