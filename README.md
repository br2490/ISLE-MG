

# ISLE Migration Guide (how to migrate to ISLE from an existing Islandora instance)

## About this guide
This guide is written for users of all levels of familiarity with Islandora and Linux system administration. Each step will include the necessary command(s) to find and retrieve the values required. 

We start by [Taking inventory of your existing Islandora stack](#taking-inventory-of-your-existing-islandora-stack). We'll find the paths (locations) of folders, files, usernames and passwords necessary to `backup and copy` your Islandora instance. Please see the [Inventory Table](#inventory-table) which can be helpful to record these data.  

In the second and third section we perform the `backups` in preperation to `copy` the required data from your existing Islandora instance to the home of your ISLE instance. We want to work on copies of your data before removing or stopping your existing Islandora instance. Note that there are default files that come with ISLE that may be desirable over your existing Islandora stack; we leave that distinction up to you (primarily: our Solr schema and FedoraGSearch transforms).

In the final section we configure ISLE's `environment`, import and point to your data, and launch your ISLE stack.

ETA, excluding copying files to new servers/locations: about two hours. The longest section, `copying` can take some time if your Islandora instance is big!

If you are uncomfortable do not hesitate to ask an ISLE Maintainer, ask fellow community members on our [Google Group](https://groups.google.com/forum/#!forum/islandora-isle), or your local IT administrator for assistance. ISLE Maintainers are dedicated to all institutions.

> **Important Note**: some of these folders and files may be owned by another user and/or group that you cannot access with your user (`id`). You may be required to request `sudo` privileges on your existing Islandora server from an IT administrator, or request that an IT administrator complete these steps for you. 

> **Caution** `sudo` grants elevated privileges on a server and comes great responsibility.  You will have the ability to do most anything including _deleting files_. Work with utmost care! If unsure always ask your IT administrator to take a `snapshot` before you begin your work and tell them when your work is complete so they can remove the snapshot (they require disk space to store).

Okay, now to the fun part!

---

## Taking inventory of your existing Islandora Stack
This section presents common locations of files, and `terminal` commands to find the data required.
To test the common locations, while logged into your server, type `cd {LOCATION} && ls`. If you can change into that directory _record that location_, otherwise use the `command` provided and record the output.

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

 4. Finding your FedoraGSearch data (i.e. transforms) folder
    - `find / -type d -ipath '*web-inf/classes/fgsconfigfinal' -ls 2>/dev/null`

### MySQL data for both Fedora and Drupal _OR_ your MySQL root password.

 5. Note well that if you have your MySQL `root` password you only need find the database values below.

 6. Finding your Fedora MySQL username, password, and database
    - `grep --include=fedora.fcfg -rnw -e 'name="dbUsername"' -e 'name="dbPassword"' -e 'name="jdbcURL"' / 2>/dev/null`
    - This command _will_ print multiple lines. The first three lines are important but please save the rest (just in case).

    Example output:
     > param name="dbUsername" value="**fedoraDB**"  
     > param name="jdbcURL" value="jdbc:mysql://localhost/**fedora3**?useUnicode=true&amp;amp;characterEncoding=UTF-8&amp;amp;autoReconnect=true"  
     > param name="dbPassword" value="**zMgBM6hGwjCeEuPD**"

    - Username: Copy the value from `dbUsername value=`
    - Password: Copy the value from `dbPassword value=`
    - Database: Copy from the value `jdbcURL value=` the database name which is directly between the "/" and the only "?"

 7. Finding your Drupal MySQL username, password, and database
    - `grep --include=filter-drupal.xml -rnw -e 'dbname.*user.*password.*"' / 2>/dev/null`
    
    Example output:
      > connection server="localhost" port="3306" dbname="**islandora**" user="**drupalIslandora**" password="**Kjs8n5zQXfPNhZ9k**"

    - Username: copy the value from `user=`
    - Password: copy the value from `password=`
    - Database: copy the value from `dbname=`
    
Phew!

---
### Inventory Table
|#| Inventory Item | Setting, Password, or Location | Note |
|-|--|--|--|
|1| Fedora Data Location |  |
|2| Drupal Data Location |  | 
|3| Solr Data Location |  | 
|4| FedoraGSearch Location |  |
|5| MySQL `root` Password |  | If available skip other MySQL users/pass
|6| MySQL Fedora Username |  |
|6| MySQL Fedora Password |  |
|6| MySQL Fedora Database |  |
|7| MySQL Drupal Username |  |
|7| MySQL Drupal Password |  |
|7| MySQL Drupal Database |  |
---

## Backing up the _required_ files from your existing Islandora Stack.
<!-- All commands that are run here are explained in section [Explanation of Commands](#explanation-of-commands) -->

 0. Login to your existing Islandora Server. 
    - Change directory to your home directory with `cd ~`. 
    - We will be create backups in your home directory to find them easily; 
    - Create a new directory in your home folder: `mkdir isledata && cd isledata` before starting.
      - Your backups will be stored in `~/isledata` (`cd ~/isledata` to return to this location)
    - If you will be copying data to a remote server please have SSH access to that server.
      - Test your connection to the remote server before starting: `ssh {USER}@{SERVER}` 

 1. Generate SQL Dumps 
    > **Note**  you may add your password directly to the commands below as: `-p{PASSWORD}` (no additional space).
    This is **not** recommended as your shell history (e.g., `bash_history`) will have those passwords stored. You may delete your shell history when you are complete (`rm ~\.bash_history`)

    **Note** If you have your MySQL root password replace {USERNAMES} with `root`
     - Make to be in our backup location `cd ~/isledata`
     - SQL dump of your Fedora database
        - `mysqldump -u {FEDORA_USERNAME} -p {FEDORA_DATABASE_NAME} | gzip > fedora.sql.gz`
     - SQL dump of your Drupal database
        - `mysqldump -u {DRUPAL_USERNAME} -p {DRUPAL_DATABASE_NAME} | gzip > drupal.sql.gz`

 2. Drupal (Islandora) webroot
      - In our backup location `cd ~/isledata`
      - `tar -zcf drupal-web.tar.gz -C {DRUPAL_DATA_LOCATION} .` (don't forget the final `.`)
        - for example: `tar -zcf drupal-web.tar.gz -C /var/www/html .`
      - Permission error? Prepend the command with `sudo` (e.g., `sudo tar -zcf drupal-web.tar.gz -C /var/www/html .`)

 3. Solr Data
      - In our backup location `cd ~/isledata`
      - `tar -zcf solr.tar.gz -C {SOLR_DATA_LOCATION} .`
        - for example: `tar -zcf solr-data.tar.gz -C /usr/local/solr .`
      - Permission error? Prepend the command with `sudo` (e.g., `sudo tar -zcf solr-data.tar.gz -C /usr/local/solr .`)

 4. Fedora Generic Search (FGS) Transforms (_optional_)
      - In our backup location `cd ~/isledata`
      - `tar -zcf fgs.tar.gz -C {FEDORAGSEARCH_LOCATION} .`
        - for example: `tar -zcf fgs.tar.gz -C /tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal .`
      - Permission error? Prepend the command with `sudo` (e.g., `sudo tar -zcf fgs.tar.gz -C /tomcat/webapps/fedoragsearch/WEB-INF/classes/fgsconfigFinal .`)

 5. Fedora Data  
    > These are large directories and copying them _will_ take several hours or even days. Please plan accordingly and prepare to leave these processes running unattended.

    The following folders are all located in the recorded {FEDORA_DATA_LOCATION}
    - These folders will be copied _directly_ to the remote server or to a new directory due to their size:
      - Fedora datastreamStore
      - Fedora objectStore
      - Fedora resourceIndex

Done! Coffee break.

## Copying Data to a new location or server
> Using `screen` we will be able to copy these files and then leave the session running on the remote server. Test running `screen` - if the application is installed you will be in a terminal multiplier; hit `CTRL+A` follwed by `d` to detach from it. Type `screen` to reconnect. 
- If screen is not installed, install it now: `sudo apt-get install screen` or `sudo yum install screen`.

> This section is STUBBED only. 
Run `screen`, in screen launch commands {1-4}. Launch command then `CTRL+A` folled by `c` to create a new terminal to run next command; repeat until all commands are running. Detach from `screen` (CTRL+A followed by d) to leave the processes running. 

 1. Copy our backup directory from `~/isledata` to a new location or server
    - rsync `~/isledata`
 2. Copy Fedora's datastreamStore folder
   - rsync Fedora datastreamStore
 3. Copy Fedora's objectStore folder
   - rsync Fedora objectStore
 4. Copy Fedora's resourceIndex
   - rsync Fedora resourceIndex

## Launching ISLE with your data


<!-- ## Copying Large Folders and File
  > This section is intended for advanced users.
  - Please read http://moo.nac.uci.edu/~hjm/HOWTO_move_data.html#_executive_summary

  - If your server is local and will not be going over WAN try using `nc` combined with `tar`.
    - `nc` is short for netcat and is UDP transfer, and can achieve near line-speed.
    - http://moo.nac.uci.edu/~hjm/HOWTO_move_data.html#tarnetcat
    - You may be tempted to use this over WAN but your data will not be encrypted.

  - If your server is _remote_ use try Globus connect -->

<!-- ## Explanation of Commands
Note: this section is a work in progress.

An explanation of the commands that you have been asked to run.

  `mysqldump`
    `mysqldump` creates a backup (i.e. 'a dump') of a SQL database
     - we passed these parameters: -u to specify a username, -p to prompt for password, and finally the name of the database to dump. 
     - the pipe (`|`) routes data: 
     - in this instance we piped from mysqldump to `gzip` to create a compressed files. Since your SQL database dump is a large text file compression saves disk space (and cuts transmission time to another location). 
 -->
