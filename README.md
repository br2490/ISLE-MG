

# ISLE Migration Guide (how to migrate to ISLE from an existing Islandora instance)

## About this guide
This guide is written for users of all levels of familiarity with Islandora and Linux system administration. Each step will include the necessary command(s) to find and retrieve the values required. 

We begin the process by [Taking inventory of your existing Islandora stack](#taking-inventory-of-your-existing-islandora-stack). In that section we find the paths (locations) of folders, files, usernames and passwords. There is an [Inventory Table](#inventory-table) that can be copied and used to record these data.  In the second section we copy the required files from your stack for use in ISLE. In the final section we prepare and launch your ISLE stack.

If you are uncomfortable do not hesitate to ask an ISLE Maintainer, ask fellow community members on our [Google Group](https://groups.google.com/forum/#!forum/islandora-isle), or your local IT administrator for assistance. ISLE Maintainers are dedicated to all institutions.

> **Important Note**: some of these folders and files may be owned by another user and/or group that you cannot access with your user (`id`).   You may be required to request `sudo` privileges on your existing Islandora server from an IT administrator, or request that an IT administrator complete these steps for you.  

> **Caution** `sudo` grants elevated privileges on a server and comes great responsibility.  You will have the ability to do most anything including _deleting files_. Work with utmost care! If unsure always ask your IT administrator to take a `snapshot` before you begin your work and tell them when your work is complete so they can remove the snapshot (they require disk space to store).

---

## Taking inventory of your existing Islandora Stack
We present common locations and `terminal` commands to find the data required.  To test the presented common locations in terminal type `cd {LOCATION}`; you are successful if you can change into that directory and can record that location, otherwise use the command provided.

### Locating your Fedora, Drupal (Islandora), and Solr data folders

 1. Finding your Fedora data folder
    - Common locations: `/usr/local/fedora/data` or `/usr/local/tomcat/fedora/data`  
    - Use find: `find / -type d -ipath '*fedora/data' -ls  2>/dev/null`
             
 2. Finding your Drupal data folder  
    - Common location: `/var/www/` (likely in a sub-folder; e.g., html, islandora, etc.)
    - `grep --include=index.php -rl -e 'Drupal' / 2>/dev/null`

 3. Finding your Solr data folder  
    - Common location: `/usr/local/solr`, `/usr/local/tomcat/solr`, or `/usr/local/fedora/solr`
    - `find / -type d -ipath '*solr/*/data' -ls  2>/dev/null`

### MySQL data for both Fedora and Drupal _OR_ your MySQL root password.

 4. If you have your MySQL `root` password find the database values below, you do not need any other username or password for this section.

 5. Finding your Fedora MySQL username, password, and database
    - `grep --include=fedora.fcfg -rnw -e 'name="dbUsername"' -e 'name="dbPassword"' -e 'name="jdbcURL"' / 2>/dev/null`
    - This command _will_ print multiple lines. The first three lines are important but please save the rest (just in case).

    Example output:
     > param name="dbUsername" value="**fedoraDB**"  
     > param name="jdbcURL" value="jdbc:mysql://localhost/**fedora3**?useUnicode=true&amp;amp;characterEncoding=UTF-8&amp;amp;autoReconnect=true"  
     > param name="dbPassword" value="**zMgBM6hGwjCeEuPD**"

    - Username: Copy the value from `dbUsername value=`
    - Password: Copy the value from `dbPassword value=`
    - Database: Copy from the value `jdbcURL value=` the database name which is directly between the "/" and the only "?"

 6. Finding your Drupal MySQL username, password, and database
    - `grep --include=filter-drupal.xml -rnw -e 'dbname.*user.*password.*"' / 2>/dev/null`
    
    Example output:
      > connection server="localhost" port="3306" dbname="**islandora**" user="**drupalIslandora**" password="**Kjs8n5zQXfPNhZ9k**"

    - Username: copy the value from `user=`
    - Password: copy the value from `password=`
    - Database: copy the value from `dbname=`
    


---
### Inventory Table
|#| Inventory Item | Setting, Password, or Location | Note |
|-|--|--|--|
|1| Fedora Data Location |  |
|2| Drupal Data Location |  | 
|3| Solr Data Location |  | 
|4| MySQL `root` Password |  | If available skip other MySQL users/pass
|5| MySQL Fedora Username |  |
|5| MySQL Fedora Password |  |
|5| MySQL Fedora Database |  |
|6| MySQL Drupal Username |  |
|6| MySQL Drupal Password |  |
|6| MySQL Drupal Database |  |
---

## Prepare the files you are _required_ to have from your existing Islandora Stack.
 0. Login to your existing Islandora Server. 
    - Change directory to your `home directory` with `cd ~`. 
    - We will be creating backups in your home directory to find them easily; 
      - Consider creating a new directory in your home folder: `mkdir isledata && cd isledata` before starting.
      - In this example your backups are now stored in `~/isledata` (`cd ~/isledata` to return to this location)
    - If you will be copying data to a remote server please have SSH access to that server.
      - Test your connection to the remote server before starting: `ssh {USER}@{SERVER}` 

 1. Generate SQL Dumps 
    > **Note**  you may add your password directly to the commands below as: `-p{PASSWORD}` (no additional space).
    This is **not** recommended as your shell history (e.g., `bash_history`) will have those passwords stored. You may delete your shell history when you are complete (`rm ~\.bash_history`)

    **Note** If you have your MySQL root password replace {USERNAMES} with `root`

     - SQL dump of your Fedora database
        - `mysqldump -u {FEDORA_USERNAME} -p {FEDORA_DATABASE_NAME} | gzip > fedora.sql.gz`
     - SQL dump of your Drupal database
        - `mysqldump -u {DRUPAL_USERNAME} -p {DRUPAL_DATABASE_NAME} | gzip > drupal.sql.gz`

 2. Drupal (Islandora) webroot
      - `tar -zcf drupal-web.tar.gz {DRUPAL_DATA_LOCATION}`

 3. Fedora Data 
    > These are large directories and copying them takes hours or days. Please plan accordingly and prepare to leave these processes running for some time.
    > Advanced ways of copying these large files and folders are explored In section Copying Large Folders and Files (i.e., methods faster than `rsync`) .
    - Fedora datastreamStore
        - < >
    - Fedora objectStore
        - < >
    - Fedora resourceIndex
        - < >


## Copying Large Folders and Files (advanced users)
Section is in development.
	- If your server is local and will not be going over WAN try using `nc`
	- 




<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA2MDE5NjczNywxMjIwMDA0NTU0LC0yMD
M3ODE2NTczLC0xMjgyNzgzOTU5LC0xNTY0MzkxMTU5LDYwOTQ5
MDM5NSwtMTExMjA3MDE5NSw2MTQ3NDU5OTksODM0MjQzMzQ5XX
0=
-->