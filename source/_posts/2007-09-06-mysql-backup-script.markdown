---
layout: post
title: "Mysql Backup Script"
date: 2007-09-06 15:04
comments: false
categories: [Infrastructure]
---

Need a way to backup all your mysql databases in seperate files without eating up your hard drive, then this is the script you need.

Its a bash shell script that will export all your databases with mysqldump into seperate files named like 'database-date.sql'.  It will also delete your old backups so you don't fill up your hard drive.

<!-- more -->

{% codeblock lang:bash %}
#!/bin/bash

DBHOST='localhost'
DBUSER='root'
DBPASSWD='password'
LOCALDIR=/path/to/your/backup/directory/

# Using the date function's --date="last week" to delete backups that are 7 days old 
OLDSUFFIX=`date --date="last week" +%Y%m%d`
SUFFIX=`date +%Y%m%d`
DBS=`mysql -u$DBUSER -p$DBPASSWD -h$DBHOST -e"show databases"`
su_cmd=""
for DATABASE in $DBS; do
  if [ $DATABASE != "Database" ]; then
    OLDFILENAME=$OLDSUFFIX-$DATABASE.sql
    FILENAME=$SUFFIX-$DATABASE.sql
    su_cmd="${su_cmd} rm -f $LOCALDIR$OLDFILENAME;"
    mysqldump -u$DBUSER -p$DBPASSWD -h$DBHOST $DATABASE > $LOCALDIR$FILENAME
  fi
done
su -c "${su_cmd}"
echo "done";
{% endcodeblock %}