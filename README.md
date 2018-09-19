
# ISLE Migration Guide (how to migrate to ISLE from an existing Islandora instance)

## About this guide
This guide is written for users of all levels of familiarity with Islandora and Linux system administration. Each step will include the necessary command(s) to find and retrieve the values required. 

We begin the process by [Taking inventory of your existing Islandora stack](#taking-inventory-of-your-existing-islandora-stack). In that section we find the paths (locations) of folders, files, usernames and passwords. There is an [Inventory Table](#inventory-table) that can be copied and used to record these data.  In the second section we copy the required files from your stack for use in ISLE. In the final section we prepare and launch your ISLE stack.

If you are uncomfortable do not hesitate to ask an ISLE Maintainer, ask fellow community members on our [Google Group](..), or your local IT administrator for assistance. ISLE Maintainers are dedicated to all institutions.

> **Important Note**: some of these folders and files may be owned by another user and/or group that you cannot access with your user (`id`).   You may be required to request `sudo` privileges on your existing Islandora server from an IT administrator, or request that an IT administrator complete these steps for you.  

> **Caution** `sudo` grants elevated privileges on a server and comes great responsibility.  You will have the ability to do most anything including _deleting files_. Work with utmost care! If unsure always ask your IT administrator to take a `snapshot` before you begin your work (and tell them when your work is complete so they can remove the snapshot (they require disk space to store).
---
## Taking inventory of your existing Islandora Stack
We present common locations and `terminal` commands to find the data required.  To test the presented common locations in terminal type `cd {LOCATION}`; you are successful if you can change into that directory and can record that location, otherwise use the command provided.

 - Locating your Fedora and Drupal (Islandora) data folders  
 	 1. Finding your Fedora data folder  
			 a. Common locations: `/usr/local/fedora` or `/usr/local/tomcat/fedora`  
			 b. Use find: `find / -type d -ipath '*fedora/data' -ls  2>/dev/null`  
	 2. Finding your Drupal data folder  
			 a. common locations: `/var/www/` (may be in a sub-folder) or   `/var/www/html`  
			 b. 

 - MySQL passwords for both Fedora and Drupal _OR_ your MySQL root password.
	 3. If you have your MySQL `root` password please skip to item 6.
	 4. Finding your Fedora MySQL username and password
		 a. 
		 b.
	 5. Finding your Drupal MySQL username and password
		 a.
		 b.
	 6.  Finding your Drupal and Fedora database names
		 a.
		 b. 
---
### Inventory Table
|#| Inventory Item | Setting, Password, or Location | Note |
|-|--|--|--|
|1| Fedora Data Location |  |
|2| Drupal Data Location |  | 
|3| MySQL `root` Password |  | If available skip other MySQL users/pass
|4| MySQL Fedora Username |  |
|4| MySQL Fedora Password |  |
|5| MySQL Drupal Username |  |
|5| MySQL Drupal Password |  |
|6| MySQL Fedora Database |  |
|6| MySQL Drupal Database |  |
---

## Prepare the files you are _required_ to have from your existing Islandora Stack.
0. Login to your existing Islandora Server. 
	1. Change directory to your `home directory` with `cd ~`. 
	2. If you will be copying data to a remote server please have SSH access to that server. 
1. SQL Dumps 
	 - SQL dump of your Fedora database
		- `mysqldump -u {FEDORA_USERNAME} -p {FEDORA_DATABASE_NAME} | gzip fedora.sql.gz`
	 - SQL dump of your Drupal database
		  - `mysqldump -u {DRUPAL_USERNAME} -p {DRUPAL_DATABASE_NAME} | gzip drupal.sql.gz`
	> **Note**  you may add your password directly to the commands below as: `-p{PASSWORD}` (no additional space).
	This is **not** recommended as your shell history (e.g., `bash_history`) will have those passwords stored. You may delete your shell history when you are complete (`rm ~\.bash_history`)
	**Note** If you have your MySQL root password replace {USERNAMES} with `root`
  2. Drupal (Islandora) webroot
		- Typical location: `/var/www/html` 
		- 
 3. Fedora Data	(these are large directories and copying takes time, please plan accordingly)
	- Fedora datastreamStore
		- < >
	- Fedora objectStore
		- < >
	- Fedora resourceIndex
		- < >

 - Specific Fedora data folders. These folders hold your archival objects and derivatives. 
 These folders are _large_.
	- resourceIndex
	- objectStore
	- datastreamStore

	>  **NOTE**: Before migration we recommend making copies of these folders to ensure your data is safe.
	> Advanced ways of copying these large files and folders are explored In section Copying Large Folders and Files (i.e., methods faster than `rsync`) .

<!--stackedit_data:
eyJoaXN0b3J5IjpbNjA5NDkwMzk1LC0xMTEyMDcwMTk1LDYxND
c0NTk5OSw4MzQyNDMzNDldfQ==
-->