# Home Assistant
Home Assistant tips and configuration files


## Push Config to Github

I followed this [tutorial](<https://peyanski.com/automatic-home-assistant-backup-to-github/>). 

1. Create private Github repo called HA_Backup
2. Create a `.gitignore` file in the /config folder in Home Assistant. Use nano with `Ctrl-x` to save and close. 

   ```BASH
   # An * ensures that everything will be ignored.
   *
   # You can whitelist files/folders with !, these will not be ignored.
   !*.yaml
   !.gitignore
   !*.md
   !*.sh
   !*.js*

   # Comment these 2 lines if you don't want to include your SSH keys
   !.ssh/
   !id_rsa*

   # Comment this if you don't want to include your Node-RED flows.
   # This is only valid if you install Node-RED as Home Assistant Add-on
   # !node-red/

   # Uncomment this if you don' want to include secrets.yaml
   #secrets.yaml

   # Ignore these files/folders
   .storage
   .cloud
   .google.token
   home-assistant.log
   ```
3. Initialize the repo: `git intit`
4. Add everything: `git add .`
5. Commit: `git commit -m "first commit"`
6. Add GitHub repo as a remote repo: `git remote add origin git@github.com:scottmillerinva/HA_Backup.git`
7. Make a `.ssh` sub folder under /config: `mkdir .ssh`
8. Create a new key and be sure to store it in /config/.ssh NOT THE DEFAULT LOCATION:

   ```BASH
   ssh-keygen -t rsa -b 4096 -C "scottmillerinva@gmail.com"
   Generating public/private rsa key pair.
   Enter file in which to save the key (/root/.ssh/id_rsa): .ssh/id_rsa
   ```
9. Tell git to use new key: `git config core.sshCommand "ssh -i /config/.ssh/id_rsa -F /dev/null"`
10. Copy the public key and add it to GitHub: `cat id_rsa.pub`
11. Push the content to GitHub: `git push -u origin master`
12. Create a script in the /config folder to automate the push: `nano ha_gitpush.sh`
13. Add the following content:

   ```BASH
   # Go to /config folder or 
   # Change this to your Home Assistant config folder if it is different
   cd /config
   
   # Add all files to the repository with respect to .gitignore rules
   git add .
   
   # Commit changes with message with current date stamp
   git commit -m "config files on `date +'%d-%m-%Y %H:%M:%S'`"
   
   # Push changes towards GitHub
   git push -u origin master
   ```

14. Make the script executable: `chmod +x ha_gitpush.sh`
15. Test the script: `./ha_gitpush.sh`
16. Add the following to the bottom of automations.yaml and restart HA:

   ```BASH
   - id: l1k3
     alias: push HA configuration to GitHub repo
     trigger:
     # Everyday at 23:23:23 time
     - at: '23:23:23'
       platform: time
     action:
     - data:
         addon: a0d7b954_ssh
         input: /config/ha_gitpush.sh
       service: hassio.addon_stdin
    ```

