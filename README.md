**Please note:** The steps in this How-To were tested on Ubuntu 22.04 with Gnome 3 desktop. While the installation of DOSBox itself will not be a problem on a system with another package manager, I did not dig into other desktop environments.

## Usecase

The aim of this approach is to avoid multiple copies of the same files inside a system where multiple users use DOSBox. In my case, I want install old DOS games so that every user on the system (e.g. my son) can use them, but not every user should have to configure everything by himself.

All users on the same physical machine shall:
a) use the same DOSBox configuration file
b) use the same 'DOS harddrive' (folder the DOS binaries are stored in)
c) shortcuts to start DOS applications directly shall be available system wide

## Prerequisities

a) (On linux,) DOSBox only checks a single location for a configuration file, which is `~/.dosbox/dosbox-<version>.conf` in the current users home directory. This would mean that the config file would have to be copied in each users home directory, copied when new users are added, and each file would have to be updated when the configuration changes. But we can pass the `-conf` parameter, where we can specify another location for the config file.

b) As above, when new DOS software is installed on the 'DOSBox harddrive' (which is essentially a folder in the linux filesystem that gets mounted as e.g. drive `C:` inside DOSBox), every user shall be able to use it with no extra effort in disk space.

c) If shortcuts are created to start DOS applications directly from the GNOME Shell, they, too, shall be available for all users on the machine with no extra configuration effort. The target application can be passed as the last parameter to the `dosbox` call with its full path on the linux filesystem. So the call to `dosbox -exit /path/to/drive_c/games/play.exe` brings up dosbox and starts the application with no additional interaction. The `-exit` switch closes the DOSBox window when the target application terminates. Wrapping this in a `.desktop` file as they are used in the GNOME 3 Shell then integrates DOS applications better than any other DOSBox launcher. Unfortunately, searching for 'custom startes on GNOME 3' or similar in most cases only explains how to create those files for the current user, which - again - would mean multiple creation/update steps for each user.

## Solution

~~~
# install DOSBox
sudo apt install dosbox

# create a folder for all common files
sudo mkdir /opt/dosbox

# create our DOSBox 'harddrive'
sudo mkdir /opt/dosbox/drive_c

# move and rename your DOS files' parent folder to our new drive_c
sudo mv /path/to/your/dosfilesparentfolder /opt/dosbox/drive_c

# all users should be given write access to drive_c, e.g. to save games
sudo chmod 0777 -R /opt/dosbox/drive_c

# move existing DOSBox config to common folder
# (will be created on first start of dosbox if it doesn't exist)
sudo mv ~/.dosbox/dosbox-0.74-3.conf /opt/dosbox/
~~~
Let's add a mount command for our newly created drive_c in
`/opt/dosbox/dosbox-0.74-3.conf`:
```diff
[autoexec]
# Lines in this section will be run at startup.
# You can put your MOUNT lines here.
+MOUNT C /opt/dosbox/drive_c
+C:
```
We now add the `-conf` parameter on the `Exec=` entry in
`/usr/share/applications/dosbox.desktop`:
```diff
-Exec=dosbox
+Exec=dosbox -conf /opt/dosbox/dosbox-0.74-3.conf
```
From now on, when DOSBox is started from the GNOME 3 application menu,
the common configuration file is used, and the common drive_c is automatically
mounted on startup and changed into to. So a) and b) are already accomplished.

For c), let's look at a sample application `sample.exe`, which we assume to be located inside
our drive_c at `/opt/dosbox/drive_c/example/sample.exe`. We create a new file `sample.desktop`
in `/usr/share/appplications/` (you need root privileges for that):
```desktop
[Desktop Entry]
Name=Sample Application
Exec=dosbox -noautoexec -exit -conf /opt/dosbox/dosbox-0.74-3.conf /opt/dosbox/drive_c/example/sample.exe
Terminal=false
Type=Application
Keywords=dosbox;
Icon=/opt/dosbox/icons/sample.png
```
A few notes on the `Exec=` line:
- `noautoexec` prevents the commands in the `[autoexec]` section of the configuration file to be executed
- `-exit` closes the dosbox window when the application terminates
- `-conf` specifies the configuration file to use; you can use a specific one for each application if needed
- the last argument is the full path to the application's executable

All `.desktop` files created in `/usr/share/applications` are accessible for all users on the system. Repeat for every application you want to access from the GNOME 3 application menu.

That's it!
