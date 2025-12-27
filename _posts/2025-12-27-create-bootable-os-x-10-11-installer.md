---
layout: post
title: Create bootable OS X 10.11 installation media from macOS 26
date: 2025-12-27 16:58 -0500
tags:
  - Apple
  - macOS
  - OS X
---

The following are steps that worked for me, to create a bootable USB copy of the OS X 10.11 installer while running macOS 26. This procedure was adapted from a [guide I found from 2015](https://forums.macrumors.com/threads/el-capitan-bootable-dvd.1923931/?post=22039675#post-22039675).

I suspect this process would be similar for other versions that are distributed as disk images (10.7 through 10.12), but I haven't tested any except 10.11.

1. Download the disk image from [Apple’s support page](https://support.apple.com/en-us/102662). This will download a file called `InstallMacOSX.dmg` that’s about 6.2 GB.
1. Open `InstallMacOSX.dmg` to mount a volume called `Install OS X`.
1. Using [Pacifist](https://www.charlessoft.com/), open the `InstallMacOSX.pkg` package contained within the `Install OS X` volume.
1. Under the **Package Resources** tab in Pacifist, expand `InstallMacOSX Contents` and the second-level `InstallMacOSX Contents` to find a file called `InstallESD.dmg`. Copy `InstallESD.dmg` to your hard drive.
1. Quit Pacifist, eject the `Install OS X` volume, and delete `InstallMacOSX.dmg`.
1. Open `InstallESD.dmg` to mount a volume called `OS X Install ESD`.
1. In Disk Utility, create a new Blank Image, with the following parameters:
	- Save As: `Bootable_OS_X_Installer_10_11` (or whatever you prefer)
	- Name: OS X Base System
	- Size: 7600 MB
	- Format: Mac OS Extended (Journaled)
	- Encryption: None
	- Partitions: Apple Partition Map
	- Image Format: DVD/CD master (Be aware when you select this, the Size may change to `100 MB`. Make sure the size is correct before clicking **Save**.)
1. In Disk Utility, the newly-created blank image should already be mounted and listed as `OS X Base System` (if not, open `Bootable_OS_X_Installer_10_11.dvdr.cdr` to mount it). Select the `OS X Base System` volume, and choose **Restore**, and then for **Restore from…** select **Image…**. Within the root of the `OS X Install ESD` volume, select `BaseSystem.dmg` and proceed with restoring from that image. (You may need to show hidden files to see `BaseSystem.dmg`; you can do that with `Command + Shift + .`.)
1. Within the `OS X Base System` volume, you will now see files present after there restore (`Applications`, `Install OS X El Capitan`, `System`, etc). Open the `System` folder, and then open the `Installation` folder. **Delete** the alias called `Packages`.
1. Copy the `Packages` folder from the root of the `OS X Install ESD` volume into `/System/Installation` on `OS X Base System` (i.e., replace the alias you just deleted with the folder). This folder is over 5 GB, so it may take several seconds.
1. Also copy both `BaseSystem.chunklist` and `BaseSystem.dmg` from the root of the `OS X Install ESD` volume (they will likely be hidden) to the root of the `OS X Base System` volume. The files should now appear next to things like `Applications`, `Install OS X El Capitan`, `System`, etc.
1. Eject the `OS X Install ESD` volume, and delete `InstallESD.dmg`.
1. Rename the `OS X Base System` volume to something more descriptive, like `Install Mac OS X El Capitan`, and eject that volume.
1. Rename `Bootable_OS_X_Installer_10_11.dvdr.cdr` to `Bootable_OS_X_Installer_10_11.iso`.
1. Use [Etcher](https://etcher.balena.io/) to write the ISO to a USB drive. Ignore the warning about the image missing a partition table.
1. Delete `Bootable_OS_X_Installer_10_11.iso` if you no longer need it.
