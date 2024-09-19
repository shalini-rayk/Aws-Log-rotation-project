# Managing AWS Resources and to perform Log Rotation

## Introduction

This project is designed to manage AWS resources and perform log rotation to delete or compress old logs from AWS resource log files, with all tasks automated using cron jobs.

## Initial Setup/Requirement

- AWS account
- Terminal which supports Bash

## Steps to Implement

1. Launch EC2 (OS - Ubuntu)
2. Configure AWS CLI
3. Create Script-file
4. Create Log-file
5. Create Logrotation-file
6. Add cron jobs for Script-file and Logrotation-file
## Process

### Configure AWS CLI

1. **Install AWS CLI**:
   - To configure AWS CLI, search for "AWS CLI install" to find download links for Mac, Linux, and Windows. Since we are working on a Linux server (connected to AWS EC2), you can install the updated version directly from there.

2. **Verify Installation**:
   - To test the installation, use the command:
     ```bash
     aws --version
     ```
     If installed properly, this command will display the version of your AWS CLI.

3. **Obtain Access Keys**:
   - You need access keys from your AWS account to configure AWS CLI in the terminal.
   - In the upper right corner of your AWS console, choose your account name or number and then choose **Security Credentials**.
   - In the **Access keys** section, choose **Create access key**. If this option is not available, you already have the maximum number of access keys and must delete one of the existing keys before creating a new key.

4. **Configure AWS CLI**:
   - Type the following command in the terminal:
     ```bash
     sudo aws configure
     ```
   - You will be prompted to enter:
     - **AWS Access Key ID**
     - **AWS Secret Access Key**
     - **Default region name** (check this in the upper right corner of your AWS account, e.g., `us-east-1`)
     - **Default output format** (e.g., `json`)
## Create Script-file

Change the permissions of the script file to `chmod 700`. If group or other users are involved, use `chmod 750` or `chmod 744`.

```
#!/bin/bash
#################################
# Author : Shalini
# Date   : 21st - Aug
# Version : v1
# This script will report the AWS resource usage
#################################
# Define the log file path

LOG_FILE="/home/ubuntu/project/log-file"
{
    echo "----- $(date '+%Y-%m-%d %H:%M:%S') -----"

  # Fetching EC2 Instances
    echo "The EC2 instances are:"
    aws ec2 describe-instances | jq '.Reservations[].Instances[] | {InstanceId, InstanceType}'

    # Fetching S3 Buckets
    echo "The S3 buckets are:"
    aws s3api list-buckets | jq '.Buckets[] | {Name}'

    # Fetching Lambda Functions
    echo "The Lambda functions are:"
    aws lambda list-functions | jq '.Functions[] | {FunctionName}'

    # Fetching IAM Users
    echo "The IAM users are:"
    aws iam list-users | jq '.Users[] | {UserName}'
    echo "----------------------------------------"
} >> "$LOG_FILE"
```
## Syntax Explanation

- `LOG_FILE="/home/ubuntu/project/log-file"`  
  Here, we are storing the log file path into a variable called `LOG_FILE`. The output of the entire script is then redirected to `"$LOG_FILE"` using `>>`, which means that whenever the script runs, the logs or output will be appended to the specified log file.

## AWS Commands Breakdown

1. **`aws ec2 describe-instances`**  
   This AWS CLI command retrieves information about one or more EC2 instances in your account. It returns a JSON response containing details such as instance IDs, instance types, availability zones, and more.

2. **`|`**  
   The pipe (`|`) is used to pass the output of the `aws ec2 describe-instances` command to the next command. In this case, it's passing the JSON response to `jq`.

3. **`jq`**  
   `jq` is a command-line JSON processor that allows you to filter and manipulate JSON data using a simple query language.

4. **`.Reservations[].Instances[]`**  
   This part of the `jq` query navigates through the JSON structure returned by the AWS CLI. The `[]` denotes an array.

5. **`| {InstanceId, InstanceType}`**  
   This part of the `jq` query extracts specific fields for each instance, in this case, `InstanceId` and `InstanceType`.
   
### Create Log File

Create log-file to store logs from script file and provide permissions as `640` (As there is no need of execution)

### Create Log Rotation File

The user and group should be `root:root` (`sudo chown root:root log-rotation file`) and permissions should be `644` (`sudo chmod 644`)

Create Log-rotation file in system wide log rotation files storage i.e, `/etc/logrotate.d/`

