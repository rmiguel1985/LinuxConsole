# LinuxConsole   

This is a tutorial to create a **Linux based console to play steam and retro games**. I found out that the options to add **retrogaiming** in **Steam** are not as good as a dedicated system, so for that reason I created this solution.
  
  
## Main Concept  
  Create a console with dual boot between Bazzite and Batocera
- **On fresh start:** Bazzite boots up.  
- **Boot batocera:** From Bazzite there will be a custom game that is just a script to boot to grub2 menu selecting by default batocera. After reboot Batocera entry in Grub will be selected and Batocera will booot.
- **Come back to Bazzite:** just reboot from batocera, Bazzite will be selected by default in Grub menu and will boot.  


## Notes
Im not affilitaed with **Bazzite**, **Batocera** or **Xubuntu**. Contact projects to get support from them
  
## Downloads  
  
- **Download Bazzite for your mini pc:** go to bazzite and download [link](https://bazzite.gg/).  
- **Download Batocera:** you need to download tar.xz [link](http://batocera.org/upgrades/x86_64/stable/last/boot.tar.xz) (the link points to the las stable version)
- **Xubuntu**: I choose this distro just to live boot and redimension bazzite partition (you could use the one you feel comfortable with) [link](https://xubuntu.org/download/).  
  
  
  
## Tutorial 

### Step 1: Install Bazzite
Follow the steps indicated on the official documentation [link](https://docs.bazzite.gg/General/Installation_Guide/)

### Step 2: Free Up Disk Space (Resize Bazzite)
1. Boot **Xubuntu**.
2.  Open the **GParted** application from the applications menu.
3.  In the top-right corner, select your main SSD (e.g. `/dev/nvme0n1`).
4.  Locate the main Bazzite partition (formatted as `BTRFS`), right-click it, and select **Resize/Move**.
5.  Drag the right edge of the partition to the left to free up space (e.g. free up 100 GB for ROMs), then confirm by clicking **Resize/Move**.
6.  Click the **green checkmark (Apply All Operations)** button in the top toolbar to apply the changes.
7.  Once finished, you will see a block of gray space labeled **Unallocated**.

> ⚠️ **Important:** Although GParted is a very safe tool, resizing a partition with an operating system in use involves modifying the file structure. Ensure that the Mini PC does not shut down or disconnect from the power supply during the process. Take in account that the Bazzite install could be broken.


### Step 3: Partition the Drive for Batocera (GParted)

1. Boot **Xubuntu**.
2. Open **GParted** from the applications menu.
3. In the top-right drop-down menu, select your main SSD (e.g. `/dev/nvme0n1`).
4. In the unallocated space (gray), create the following two partitions:

    | Partition | Size | File System | Label |
    |---|---|---|---|
    | **Batocera OS** | 6144 MB (6 GB) | FAT32 | `BATOCERA` |
    | **Batocera Data** | Remaining space | BTRFS | `SHARE` |

6. Click the **green Checkmark (Apply)** button to apply the operations, then close GParted.

> ⚠️ **Important:** It is essential to respect partition labels names, them must be `BATOCERA`(for **Batocera OS**) and `SHARE`(for **Batocera Data**) in **UPPERCASE**.

> ℹ️ For the **Batocera Data** partiton we are using BTRFS to make it visible from Bazzite. So it will possible to move retro games (legit security copies from your original games) from Bazzite to Batocera Rom folder.

### Step 4: Copy the Batocera Files

1. Restart the Mini PC and log in to **Bazzite Desktop Mode**.
2. Open your file manager and locate the new partition named `BATOCERA` (6 GB).
3. Extract the `boot.tar.xz` file you downloaded from Batocera.
4. Copy **all the extracted contents** directly into the `BATOCERA` partition.


### Step 5: Configure GRUB2 in Bazzite

Bazzite uses **GRUB2** as its bootloader. We are going to add the entry for Batocera:

1. Open a terminal and edit the custom configuration file:
   ```bash
   sudo nano /etc/grub.d/40_custom
   ```

2. Add the following lines at the end of the file:

   ```bash
   menuentry "Batocera" {
       search --set=root --label BATOCERA
       linux /boot/linux label=BATOCERA console=tty3 quiet loglevel=0 vt.global_cursor_default=0
       initrd /boot/initrd.gz
   }
   ```

3. Save by pressing **Ctrl + O**, press **Enter**, and exit with **Ctrl + X**.

4. Regenerate the GRUB2 configuration in Bazzite by running:

   ```bash
   ujust regenerate-grub
   ```

### Step 6: Create a script to launch Batocera
1. In the Bazzite terminal, open the permissions settings:
     ```bash
         sudo visudo
     ```
2. Add these lines to the very end of the file (replace `your_user_name` with your actual username):
     ```bash
         your_user_name ALL=(ALL) NOPASSWD: /usr/bin/grub2-editenv 
         your_user_name ALL=(ALL) NOPASSWD: /usr/sbin/grub-reboot
     ```
 3. Save and close the editor.
 4. Create the script file by running:
     ```bash
         nano ~/Projects/batocera/boot-batocera.sh
     ```
5. Paste the following code:
     ```bash
        #!/bin/bash
        sudo grub2-editenv /boot/grub2/grubenv set next_entry="Batocera" 
        steamos-reboot-now
     ```
     > ℹ️ depending on the os selected to install maybe we will need to use **reboot** instead of **steamos-reboot-now**
6. Save the file, exit the editor, and grant execution permissions:
     ```bash
        sudo chown $USER:$USER ~/Projects/batocera/boot-batocera.sh
        sudo chmod 755 ~/Projects/batocera/boot-batocera.sh
     ```

### Step 7: Create the Bazzite Shortcut to Batocera in Steam
1.  Open **Steam** in desktop mode.  
2.  Click **Add a Product** → **Add a Non-Steam Game...**    
3.  Select the `boot-batocera.sh` file.    
4.  In the shortcut properties in Steam, rename it to **"Batocera"** and assign a custom icon if desired.
   
	4.1  There are images in the repo for logo, grid and banners. 



## How to use
- **From Bazzite to Batocera:** Select the *"Batocera"* shortcut in your Steam library. 
- **From Batocera to Bazzite:** Go to the Batocera main menu and select **Quit -> Restart System**. The PC will boot into Bazzite by default.

## Batocera
**Batocera.linux** is an open-source, retro-gaming distribution built specifically for emulation. Takea a look at their [wiki](https://wiki.batocera.org/) to start using it.

## Contributing  
  
- Submit a **[Pull Request](https://help.github.com/articles/about-pull-requests/)** to add new features or improvements.  
  
## License  
  
Copyright © 2025 [Ruben Miguel Corcoba](https://github.com/rmiguel1985)  
  
This program is free software: you can redistribute it and/or modify it    
under the terms of the **GNU General Public License v3.0** or later.  
  
See the full license in [LICENSE](LICENSE) or at <https://www.gnu.org/licenses/>.  
  
