# DO
### Method
To utilize the DO with Node-RED, we will use the API via a CLI command. This requires the user to elevate privileges using sudo and enter a password. Since Node-RED runs under the user 'ncdio', we need to modify privileges to allow Node-RED to execute this specific command without a password.
### Elevate privileges for the dido cmd set only
1. Open the sudoers file with the following cmd:
   ```cmd
   sudo visudo
   ```
2. Add the following line to the file under the ```# Allow members of group sudo to execute any command```
. This will restrict all privileges to only this command set:
   ```
      ncdio ALL=(ALL) NOPASSWD: /usr/bin/rmsg dido cmd *
   ```
 4. Save the changes ```ctrl- x``` to exit ```shift - Y ``` to save.
### Elevate privileges for the dido cmd set only 
