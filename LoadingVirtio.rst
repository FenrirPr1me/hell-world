How To Load VIRTIO drivers from CLI using a Windows Recovery Live CD

With the affected VM powered off, inject the Windows Installation ISO in one “tray” and the Scale tools ISO (which is what has the virtue drivers on it) in the other tray
Change the boot order on the VM card, dragging the VSD (virtual hard dive) down into the “Do Not Boot” section, and pull the CD tray with the Windows ISO up so that it is the only thing in the “Boot” section.

Power on the VM, and hit any button on the keyboard to boot from the live CD

Once in the menu, choose “repair computer” or a similar option depending on the OS.

Choose to open a command prompt.

The system will start you with the drive letter X:

Type in DISKPART to enter the disk partition viewer tool.

Once open, type LIST VOL to display all viewable volumes.  The return should only display the letter paths for the two ISO that are in the virtual CD trays.

Pick the letter for the scale virtio driver ISO, for this example we’ll say it is drive letter E.

Type exit to leave the dispart tool.

Type “ E: “ without the quotation marks to change the targeted path to the scale ISO.

Type in “ dir “ then hit enter to display the folders in that ISO.

You should see ‘drivers’ in the list.

Type “ cd drivers “ to change the directory to the drivers folder.

Then type “ dir “ and hit enter once more to display the contents of that folder. You should see “ stor “ in the list (which stands for storage).

Type “ cd stor “ to change the directory to the stor folder.

Once more, type “ dir “ then hit enter to display the contents of that folder.  You will see a list of OSes.  Chose the appropriate one for the OS you’re working to restore.
 <for this example we’ll use Windows Server 2016>

Type “ cd 2k16 “ then hit enter to open the folder holding the storage drivers for Windows Server 2016.

Type “ dir “ to display the contents of the folder.  Depending on the age of your OS you will either see x86 (for 32-bit OSes) or amd64 (for 64-bit OSes)

Type “ cd amd64 “ to enter that folder.

Type “ dir “ and hit enter to see the contents of the folder.  You should see viostor.inf in the list. 

Type “ drvload viostor.inf “ and hit enter to load the drivers.

To make sure the VM can see that VSDs now, type in DISKPART again to enter the tool and type “ list vol “ to see if the drives are viewable now.

If they are now present, type exit and hit enter to exit the DISKPART tool.

Type exit and hit enter again to leave the CMD prompt.
