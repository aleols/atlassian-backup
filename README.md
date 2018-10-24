# atlassian-backup

This tool backs up JIRA Server and Confluence Server databases according the
[best practices outlined by Atlassian](https://confluence.atlassian.com/adminjiraserver072/backing-up-data-828788079.html).

MySQL and PostgreSQL databases are supported.

## Config file
The backups are controlled by a config file.  The config file is written in the
Bash shell language.  The example config file `configs/atlassian-backup.conf`
contains documentation on the variables that can be set in the config file.  It
also contains example values.

You can skip JIRA backups by setting `JIRA_BACKUP_DIR` in the config file to the
empty string.  Similarly, you can skip Confluence backups by setting
`CONFLUENCE_BACKUP_DIR` to the empty string.

## Running atlassian-backup
The default config file location is `/etc/atlassian-backup.conf`, but ordinarily
you specify the config file on the command line:
```
atlassian-backup [config_file]
```

As it runs, `atlassian-backup` tells you what it is doing:
```
Backing up JIRA
  - Backing up JIRA attachments
/bin/tar: Removing leading `/' from member names
    Created /atlassian/backups/jira/jira-2017-11-07-19-34-attachments.tgz
  - Dumping JIRA database
    Created /atlassian/backups/jira/jira-2017-11-07-19-34-database-dump.sql.gz

Backing up Confluence
  - Backing up Confluence attachments
/bin/tar: Removing leading `/' from member names
    Created /atlassian/backups/confluence/confluence-2017-11-07-19-34-attachments.tgz
  - Dumping Confluence database
    Created /atlassian/backups/confluence/confluence-2017-11-07-19-34-database-dump.sql.
```

## Scheduling automatic backups with cron
On UNIX it is convenient to run `atlassian-backup` using cron.  The following backup
scheme backs up JIRA and Confluence each morning.

On the first of every month, a backup is created as usual.  Then, it is moved to the
`monthly/` subdirectory, where it is retained indefinitely.  Finally, to save space,
**all daily backups are removed from `$JIRA_BACKUP_DIR` and `$CONFLUENCE_BACKUP_DIR`**.
Presumably these are just the backups from the preceding month.

### Procedure

Log in as user `jira`.

The monthly backups will be retained in `$JIRA_BACKUP_DIR/monthly/` and
`$CONFLUENCE_BACKUP_DIR/monthly/`, where `$JIRA_BACKUP_DIR` and
`$CONFLUENCE_BACKUP_DIR` are defined in the config file.  Create these directories:
```
mkdir /atlassian/backups/jira/monthly/ /atlassian/backups/confluence/monthly/
```

Edit your cron table using the command:
```
crontab -e
```

Add the following to your cron table:
```
10 07 * * * /atlassian/atlassian-backup/bin/atlassian-backup /atlassian/atlassia
n-backup/configs/atlassian-backup.conf
# On the first of each month, after the backup runs,
# save the backups from the first and delete the rest.
10 10 1 * * cd /atlassian/backups/confluence && mv confluence-20??-??-01-*z monthly && r
m confluence-20??-??-??-*z
10 11 1 * * cd /atlassian/backups/jira && mv jira-20??-??-01-*z monthly && rm jira-20??-
??-??-*z
```

## History

Matt Kirby of Puppet Labs created
[jira-backup](https://github.com/puppetlabs/jira-backup) to back up JIRA alone.

John McGehee of Wave Computing enhanced `jira-backup` to back up Confluence as
well as JIRA.  To reflect the new functionality, the application was renamed
[atlassian-backup](https://github.com/jmcgeheeiv/atlassian-backup).
