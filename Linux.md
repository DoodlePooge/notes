# Installing Linux

**Index:**

- [Create Boot Drive](#create-boot-drive)
- [Use Boot Drive to Install OS](#boot-pc-on-boot-drive)
- [Handle Secondary Drives](#mounting-secondary-drives)
- [Installing Apps](#installing-apps)
- [Linux Gaming](#linux-gaming)
- [Fixes for Common Issues](#potential-issues)
- [Glossary](#glossary)

## Create Boot Drive

You will need an empty flash drive or external hard drive with at least 16 GB to be safe.

Pick your distro and download the latest LTS (Long-Term Support) image file (`.iso`).

### Windows

You will need to install [Etcher](https://etcher.balena.io/) to create a boot drive.

Once installed, all you do it:

1. Plug in the external drive that will become the boot drive
2. Open the Etcher application
3. With the app you select both the downloaded `.iso` and the external drive
4. Wait for it to process and done!

### Linux

#### Disks App from GNOME

<sub>Not all distros will have this app.</sub>

1. Open Disks.
2. Plug in Target drive.
3. Select Target drive in left hand column.
4. Select the three lines or dots on the upper right of the window.
5. Click Restore Disk Image.
    ![GNOME Disks app selecting Restore Disk Image](https://support.endlessos.org/screenshots/gnome-disks-menu-restore-disk-image.png)
6. Select the Linux ISO as Image to Restore.
    ![GNOME Disks app restore disk](https://support.endlessos.org/help-center/article_attachments/360002119232/img-05.png)
7. Follow the instructions.

#### Startup Disk Creator

We’re going to use an application called ‘Startup Disk Creator’ to write the ISO image to your USB stick. This is installed by default on Ubuntu, and can be launched as follows:

1. Insert your USB stick (select ‘Do nothing’ if prompted)
2. Open Applications
3. Use the search field to look for Startup Disk Creator
4. Select Startup Disk Creator from the results to launch the application\
  <sub>(In case the Startup Disk Creator is not installed, install it in a terminal by running `sudo apt install usb-creator-gtk`.)</sub>

5. Make sure the selections are correct and create the disk\
  ![Startup Disk App](https://assets.ubuntu.com/v1/48c17275-bionic-make-startup-disk.png)\
  <sub>When launched, Startup Disk Creator will look for the ISO files in your Downloads folder, as well as any attached USB storage it can write to.\
  It’s likely that both your Ubuntu ISO and the correct USB device will have been detected and set as ‘Source disc image’ and ‘Disk to use’ in the application window. If not, use the ‘Other’ button to locate your ISO file and select the exact USB device you want to use from the list of devices.</sub>

## Boot PC on Boot Drive

> [!WARNING]
> Before starting this process, please back-up any important files that may be on your main drive that you plan to put linux on.

Plug in the boot drive if it's not already, we will be restarting the computer and running the boot drive.

Here are some ways this can be achieved:

- The most universal way is restarting the computer and pressing the function key that your device uses to access the boot menu. Usually this is `F12`, but this can vary. You will see a screen showing the key to press before your login screen.
- Windows Advanced Startup:
  - In Windows 11, go to `Settings > System > Recovery`. Next to Advanced startup, select Restart now. On Blue screen select "Use a Device".
  - In Windows 10, go to `Settings > Update & Security > Recovery`. Under the Advanced startup section, select Restart. On Blue screen select "Use a Device".

Either menu should provide at least one option to choose to boot from. Select your boot drive.

### Ubuntu

When booting from the drive you created Ubuntu prompts you of what you would like to use the drive for.

![GNU GRUB Ubuntu Installation Options](https://i.sstatic.net/UZiPB.png)

> [!IMPORTANT]
> If you are installing on a machine that requires Nvidia Drivers you must select the option that has `(safe graphics)`

Follow the steps provided for installation to your preference.

Things I recommend during installation:

- Interactive installation
- **For computers with an Nvidia GPU** make sure you check third-party software\
![This is the caption for the image.](https://ubuntucommunity.s3.us-east-2.amazonaws.com/optimized/3X/f/f/ff8b742c5b79efaaed137928080a48b7ad260ba7_2_800x496.jpeg)
- Erase Disk and Install Distro
  - You'll have to select your main drive
  - NO ADVANCED FEATURES

You will then be asked to restart the computer and prompted to remove the boot drive to finish setting up the OS.

## Mounting Secondary Drives

You'll notice that your other hard drives and SSDs show up as external drives. We'll need to mount them and have the system recognize them as internal on startup.

### Reformat Drives

When switching from windows, the drives tend to be in format `ntfs` which is specific to windows. We'll want to make these `ext4`.

> [!WARNING]
> Before starting this process, please back-up any important files that may be on these drives as **reformating them with new partitions will wipe them clean**.

#### [Disks App from GNOME](https://docs.puri.sm/Software/PureOS/Tips/disks.html)

#### [Terminal](https://www.digitalocean.com/community/tutorials/create-a-partition-in-linux)

### Mounting on Boot

In order to add a disk to be a main file system we will need to edit the table in `fstab`.

`sudo nano /stc/fstab`

The file will look similar to this:

```bash
# /etc/fstab: static file system information.
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>

proc  /proc  proc  defaults  0  0
# /dev/sda5
UUID=be35a709-c787-4198-a903-d5fdc80ab2f8  /  ext3  relatime,errors=remount-ro  0  1
# /dev/sda6
UUID=cee15eca-5b2e-48ad-9735-eae5ac14bc90  none  swap  sw  0  0

/dev/scd0  /media/cdrom0  udf,iso9660  user,noauto,exec,utf8  0  0
```

You'll notice it's not a clean table. The commented line tells us what each column is for. Columns are determined by having at space. Basically, don't worry about making it pretty.

Here's some [context](https://man7.org/linux/man-pages/man5/fstab.5.html) to each column:

- *file system* - The identifier of the disk. Best practice to use UUID in case you need to swap ports.
- *mount point* - The directory that will be the root of this disk. (Your primary disk will always have `/` as the mount point as that is the true root of the computer)
- *type* - Format of the disk (`ntfs`, `ext4`, `vfat`)
- *options* - Mounting options. `defaults` is good for this.
- *dump* - This is `0` unless you wish to configure file dumping
- *pass* - Sort of determining the priority of mounting/booting disk. For extra disks like these we'll do a `1` or `2`.

We will want to add a line below the other file systems and add all our extra drives we want to treat as internal drives. To find the UUID you can run `lsblk -f` or in GNOME's Disks app.

Most of your added lines might look like this: 

```bash
/dev/disk/by-uuid/[found uuid of disk] /media/mnt ext4 defaults 0 2
UUID=[found uuid of disk] /secondary ext4 defaults 0 1
```

Reboot the computer to check that your changes worked.

## Installing Apps

Ubuntu's App store by default shows results that can be installed through snap. If you don't want snap you can specifically filter your searches for Debian packages instead.

> [!WARNING]
> *I DO NOT RECOMMEND INSTALLING APPS THROUGH FLATPACK OR SNAP*\
> Many times you run into permission issues (i.e. steam cannot access files in other locations like a secondary drive). It takes extra steps to configure permissions.

 If what you're looking for is not in the app store, the best place to look is on the websites for the apps themselves.

### Debian Packages

You downloaded a `.deb` file, now what?

To install this file it's a quick command.

`sudo dpkg -i ~/Downloads/exampleName.deb`

### AppImages

You downloaded an `.AppImage` file, now what?

For apps you'll likely keep coming back to, it's good to dedicate a folder for your AppImage files.

Create folder in your file explorer or in the terminal. Example `mkdir ~/AppImages`.

Move any AppImage files from downloads into that folder.

To run them you simply just call the path like a command. Example from home directory: `./AppImages/exampleName.AppImage`.

If it fails to run, you need to give yourself the correct permissions which can be done with this command:

`sudo chown a+x [path of AppImage file]/exampleName.AppImage`

## Linux Gaming

### Steam

To be able to run "Windows only" games go to `Steam -> Settings -> Compatibility` and check "Enable Steam Play for all titles" and restart steam.

## Potential Issues

- [Visual Glitches](#visual-glitches)
- [Discord Streaming Issues](#disable-wayland)
- [Login Loop](#login-loop)
- [Steam Storage](#steam-cant-find-other-drives)

### Visual Glitches

Depending on the severity of the GUI issues you can open the terminal screen in Ubuntu using `Ctrl+Alt+F4`. It will likely ask for your login.

Run `sudo ubuntu-drivers install`

If this does not fix the issue check your version with the recommended:

Your version -> run `cat /proc/driver/nvidia/version`

Look for the recommended one -> run `sudo ubuntu-drivers list`

Example output:

```bash
nvidia-driver-470
nvidia-driver-470-server
nvidia-driver-535
nvidia-driver-535-open
nvidia-driver-535-server
nvidia-driver-535-server-open
nvidia-driver-550
nvidia-driver-550-open
nvidia-driver-550-server
nvidia-driver-550-server-open (recommended)
```

To install a specific version (i.e. `nvidia-driver-535`) run `sudo ubuntu-drivers install nvidia:535`

### Disable Wayland

Having issues streaming on Discord? Wayland is probably why. You'll want to open up your terminal, we'll be editing `/etc/gdm3/custom.conf` to disable it.

Run `sudo nano /etc/gdm3/custom.conf`

```bash
...
[daemon]
# Uncomment the line below to force the login screen to use Xorg
# WaylandEnable = false

# Enabling automatic login
# AutomaticLoginEnable = true
# AutomaticLogin = user1

# Enabling timed login
#  TimedLoginEnable = true
#  TimedLogin = user1
#  TimedLoginDelay = 10
...
```

Uncomment the line `WaylandEnable = false`

```bash
...
[daemon]
# Uncomment the line below to force the login screen to use Xorg
WaylandEnable = false

# Enabling automatic login
# AutomaticLoginEnable = true
# AutomaticLogin = user1

# Enabling timed login
#  TimedLoginEnable = true
#  TimedLogin = user1
#  TimedLoginDelay = 10
...
```

Reboot your computer and check if it helped!

### Login Loop

If you made any changes to your `~/.bashrc` file this may cause problems. If you ran into a login loop I recommend reverting changes made to this file.

### Steam can't find other drives

There are 2 things to primarily check.

1. Make sure your disks are [mounted properly](#mounting-secondary-drives)
2. If you are using snap (and wish to continue to) for your steam installation you may need to grant steam access to the drive first: `sudo snap connect steam:removable-media`
3. If steam was installed with flatpack... undo that :)\
[See Installing Apps](#installing-apps)

## Glossary

**Daemon** - A program that runs in the background and runs tasks usually at start-up.

**Disk** - An umbrella term for computer components holding storage. (i.e. flash drives, HDD, SSD, CD)

**Distro** - Short for distribution. An operating system that includes the Linux kernel for its kernel functionality.

**Drivers** - Software that tells the computer how it should handle/use hardware components. (i.e. Some headsets, microphones, tablets need specific drivers installed to function)

**GNOME** - A common Open source GUI (Graphical User Interface) for linux systems.

**GPU** - Graphics Processing Unit.

**ISO file/image** - An exact copy of an entire optical disk such as a CD, DVD, or Blu-ray archived into a single file.

**Kernel** - The core of an Operating System. It manages the hardware's resources used by running applications.

**nano** - A file editor tool used in the terminal/commandline. Many files require permissions to edit so `sudo` is necessary.

**OS** - Operating System. A system of tools/programs that manage the communication between hardware and software. Includes GUI, file system, security, and the kernel (the core of managing the computer).

**sudo** - super user do. Used to run a command as if you are the root user.
