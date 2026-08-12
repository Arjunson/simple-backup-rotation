# Shell Scripts Backup Rotation

This script creates compressed backups of the `practice-scripts` directory and automatically keeps only the **7 most recent backups**.

## Directory Structure

The expected directory structure is:

```text
parent-directory/
└── practice-scripts/
    ├── backup-rotation.sh
    ├── script1.sh
    ├── script2.sh
    └── ...
```

The backup script determines its own location automatically, so it does not need a hard-coded source path.

By default, backups are stored in:

```text
$HOME/shell-backups/
```

For example:

```text
/home/username/shell-backups/
```

The backup directory will be created automatically if it does not already exist.

## Backup File Format

Each backup is created as a compressed `.tar.gz` archive:

```text
practice-scripts_YYYY-MM-DD_HH-MM-SS.tar.gz
```

Example:

```text
practice-scripts_2026-08-12_10-30-00.tar.gz
```

The archive contains the complete `practice-scripts` directory.

## How the Script Works

The script performs these steps:

1. Finds the directory where the backup script is located.
2. Creates the backup directory if necessary.
3. Creates a compressed `tar.gz` archive.
4. Stores the archive in `$HOME/shell-backups`.
5. Deletes backups older than the 7 most recent backups.
6. Lists the backups that remain.

### Backup Location

The default backup location is:

```bash
$HOME/shell-backups
```

You can change it by setting the `BACKUP_DIR` environment variable.

For example:

```bash
BACKUP_DIR=/tmp/my-backups ./backup-rotation.sh
```

Or:

```bash
export BACKUP_DIR=/path/to/backups
./backup-rotation.sh
```

## Make the Script Executable

After creating the script, make it executable:

```bash
chmod +x backup-rotation.sh
```

You can verify the permissions with:

```bash
ls -l backup-rotation.sh
```

You should see an `x` in the permissions, for example:

```text
-rwxr-xr-x
```

## Run a Backup Manually

From the `practice-scripts` directory:

```bash
./backup-rotation.sh
```

You should see output similar to:

```text
Backup created: /home/username/shell-backups/practice-scripts_2026-08-12_10-30-00.tar.gz
Size: 12K
Done with the backups:
-rw-r--r-- 1 username username 12K Aug 12 10:30 practice-scripts_2026-08-12_10-30-00.tar.gz
```

## Cron Job

Cron can be used to run the backup automatically.

First, find the absolute path to the script:

```bash
pwd
```

For example:

```text
/home/username/practice-scripts
```

The full script path would then be:

```text
/home/username/practice-scripts/backup-rotation.sh
```

### Edit Your Crontab

Run:

```bash
crontab -e
```

Add a cron entry.

For example, to run the backup **every day at 2:00 AM**:

```cron
0 2 * * * /home/username/practice-scripts/backup-rotation.sh >> /home/username/shell-backups/backup.log 2>&1
```

Replace `username` with your actual username.

### Common Cron Schedules

Run every day at 2:00 AM:

```cron
0 2 * * * /home/username/practice-scripts/backup-rotation.sh
```

Run every 6 hours:

```cron
0 */6 * * * /home/username/practice-scripts/backup-rotation.sh
```

Run every Sunday at 3:00 AM:

```cron
0 3 * * 0 /home/username/practice-scripts/backup-rotation.sh
```

Run every hour:

```cron
0 * * * * /home/username/practice-scripts/backup-rotation.sh
```

## Recommended Cron Configuration

For a simple daily backup, use:

```cron
0 2 * * * /home/username/practice-scripts/backup-rotation.sh >> /home/username/shell-backups/backup.log 2>&1
```

The following part:

```text
>> /home/username/shell-backups/backup.log 2>&1
```

saves the script's output and errors to a log file.

This is useful for checking whether the cron job is running successfully.

## Check Your Cron Jobs

To see your current cron jobs:

```bash
crontab -l
```

You should see your backup entry, for example:

```text
0 2 * * * /home/username/practice-scripts/backup-rotation.sh >> /home/username/shell-backups/backup.log 2>&1
```

## Check the Backups

List the backup directory:

```bash
ls -lh ~/shell-backups/
```

You should see something like:

```text
practice-scripts_2026-08-06_02-00-00.tar.gz
practice-scripts_2026-08-07_02-00-00.tar.gz
practice-scripts_2026-08-08_02-00-00.tar.gz
...
practice-scripts_2026-08-12_02-00-00.tar.gz
```

Only the **7 newest backups** should remain.

## Check the Cron Log

If you configured logging, check it with:

```bash
cat ~/shell-backups/backup.log
```

Or follow it live:

```bash
tail -f ~/shell-backups/backup.log
```

## Restore a Backup

To inspect an archive without extracting it:

```bash
tar -tzf ~/shell-backups/practice-scripts_YYYY-MM-DD_HH-MM-SS.tar.gz
```

To extract a backup:

```bash
tar -xzf ~/shell-backups/practice-scripts_YYYY-MM-DD_HH-MM-SS.tar.gz
```

To extract it into a specific directory:

```bash
tar -xzf ~/shell-backups/practice-scripts_YYYY-MM-DD_HH-MM-SS.tar.gz -C /path/to/restore
```

## Retention Policy

The script keeps:

```text
7 backups
```

When an 8th backup is created, the oldest backup is removed.

For example:

```text
Backup 1  ← oldest
Backup 2
Backup 3
Backup 4
Backup 5
Backup 6
Backup 7
Backup 8  ← newest
```

After cleanup:

```text
Backup 2
Backup 3
Backup 4
Backup 5
Backup 6
Backup 7
Backup 8
```

## Important Notes

### Use Absolute Paths in Cron

Cron has a different environment from your normal shell. Always use the full path to the script in your crontab.

Good:

```cron
0 2 * * * /home/username/practice-scripts/backup-rotation.sh
```

Avoid:

```cron
0 2 * * * ./backup-rotation.sh
```

### Backup Directory Must Be Writable

The user running the cron job must have permission to create files in:

```text
$HOME/shell-backups
```

The script creates this directory automatically if it does not exist.

### Cron Uses the User's Home Directory

Because the script uses:

```bash
BACKUP_DIR="${BACKUP_DIR:-$HOME/shell-backups}"
```

the default backup location is based on the home directory of the user running the script.

Therefore, install the cron job using:

```bash
crontab -e
```

for the same user who owns the scripts.

## Quick Setup

A complete initial setup can be done with:

```bash
cd /path/to/practice-scripts

chmod +x backup-rotation.sh

./backup-rotation.sh

crontab -e
```

Then add:

```cron
0 2 * * * /path/to/practice-scripts/backup-rotation.sh >> /path/to/shell-backups/backup.log 2>&1
```

Verify the cron entry:

```bash
crontab -l
```

And check the backups:

```bash
ls -lh ~/shell-backups/
```

## Summary

| Item                 | Default                                       |
| -------------------- | --------------------------------------------- |
| Source               | Directory containing `backup-rotation.sh`     |
| Backup directory     | `$HOME/shell-backups`                         |
| Backup format        | `.tar.gz`                                     |
| Backup naming        | `practice-scripts_YYYY-MM-DD_HH-MM-SS.tar.gz` |
| Retention            | 7 latest backups                              |
| Recommended schedule | Daily at 2:00 AM                              |
| Log file             | `$HOME/shell-backups/backup.log`              |