- **/var/log** - where system wide log files are stored
- **/usr/sbin/logrotate** - logrotate tool
- **/etc/logrotate.conf** - Main configuration file
- **/etc/logrotate.d/** - where log rotation files are stored for all the files present in `/var/log`
### Syntax in Log-Rotation File

```
/home/ubuntu/Aws-project/log-file
 {
   size 0
    rotate 1
    compress
    delaycompress
    missingok
    notifempty
    create 640 ubuntu ubuntu
}
```
### Explanation

- **/home/ubuntu/project/log-file**: The path to the log file to be rotated. (Using the above syntax, it will rotate and compress the log file.)
- **size 0**: Rotate the log file regardless of its size (rotates every time logrotate runs).
- **rotate 1**: Keep only 1 previous version of the log file.
- **compress**: Compress the rotated log file to save space.
- **delaycompress**: Delay compression of the most recent rotated log until the next rotation.
- **missingok**: If the log file is missing, skip rotation without raising an error.
- **notifempty**: Skip rotation if the log file is empty.
- **create 640 ubuntu ubuntu**: After rotation, create a new log file with `640` permissions, owned by the `ubuntu` user and group.
  
### Add cron jobs for script-file and logrotation-file  —

A cron job is a scheduled task in Unix-like operating systems that runs automatically at specified times or intervals. It is helpful in automating tasks. So there is no need to manually run script-file and logrotation-file.

To add cronjob  :
“sudo crontab -e”  
Enter 1
Enter the cronjob:

### For script-file

“ */15 * * * * /home/ubuntu/project/Script-file “

*/15 (Minute Field): Executes the task every 15 minutes.
* (Hour Field): Runs the task every hour of the day.
* (Day of the Month Field): Executes the task every day of the month.
* (Month Field): Runs the task every month of the year.
* (Day of the Week Field): Executes the task every day of the week.

/home/ubuntu/project/Script-file (Command): Specifies the path to the script or command to be executed.

### For log-rotation file

“ 0 * * * * /usr/sbin/logrotate /etc/logrotate.d/logrotation-file >> /home/ubuntu/Aws-project/debug-file 2>&1 “
                             OR
“ 0 * * * * /usr/sbin/logrotate /etc/logrotate.d/logrotation-file “

0 (Minute Field): The job runs at the start of the hour (minute 0).

* (Hour Field): The job runs every hour of the day.
* (Day of the Month Field): The job runs every day of the month.
* (Month Field): The job runs every month of the year.
* (Day of the Week Field): The job runs every day of the week.

/usr/sbin/logrotate: Path to the logrotate executable.

/etc/logrotate.d/logrotation-file: Path to the logrotate configuration file containing rotation rules.

>> /home/ubuntu/Aws-project/debug-file: Appends the standard output of the logrotate command to the debug file. (This step is not necessary but if you want to check if there occurs any error in cronjobs then that error will be printed in debug file)

2>&1: Redirects both standard output and standard error to the debug file.
# Testing

## Testing Script File

In order to manually test if script file is working or not do `bash script-file`. If this is successful there should be a log in `log-file`. Open the content of the `log-file` using the `cat log-file` command and if there is a log stating how many ec2 instances and s3 buckets etc are there, then the script-file is running fine.

## Testing Log-Rotation File

```
sudo /usr/sbin/logrotate -f /etc/logrotate.d/logrotation-file
```
# Testing

## Testing Script File

In order to Manually test if script file is working or not do `bash script-file`. If this is successful there should be a log in `log-file`. Open the content of the `log-file` using the `cat log-file` command and if there is a log stating how many ec2 instances and s3 buckets etc are there, then the script-file is running fine.

## Testing Log-Rotation File

`-f` means it forcefully logrotate the `logrotation-file`. To check if this is successful, do the `cat log-file` command and if the file is empty which means it cleared the previous log which we got by manually running `script-file`.

## Testing Cronjobs

If for every 15 minutes we are seeing a log in `log-file` then the cronjob for `script-file` is success

Every hour if you are seeing the logs are being rotated which means 
- Old logs being removed from `log-file`
- Automatically a `log-file.1` got created to store these old logs
- The `log-file` becomes empty to store another cycle of new log files
- In the next cycle the `log-file.1` files will be compressed and sent to an automatically created `log-file.gz`
- Once again the `log-file` becomes empty and sends logs to `log-file.1` file. This cycle continues every hour

## Important Notes

We need to make sure the AWS CLI is configured by using `sudo aws configure`
The permissions to all the files are assigned as mentioned or according to the needs and avoid providing unnecessary open permissions
