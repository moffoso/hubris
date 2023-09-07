> <u>Status as of 28.03.2023 16:52</u>
> players disconnect after mapvoting and before loading the new lobby

created user steam
```shell
sudo adduser steam
sudo adduser steam sudo
```

change to user steam and switch to steam home directory
```bash
sudo -u steam -s
cd
```

add multiverse for 32-bit architecture applications
```bash
sudo add-apt-repository multiverse
 sudo apt install software-properties-common
 sudo dpkg --add-architecture i386
 sudo apt update
```

install steamcmd
```bash
sudo apt install lib32gcc-s1 steamcmd
```

create directory for steam
```bash
sudo mkdir games
cd games
sudo mkdir kf2
cd kf2
```

create install/update script for kf2
```bash
sudo nano updatekf2.txt
#add following lines to the txt
login anonymous
force_install_dir /home/steam/games/kf2
app_update 232130 validate
quit
```

goto /usr/games to execute script as follows
```bash
cd /usr/games
./steamcmd +runscript /home/steam/games/kf2/updatekf2.txt
```

start kfserver
```bash
cd /home/steam/Steam/steamapps/common/kf2server
./Binaries/Win64/KFGameSteamServer.bin.x86_64 kf-bioticslab
```

webinterface
10.130.5.130:8080

config
```bash
sudo nano /Steam/steamapps/common/kf2server/KFGame/Config/LinuxServer-KFGame.ini
```
