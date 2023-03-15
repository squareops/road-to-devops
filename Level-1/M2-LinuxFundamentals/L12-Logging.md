# Logging 
## What is Logfile ?

A log file is an event that took place at a certain time and might have metadata that contextualizes it.

Log files are a historical record of everything and anything that happens within a system, including events such as transactions, errors and intrusions. That data can be transmitted in different ways and can be in both structured, semi-structured and unstructured format

The basic anatomy of a log file includes:

- The timestamp – the exact time at which the event logged occurred
- User information
- Event information – what was the action taken

However, depending on the type of log source, the file will also contain a wealth of relevant data. For example, server logs will also include the referred webpage, http status code, bytes served, user agents, and more.

## Where do Log Files Come From?
Just about everything produces some version of a log, including:

- Apps
- Containers
- Databases
- Firewalls
- Endpoints
- IoT devices
- Networks
- Servers
- Web Services
- Log File Sources

The list goes on, but the point is, almost all infrastructure that you interact with on a daily basis produces a log file.

## Who Uses Log Files?
Log files can provide almost every role at an organization with valuable insights. Below are some of the most common use cases by job function:

### ITOps
- identify infrastructure balance
- Manage workloads
- Maintain Uptime/Outages
- Ensure business continuity
- Reduce cost and risk

### DevOps
- Managing CI/CD
- Maintain application uptime
- Detect critical application errors
- Identify areas to optimize application performance

### DevSecOps
- Drive a shared ownership on application development and security
S- aving time/money and reputational risks by finding potential issues before deployment

### SecOps/Security 
- Uncover clues around the ‘who, when, where’ of an attack
- Identify suspicious activity
- See spikes in blocked/allowed traffic
- Implementing the methodologies such as the OODA Loop

### IT Analysts 
- Compliance management and Reporting
- OpEx and CapEx
- Business Insights

## Types of Logs
Nearly every component in a network generates a different type of data and each component collects that data in its own log. Because of that, many types of logs exist, including:

- Event Log: a high-level log that records information about network traffic and usage, such as login attempts, failed password attempts, and application events.
- Server Log: a text document containing a record of activities related to a specific server in a specific period of time.
- System Log (syslog): a record of operating system events. It includes startup messages, system changes, unexpected shutdowns, errors and warnings, and other important processes. Windows, Linux, and macOS all generate syslogs.
- Authorization Logs and Access Logs: include a list of people or bots accessing certain applications or files.
- Change Logs: include a chronological list of changes made to an application or file.
- Availability Logs: track system performance, uptime, and availability.
- Resource Logs: provide information about connectivity issues and capacity limits.
- Threat Logs: contain information about system, file, or application traffic that matches a predefined security profile within a firewall.

## The Importance of Log Management
While there are seemingly infinite insights to be gained from log files, there are a few core challenges that prevent organizations from unlocking the value offered in log data.

### Challenge #1: Volume
With the rise of the cloud, hybrid networks, and digital transformation, the volume of data collected by logs has ballooned by orders of magnitude. If almost everything produces a log, how can an organization manage the sheer volume of data to quickly realize the full value offered by log files?

### Challenge #2: Standardization
Unfortunately, not all log files follow a uniform format. Depending on the type of log, the data may be structured, semi-structured or unstructured. In order to absorb and derive valuable insights from all log files in real-time, the data requires a level of normalization to make it easily parsable.

### Chalenge #3: Digital Transformation
According to Gartner, many organizations, especially midsize enterprises and organizations with less-mature security operations, have gaps in their monitoring and incident investigation capabilities. The decentralized approach to log management in their IT environments makes detecting and responding to threats nearly impossible.

In addition, many organizations rely on SIEM solutions that are limited by cost and capability. SIEM licensing models are based on the volume or velocity of data ingested by the SIEM often increase costs for the technology, making broad data collection cost-prohibitive (although many log management tools have similar pricing models). In addition, as data volumes grow, SIEM tools might experience performance issues, as well as increasing operations costs for tuning and support.

## Log locations

All the system applications create their log files in /var/log and its sub-directories. Here are few important applications and their corresponding log directories −

    Application	Directory

        httpd	/var/log/httpd

        samba	/var/log/samba

         cron	/var/log/

         mail	/var/log/

        mysql	/var/log/

## Journalctl 
#### Use journalctl to View Your System's Logs

### What is journalctl?

journalctl is a command for viewing logs collected by systemd. The systemd-journald service is responsible for systemd’s log collection, and it retrieves messages from the kernel, systemd services, and other sources.

These logs are gathered in a central location, which makes them easy to review. The log records in the journal are structured and indexed, and as a result journalctl is able to present your log information in a variety of useful formats.

