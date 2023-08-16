## Linux User Management 

A user is an entity, in a Linux operating system, that can manipulate files and perform several other operations. Each user is assigned an ID that is unique for each user in the operating system. In this post, we will learn about users and commands which are used to get information about the users. After installation of the operating system, the ID 0 is assigned to the root user and the IDs 1 to 999 (both inclusive) are assigned to the system users and hence the ids for local user begins from 1000 onwards. 

In a single directory, we can create 60,000 users. Now we will discuss the important commands to manage users in Linux. 

1. To list out all the users in Linux, use the awk command with -F option. Here, we are accessing a file and printing only first column with the help of print $1 and awk. 

        awk -F':' '{ print $1}' /etc/passwd

2. Using id command, you can get the ID of any username. Every user has an id assigned to it and the user is identified with the help of this id. By default, this id is also the group id of the user.  

        id username

3. The command to add a user. useradd command adds a new user to the directory. The user is given the ID automatically depending on which category it falls in. The username of the user will be as provided by us in the command. 
    
        sudo useradd username

4. Using passwd command to assign a password to a user. After using this command we have to enter the new password for the user and then the password gets updated to the new password.  

        passwd username

5. Accessing a user configuration file.  

        cat /etc/passwd

6. The command to change the user ID for a user.  

        usermod  -u new_id username

7. Command to Modify the group ID of a user.  

        usermod -g  new_group_id username

8. You can change the user login name using usermod command. The below command is used to change the login name of the user. The old login name of the user is changed to the new login name provided.  

        sudo usermod -l new_login_name old_login_name

9. The command to change the home directory. The below command change the home directory of the user whose username is given and sets the new home directory as the directory whose path is provided.  

        usermod -d new_home_directory_path username

10. You can also delete a user name. The below command deletes the user whose username is provided. Make sure that the user is not part of a group. If the user is part of a group then it will not be deleted directly, hence we will have to first remove him from the group and then we can delete him.  

        userdel -r username

To create a user/group use the following commands:

    adduser: add a user to the system.

    passwd: to assign a password to a user.

    userdel: delete a user account and related files.

    addgroup: add a group to the system.

    delgroup: remove a group from the system.

    usermod: modify a user account.

    chage: change user password expiry information.

    sudo: run one or more commands as another user (typically with superuser permissions).

### Relevant files: 

    /etc/passwd (user information)

    /etc/shadow (encrypted passwords)

    /etc/group (group information)
 
    /etc/sudoers (configuration for sudo)

### For more information refer the following link 

[Linux Admin - User Management](https://www.tutorialspoint.com/linux_admin/linux_admin_user_management.htm)