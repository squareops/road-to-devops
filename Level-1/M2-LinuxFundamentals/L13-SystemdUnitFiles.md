## How To Use Systemctl to Manage Systemd Services and Units

![](Images/systemctl.png)

systemd is an init system and system manager that has widely become the new standard for Linux distributions. Due to its heavy adoption, familiarizing yourself with systemd is well worth the trouble, as it will make administering servers considerably easier. Learning about and using the tools and daemons that comprise systemd will help you better appreciate the power, flexibility, and capabilities it provides, or at least help you to do your job with less hassle.

In this guide, we will be discussing the systemctl command, which is the central management tool for controlling the init system. We will cover how to manage services, check statuses, change system states, and work with the configuration files.

Please note that although systemd has become the default init system for many Linux distributions, it isn’t implemented universally across all distros. As you go through this tutorial, if your terminal outputs the error bash: systemctl is not installed then it is likely that your machine has a different init system installed.

## Service Management

The fundamental purpose of an init system is to initialize the components that must be started after the Linux kernel is booted (traditionally known as “userland” components). The init system is also used to manage services and daemons for the server at any point while the system is running. With that in mind, we will start with some basic service management operations.

In systemd, the target of most actions are “units”, which are resources that systemd knows how to manage. Units are categorized by the type of resource they represent and they are defined with files known as unit files. The type of each unit can be inferred from the suffix on the end of the file.

For service management tasks, the target unit will be service units, which have unit files with a suffix of .service. However, for most service management commands, you can actually leave off the .service suffix, as systemd is smart enough to know that you probably want to operate on a service when using service management commands.

## Starting and Stopping Services
To start a systemd service, executing instructions in the service’s unit file, use the start command. If you are running as a non-root user, you will have to use sudo since this will affect the state of the operating system:

        sudo systemctl start application.service

As we mentioned above, systemd knows to look for *.service files for service management commands, so the command could be typed like this:

        sudo systemctl start application

Although you may use the above format for general administration, for clarity, we will use the .service suffix for the remainder of the commands, to be explicit about the target we are operating on.

To stop a currently running service, you can use the stop command instead:

        sudo systemctl stop application.service

## Restarting and Reloading
To restart a running service, you can use the restart command:

        sudo systemctl restart application.service

If the application in question is able to reload its configuration files (without restarting), you can issue the reload command to initiate that process:

        sudo systemctl reload application.service

If you are unsure whether the service has the functionality to reload its configuration, you can issue the reload-or-restart command. This will reload the configuration in-place if available. Otherwise, it will restart the service so the new configuration is picked up:

        sudo systemctl reload-or-restart application.service

## Enabling and Disabling Services

The above commands are useful for starting or stopping services during the current session. To tell systemd to start services automatically at boot, you must enable them.

To start a service at boot, use the enable command:

        sudo systemctl enable application.service

This will create a symbolic link from the system’s copy of the service file (usually in /lib/systemd/system or /etc/systemd/system) into the location on disk where systemd looks for autostart files (usually /etc/systemd/system/some_target.target.wants. We will go over what a target is later in this guide).

To disable the service from starting automatically, you can type:

        sudo systemctl disable application.service

This will remove the symbolic link that indicated that the service should be started automatically.

Keep in mind that enabling a service does not start it in the current session. If you wish to start the service and also enable it at boot, you will have to issue both the start and enable commands.

## Checking the Status of Services
To check the status of a service on your system, you can use the status command:

        systemctl status application.service

This will provide you with the service state, the cgroup hierarchy, and the first few log lines.

For instance, when checking the status of an Nginx server, you may see output like this:
```
Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2015-01-27 19:41:23 EST; 22h ago
 Main PID: 495 (nginx)
   CGroup: /system.slice/nginx.service
           ├─495 nginx: master process /usr/bin/nginx -g pid /run/nginx.pid; error_log stderr;
           └─496 nginx: worker process
Jan 27 19:41:23 desktop systemd[1]: Starting A high performance web server and a reverse proxy server...
Jan 27 19:41:23 desktop systemd[1]: Started A high performance web server and a reverse proxy server.
This gives you a nice overview of the current status of the application, notifying you of any problems and any actions that may be required.
```
There are also methods for checking for specific states. For instance, to check to see if a unit is currently active (running), you can use the is-active command:

        systemctl is-active application.service