### Using journalctl for the First Time
Run the journalctl command without any arguments to view all the logs in your journal:

        journalctl

If you do not see output, try running it with sudo:

        sudo journalctl

If your Linux user does not have sudo privileges, add your user to the sudo group.

### Default Log Format and Ordering

journalctl will display your logs in a format similar to the traditional syslog format. Each line starts with the date (in the server’s local time), followed by the server’s hostname, the process name, and the message for the log.

        Aug 31 12:00:25 debian sshd[15844]: pam_unix(sshd:session): session opened for user example_user by (uid=0)

Your logs will be displayed from oldest to newest. To reverse this order and display the newest messages at the top, use the -r flag:

        journalctl -r

### Paging through Your Logs

journalctl pipes its output to the less command, which shows your logs one page at a time in your terminal. If a log line exceeds the horizontal width of your terminal window, you can use the left and right arrow keys to scroll horizontally and see the rest of the line.

Furthermore, your logs can be navigated and searched by using all the same key commands available in less:

### For more information 
https://www.linode.com/docs/guides/how-to-use-journalctl/

## Logrotate 

Logrotate is a system utility that manages the automatic rotation and compression of log files. If log files were not rotated, compressed, and periodically pruned, they could eventually consume all available disk space on a system.

Logrotate is installed by default on Ubuntu 20.04, and is set up to handle the log rotation needs of all installed packages, including rsyslog, the default system log processor.

### Step 1 — Confirming Your Logrotate Version

Logrotate is installed by default on Ubuntu. However, if you need to install it, run the following commands to update your package list and retrieve the package:

        sudo apt update
        sudo apt install logrotate

If you’re using a non-Ubuntu server, first make sure Logrotate is installed by asking for its version information:

        logrotate --version

```
Output

logrotate 3.14.0

    Default mail command:       /usr/bin/mail
    Default compress command:   /bin/gzip
    Default uncompress command: /bin/gunzip
    Default compress extension: .gz
    Default state file path:    /var/lib/logrotate/status
    ACL support:                yes
    SELinux support:            yes
```    

If Logrotate is installed but the version number is significantly different, you may have issues with some of the configuration options that are explored in this tutorial. Refer to the documentation for your specific version of Logrotate by reading its manual (man) page:

        man logrotate

You can also consult the online version of the Logrotate documentation. Next we’ll look at Logrotate’s default configuration structure on Ubuntu.

### Step 2 — Exploring the Logrotate Configuration

Logrotate’s configuration information can generally be found in two places on Ubuntu:

- /etc/logrotate.conf: this file contains some default settings and sets up rotation for a few logs that are not owned by any system packages. It also uses an include statement to pull in configuration from any file in the /etc/logrotate.d directory.

- /etc/logrotate.d/: this is where any packages you install that need help with log rotation will place their Logrotate configuration. On a standard install you should already have files here for core system tools like apt, dpkg, rsyslog and so on.

By default, logrotate.conf will configure weekly log rotations, with log files owned by the root user and the syslog group, with four log files being retained at a time(rotate 4), and new empty log files being created after the current one is rotated (create).

Let’s take a look at a package’s Logrotate configuration file in /etc/logrotate.d. cat the file for the apt package utility:

        cat /etc/logrotate.d/apt

```
Output
/var/log/apt/term.log {
  rotate 12
  monthly
  compress
  missingok
  notifempty
}

/var/log/apt/history.log {
  rotate 12
  monthly
  compress
  missingok
  notifempty
}

```
This file contains configuration blocks for two different log files in the /var/log/apt/ directory: term.log and history.log. They both have the same options. Any options not set in these configuration blocks will inherit the default values or those set in /etc/logrotate.conf. Any setting in a logrotate file will override logrotate’s default values, which are configured in /etc/logrotate.conf. The options set for the apt logs are:

- rotate 12: keep twelve old log files. This overrides the rotate 4 default.
- monthly: rotate once a month. This overrides the weekly default.
- compress: compress the rotated files. this uses gzip by default and results in files ending in .gz. The compression command can be changed using the compresscmd option.
- missingok: don’t write an error message if the log file is missing.
- notifempty: don’t rotate the log file if it is empty.

These configuration files also inherit the default create behaviour, which instructs Logrotate to create new logs after rotation. This could be overridden with nocreate, although that would effectively disable most of the other functionality.

There are many more configuration options available. You can read about all of them by typing man logrotate on the command line to bring up Logrotate’s manual page.

### For more information refer the following link

[How to manage logfiles with logrotate](https://www.digitalocean.com/community/tutorials/how-to-manage-logfiles-with-logrotate-on-ubuntu-20-04) 
