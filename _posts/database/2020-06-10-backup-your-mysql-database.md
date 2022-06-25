---
layout: post
title:  "Backup your MySQL database"
slug: 'bacup-your-mysql-database'
date:   2020-06-10 09:30:00 +0300
category: database
tags: [mysql, backup, cronjob]
icon: table
---

I recommend that you set up a backup for any database that you have. This guarantees your production data will be safe and you don't loose data in case of a desaster. Below, I've sketched out easy process of setting up a backup with regular data dumps.

## Create the backup

I advice you to always prefix your backups in some meaningfull way, for example you could sufix it with `_backup_` followed by the current date and time.

```
$ mysql -u root -p
mysql> CREATE DATABASE db_name_backup_06_10_2020;
mysql> exit
```

## Load data from old database into new database

```
$ mysqldump -u root -password=secure_password db_name | mysql -u root -password=my_password db_name_backup_2020_10_06
```

## Automate the backup

If you want to get it to the next step you could also create a shell script to do the job for you. Shell scripts allow you to easily automate tasks in Linux.

### Create Shell Script

Open terminal and run the following command to create an empty shell script file.

```
$ vim /home/db_backup.sh
```

### Add shell script to backup MySQL

Add the following code to your shell script file. Replace dbname, dbuser and dbpass with your database name, username and password respectively.

```
#!/bin/sh

echo "Backup starting..."
sufix="backup_$(date +'%m-%d-%Y')"
db_backup="db_name_${sufix}.sql"
sudo mysqldump  -u db_user -p db_pass --no-tablespaces db_name  >/home/${db_backup}
echo "Backup complete!"
```

First define the execution environment for shell script and echo a message that indicates the backup has started. 
Then create a sufix variable to store current date `backup_06_10_2020`.
Use this variable to define the backup file name (.sql) for the database backup. 
Finally, run the MySQL command to backup database, store it in `/home` folder and echo the message that database backup is complete.

If you want to compress the backup data, you can modify the mysqldump command and basically pipe the output to `gzip` command to create the gzip file. Also you have to modify the final file name to be a .gz file instead of a .sql file.

```
sufix="_backup_$(date +'%m-%d-%Y')"
db_backup="db_name_${sufix}.gz"
sudo mysqldump  -u db_user -p db_pass --no-tablespaces db_name | gzip -c >/home/${db_backup}
```

### Make the script executable

To make your shell script executable run the following command.

```
$ sudo chmod +x /home/db_backup.sh
```

### Test the script

Execute the shell script using the following command.

```
$ sudo /home/db_backup.sh
Backup starting...
Backup complete!
```

### Schedule it as a cron comand

Now we can add the above command as a cronjob to regularly take database backup.

Open crontab file with the following command.

```
$ sudo crontab -e
```

Add the following line to it. It will run your db_backup.sh script everyday at 10.a.m. You can change it as per your requirement.

```
0 2 * * * sudo /home/db_backup.sh >/dev/null 2>&1
```

Save and close the file by pressing `esc + :wq`. You can check if you correctly saved the file with the command.

```
$ sudo crontab -l
```

In the above code “0 2 * * *” indicates that the cron job should be run every day at 2:00AM. 
- `sudo /home/db_backup.sh` is your command. 
- `> /dev/null 2>&1` indicates that all standard output and error messages must be sent to `/dev/null`, meaning that should not be displayed.

To avoid `file not found` or `permission denied` errors make sure you write the full path to your shell script and that you use sudo to schedule the cronjob.

That's all, you have learnt how to create a shell script to backup MySQL database. Also you automated the process by creating a cronjob to run the shell script every day. 
If you run the above shell script everyday as a cronjob, it will create different backup files, one for each day of the month.