This will return the current unit state, which is usually active or inactive. The exit code will be “0” if it is active, making the result simpler to parse in shell scripts.

To see if the unit is enabled, you can use the is-enabled command:

        systemctl is-enabled application.service

This will output whether the service is enabled or disabled and will again set the exit code to “0” or “1” depending on the answer to the command question.

A third check is whether the unit is in a failed state. This indicates that there was a problem starting the unit in question:

systemctl is-failed application.service
This will return active if it is running properly or failed if an error occurred. If the unit was intentionally stopped, it may return unknown or inactive. An exit status of “0” indicates that a failure occurred and an exit status of “1” indicates any other status.

## System State Overview

The commands so far have been useful for managing single services, but they are not very helpful for exploring the current state of the system. There are a number of systemctl commands that provide this information.

Listing Current Units

To see a list of all of the active units that systemd knows about, we can use the list-units command:

        systemctl list-units

This will show you a list of all of the units that systemd currently has active on the system. The output will look something like this:

```
Output
UNIT                                      LOAD   ACTIVE SUB     DESCRIPTION
atd.service                               loaded active running ATD daemon
avahi-daemon.service                      loaded active running Avahi mDNS/DNS-SD Stack
dbus.service                              loaded active running D-Bus System Message Bus
dcron.service                             loaded active running Periodic Command Scheduler
dkms.service                              loaded active exited  Dynamic Kernel Modules System
getty@tty1.service                        loaded active running Getty on tty1
. . .
```

The output has the following columns:

- UNIT: The systemd unit name
- LOAD: Whether the unit’s configuration has been parsed by systemd. The configuration of loaded units is kept in memory.
- ACTIVE: A summary state about whether the unit is active. This is usually a fairly basic way to tell if the unit has started successfully or not.
- SUB: This is a lower-level state that indicates more detailed information about the unit. This often varies by unit type, state, and the actual method in which the unit runs.
- DESCRIPTION: A short textual description of what the unit is/does.
Since the list-units command shows only active units by default, all of the entries above will show loaded in the LOAD column and active in the ACTIVE column. This display is actually the default behavior of systemctl when called without additional commands, so you will see the same thing if you call systemctl with no arguments:

        systemctl

We can tell systemctl to output different information by adding additional flags. For instance, to see all of the units that systemd has loaded (or attempted to load), regardless of whether they are currently active, you can use the --all flag, like this:

        systemctl list-units --all

This will show any unit that systemd loaded or attempted to load, regardless of its current state on the system. Some units become inactive after running, and some units that systemd attempted to load may have not been found on disk.

You can use other flags to filter these results. For example, we can use the --state= flag to indicate the LOAD, ACTIVE, or SUB states that we wish to see. You will have to keep the --all flag so that systemctl allows non-active units to be displayed:

        systemctl list-units --all --state=inactive

Another common filter is the --type= filter. We can tell systemctl to only display units of the type we are interested in. For example, to see only active service units, we can use:

        systemctl list-units --type=service

## Listing All Unit Files

The list-units command only displays units that systemd has attempted to parse and load into memory. Since systemd will only read units that it thinks it needs, this will not necessarily include all of the available units on the system. To see every available unit file within the systemd paths, including those that systemd has not attempted to load, you can use the list-unit-files command instead:

        systemctl list-unit-files

Units are representations of resources that systemd knows about. Since systemd has not necessarily read all of the unit definitions in this view, it only presents information about the files themselves. The output has two columns: the unit file and the state.

```
Output
UNIT FILE                                  STATE
proc-sys-fs-binfmt_misc.automount          static
dev-hugepages.mount                        static
dev-mqueue.mount                           static
proc-fs-nfsd.mount                         static
proc-sys-fs-binfmt_misc.mount              static
sys-fs-fuse-connections.mount              static
sys-kernel-config.mount                    static
sys-kernel-debug.mount                     static
tmp.mount                                  static
var-lib-nfs-rpc_pipefs.mount               static
org.cups.cupsd.path                        enabled
. . .
```

The state will usually be enabled, disabled, static, or masked. In this context, static means that the unit file does not contain an install section, which is used to enable a unit. As such, these units cannot be enabled. Usually, this means that the unit performs a one-off action or is used only as a dependency of another unit and should not be run by itself.

### For more information:

https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units



