
**Detailed Runbook: MySQL Hourly Backup to Amazon S3**

**Configure hourly backups of the MySQL database, upload them to Amazon S3, keep only the latest three local backups, and log every execution.**

Step 1 - Install AWS CLI
Install and verify AWS CLI.
wget https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -O awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
aws --version
rm -rf awscliv2.zip aws/


Step 2 - Configure AWS
Configure credentials and verify S3 access.

aws configure
aws s3 ls
aws s3 ls s3://<bucket_name>/


**Step 3 - Create directories**
Create standard locations.
sudo mkdir -p /opt/scripts
sudo mkdir -p /opt/backups/mysql
sudo mkdir -p /opt/log
sudo chown -R ubuntu:ubuntu /opt/scripts /opt/backups /opt/log


**Step 4 - Store MySQL credentials**

mysql_config_editor set --login-path=backup --host=127.0.0.1 --user=<user_name> --password
mysql_config_editor print --all

**Step 5 - Backup script**
Create /opt/scripts/mysql_backup.sh with mysqldump, gzip, S3 upload, and local retention.
mysqldump \
 --login-path=backup \
 --single-transaction \
 --skip-lock-tables \
 --quick \
 --no-tablespaces \
 "$DATABASE" | gzip > "${BACKUP_DIR}/${BACKUP_FILE}"
aws s3 cp "${BACKUP_DIR}/${BACKUP_FILE}" "s3://dev-v3-log.etvbharat/mysql-backups/"
ls -1t ${BACKUP_DIR}/*.sql.gz | tail -n +4 | xargs -r rm -f


**Step 6 - Permissions**

Made the script executable and granted LOCK TABLES.

chmod +x /opt/scripts/mysql_backup.sh

GRANT LOCK TABLES ON <db_name>.* TO '<user_name>'@'%';

FLUSH PRIVILEGES;

**Step 7 - Validation**
Validated the backup.
/opt/scripts/mysql_backup.sh
ls -lrth /opt/backups/mysql/
gzip -t /opt/backups/mysql/<backup>.sql.gz
zcat /opt/backups/mysql/<backup>.sql.gz | head -20

Step 8 - Logging
Configured log file.
sudo touch /opt/log/mysql_backup.log
sudo chown ubuntu:ubuntu /opt/log
sudo chown ubuntu:ubuntu /opt/log/mysql_backup.log
Step 9 - Cron
Schedule hourly execution.
crontab -e
0 * * * * /opt/scripts/mysql_backup.sh >> /opt/log/mysql_backup.log 2>&1
Issues Resolved
1. Incorrect mysql_config_editor host (%). 2. Authentication error 1045. 3. LOCK TABLES privilege.
4. PROCESS privilege warning resolved using --no-tablespaces. 5. Log permission issue resolved
with chown.
