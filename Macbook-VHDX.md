# Booting to VHDx... on A Mac!

Note: This has only been tried on a Mac *without* a T2 Chip, so I have no idea what will happen if you try it on a Mac with a T2 Chip.
I also had SIP disabled just in case, but I am not sure if that will affect the actual results. 

Please contact me if you face issues

### PS: I DO NOT CLAIM ANY RESPONSIBILITY FOR ANY DAMAGE INCURRED

## Prerequisites:
1. A Mac
2. The Windows Support Files from the Boot Camp Assistant.
3. A Thumdrive
4. A VHDx already set up. Refer to: "Booting to VHDx: A guide" for more details
5. Another Storage device to transfer the VHDx

## Before we start:

Firstly, I will be disabling System Integrity Protection (SIP) just to be sure nothing inteferes with our process. To do so, hold ```CMD + R``` while the Mac is booting to get into Recovery Mode.

Once inside Recovery Mode, go to Utilities --> Terminal, and type the following command: ```csrutil disable```. Then, restart your mac.

## Preparing the Windows PE Boot Drive

To prepare Windows PE, you will require a Windows Environment to download the ADK and Windows PE. Refer to [this article](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-create-usb-bootable-drive) for the downloads the ```copype``` command used for creating the initial Windows PE Image

Next, we will follow [this article](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-mount-and-customize) to mount our image, and add our Mac BootCamp Drivers.

```cmd
Dism /Mount-Image /ImageFile:C:\path\to\winpe\image /index:1 /MountDir:C:\WinTest

Dism /Add-Driver /Image:C:\MountDir /Driver:C:\Path\To\Driver\Folder /recurse

Dism /Unmount-Image /MountDir:C:\MountDir /commit
```

You may also take the time to [optimise your image](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-optimize) as can be read about there.

## Flashing the Thumbdrive

Launch the Deployment and Imaging Tools Environment Promt as an Administrator

```cmd
MakeWinPEMedia /ISO C:\Path\to\WinPE\store C:\Path\To\WinPE\Iso
```

Then use Rufus to image the iso to your thumbdrive.

## Booting your mac

Hold the option key when booting, then select the EFI Boot option (your thumbdrive with WinPE should be plugged in before this, and your vhdx copied to either the same or another drive, ready to be copied. You may also use a network storage device, but I am not going to go into that)

Once booted, go into ```diskpart``` to ensure that you are able to see the internal disk of the Macbook.

```cmd
diskpart
list disks
```

Once the disks show up, we can continue with what we need to do, which is to
1. Format the Partition containing the VHDx with ntfs
2. Copy the VHDx
3. Install the Mac BootCamp Drivers into the VHDx
4. Create boot files to Macbook's EFI Partition
 
(ignore the rem words, those are comments.)

```cmd
list volumes
rem We now need to select the EfI Partition. This is usally a hidden partition.
select volume x
assign letter=S

select volume y rem where y is the partition that will hold the vhdx
format fs=ntfs quick label=yourlabel
exit

Copy L:\Vhdx.vhdx C:\Vhdx.vhdx rem "where L is the Drive letter of the drive with the VHDx"

diskpart
select vdisk file=C:\Vhdx.vhdx
attach vdisk
list volume 

R: rem where R is the mount point of the VHD
Dism /Image:R:\ /Add-Driver /driver:D:\$WinPEDriver$ /recurse rem where D is the drive with your macbook drivers
cd R:\Windows\system32
bcdboot R:\Windows /s S: /f UEFI
```

and DONE

## Resources used:
- https://discussions.apple.com/thread/250308247
- https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-create-usb-bootable-drive
- https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-mount-and-customize
- [VHDx Creation Guide](./README_VHDX.md)
