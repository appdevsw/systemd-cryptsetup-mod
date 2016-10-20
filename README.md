# systemd-cryptsetup-mod
**Automatically unlocking additional Truecrypt or Luks volumes during boot, without asking password twice.**

The solution was tested on <b>Fedora 24  64-bit</b>. <br>
Additional containers should have the same passwords as an encrypted root.

 - Download systemd form https://github.com/systemd/systemd.     Use
    `Clone or download` button.
    
 - Unpack downloaded archive and compile it.     You have to do all
    standard steps like configure, install missing libraries and finally
    make.     Don’t be afraid. We need only one application, not the
    entire systemd.
    
 - In a text editor open file `cryptsetup.c` located in
    `systemd-master/src/cryptsetup/` directory.    
    Modify it, like I did in this github repository: <br>
    	https://github.com/appdevsw/systemd-cryptsetup-mod/commits/master/cryptsetup.c.<br>
    There are two pieces of code between lines
        
	>    //---------------- new code begin     
	>
	>   //---------------- new code end

    Copy and paste these fragments to your edited file in the right places.<br>
    You can also view the differences between the original and modified version:<br>
    https://github.com/appdevsw/systemd-cryptsetup-mod/commit/b49450409bd9cabeb9f055262c175b6a54561ae5#diff-babd06e0454f527a616eb6cf3796ed8c<br>
    
    
 - Run make again. After a successful compilation we need two files
    from `.libs` subdirectory: 
    
    > systemd-cryptsetup	 	
    > libsystemd-shared-230.so
   
    Check if they are there.

 - **Next commands should be executed as root.**

       

 - Create a new file in `/etc` directory, named `crypttab-other.conf`.  
    The file will contain the information about additional encrypted
    containers.     Each line should have 4 words:<br>
           (1) First word from the `/etc/crypttab` file. This is usually the symbol of an encrypted root.<br>
           (2) `truecrypt` or `luks`<br>
           (3) Path to  device/partition thet should be unlocked<br>
           (4) Mount point<br>

	>            Example: 	 
	>               lvm_crypt   truecrypt   /dev/sda3   mytc
	
	>            It means: 
	>               Unlock the truecrypt partition /dev/sda3 
	>               with the same password as lvm_crypt 
	>               and mount it as /dev/mapper/mytc

 - Now we have 3 files and we need them inside `initramfs` image, which is
   used to boot our system.     We have to create a new `initramfs` using
   `dracut` utility and force the `dracut` to include our files.

   

 - Include `crypttab-other.conf` into `initramfs` :<br>
	   Edit `/usr/lib/dracut/dracut.conf.d/01-dist.conf` and modify the line with `install_optional_items` adding a reference to our  `crypttab-other.conf` file. <br><br>
             example: 
               
	> install_optional_items+=" vi /etc/virc ps grep cat rm
	> /etc/crypttab-other.conf"

 - Include `systemd-cryptsetup` and `libsystemd-shared-230.so` into
   initramfs : <br>   
	   a) Copy  `libsystemd-shared-230.so` to `/usr/lib64/`
   directory. <br>
         In my case it was a new file, so adding it to `/usr/lib64` should not cause any harm.<br>    
		b) Make a copy of the current existing file
   `/usr/lib/systemd/systemd-cryptsetup`
   

	> cd /usr/lib/systemd/<br>
	> cp ./systemd-cryptsetup ./systemd-cryptsetup.copy


   c) Replace `/usr/lib/systemd/systemd-cryptsetup` with our file from `.libs`
   subdirectory
    

	>cp (*our compilation dir*)/.libs/systemd-cryptsetup /usr/lib/systemd/

 
 - Create new initramfs:           
	>     dracut -f
	
     After this command check the `/boot` directory. There should be a file                 `initramfs(kernel version).img` with the current modification time.<br>    
     You can use `lsinitrd` utility to display the  content of the `initramfs` and check if our files are there.
     
 - Restart the system and check if your devices are unlocked.
     
    

	> ls /dev/mapper/*

    You can automatically mount unlocked devices using fstab.
     
 - Remember, that these modifications will remain unchanged only to
   the next `systemd` upgrade.<br>
       In this case you have to replace `/usr/lib/systemd/systemd-cryptsetup` with your modified version again.<br>
       Do you need to download and compile the systemd again? It's up to you.

    
