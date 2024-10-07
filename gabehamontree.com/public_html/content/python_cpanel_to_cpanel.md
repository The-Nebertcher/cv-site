Title: Python, cPanel -> cPanel Migrations
Date: 2023-12-17 10:20
Category: Python
Tags: blog, tutorial
Slug: cpanel_migrations
Authors: Gabe Hamontree
Summary: Migrating cPanel to cPanel With Python!
Status: published

Here is a much cleaner example script to migrate from cPanel to cPanel:

```
    #!/usr/bin/python
    
    # --------------------------------------------------------------------
    # import libraries
    # --------------------------------------------------------------------
    import os
    import subprocess
    import sys
    import scp
    # --------------------------------------------------------------------
    # Global Variables
    # --------------------------------------------------------------------
    # Making the notes file
    os.system('touch ~/notes.txt')
    
    # Old_Server Info
     old_server = raw_input('What is the Old Server IP?\n')
     old_user = raw_input('What is the Old Account Username?\n')
     old_pass = raw_input('What is the Old Account Password?\n')
    
    # New_Server Info
     chi_server = raw_input('What is the Chi server?\n')
     chi_user = raw_input('What is the Chi username?\n')
     chi_pass = raw_input('What is the Chi Password?\n')
    
    # Gather Info
    directory_size = 'echo -e "\n+++Directory sizes+++";echo -e "Size of /home: $(du -h ~ --max-depth=0 2>/dev/null| awk \'{print $1}\')\n"; echo -e \"Size of /: $(du -h / --max-depth=0 2>/dev/null| awk \'{print $1}\')\n\"; echo -e \"Size of /ftp/pub: $(du -h /ftp/pub --max-depth=0 | awk \'{print $1}\')\n"; echo -e "+++SSL+++"; openssl s_client -connect localhost:443 -ssl2 1>/dev/null 2>/dev/null && echo \"Client may have an SSL. Check https://www.sslshopper.com/ssl-checker.html" || echo \"No SSL." >> ~/notes.txt'
    domains = 'echo -e \"\n+++Domains/Web Root+++\n\nNumber of Domains: $(grep ServerName /etc/httpd/conf/httpd.conf| sed -n -e \' /^\s*#.*$/! p\' | grep -c ServerName)\n\" &&(counter=0) | for x in $(grep -E \'(ServerName|DocumentRoot)\' /etc/httpd/conf/httpd.conf | sed -n -e \' /^\s*#.*$/!p\' | tr -d \'\011\' | sed \'s/^ *//\' | awk \'{print $2}\' | sed -e \'s/^"//\'  -e \'s/"$//\'); do while [[ $counter -lt 2 ]]; do  if [[ $counter -eq 0 ]]; then echo \"Domain: $x\"; let counter=$counter+1; break; elif [[ $counter -eq 1 ]]; then echo -e \"Document Root: $x\"; ls $x >&/dev/null && echo -e \"Size of $x: $(du -h --max-depth=0 $x | awk \'{print $1}\')\n" || echo -e \" \"; let counter=0; break; else counter=0; fi; done; done; >> /notes.txt'
    email_users = 'echo -e \"\n+++Email Users+++" && (dir=mail; for x in $(ls /usr/local); do if [ $x = imap-server-1.0 ]; then dir=imap; elif [ $x = dovecot ]; then dir=maildirs; else :; fi; done; if [[ $dir = \"mail\" ]]; then echo -e \"\nClient is using POP\"; elif [[ $dir = \"imap\" ]]; then echo -e "\nClient is using IMAP 1.0 (Not Dovecot)\"; elif [[ $dir = \"maildirs\" ]]; then echo -e \"\nClient is using IMAP (Dovecot)\"; else echo -e \"\nScript failed to get email program.\n\"; fi; echo -e \"Size of /var/spool/$dir: $(du -h /var/spool/$dir --max-depth=0 | awk \'{print $1}\')\n\";)&& cat /etc/features | tail -n+4 | grep -v mail=-1 | cut -d\':\' -f1 >> ~/notes.txt'
    ftp_users = 'echo -e "\n+++FTP Users+++\n"; cat /etc/features | grep -v ftp=-1 | tail -n+4 | cut -d\':\' -f1 >> ~/notes.txt'
    aliases = 'echo -e "\n+++Email Forwarders+++\n"&&(host="$(hostname)")| sed -n -e \'/root:postmaster/,$p\' /etc/aliases | tail -n+4 | sed -e \"s/:/@$(hostname) forwards to /g\" >> ~/notes.txt'
    databases = 'echo -e "\n+++Databases+++\n" && ls -alh /var/lib/mysql > /dev/null && echo -e \"Size of /var/lib/mysql: $(du -h /var/lib/mysql | sed -n -e \'/^.*\s\/.*lib\/mysql$/p\' | awk \'{print $1}\')\n\" && mysql -e \"show databases;\" >> ~/notes.txt'
    
    # --------------------------------------------------------------------
    # Performing the Account Review
    # --------------------------------------------------------------------
    # Directory Sizes
    os.system('directory_size')
    
    # Domains/Subdomains
    os.system('domains')
    
    # Email Users
    os.system('email_users')
    
    # FTP Users
    os.system('ftp_users')
    
    # Aliases
    os.system('aliases')
    
    # Databases
    os.system('databases')
    
    # --------------------------------------------------------------------
    # Making the Tarball
    # --------------------------------------------------------------------
    # Make Folders
    os.system('cd / && mkdir /migration && mkdir /migration/websites && mkdir /migration/databases && mkdir /migration/ftp')
    
    # Move Site Content
    os.system('cp -rv /var/www/html /migration/websites')
    
    # Move Databases
    os.system('for x in $(mysql -BNe \'show databases\' | grep -v \"^mysql$\|information_schema\|^test$\");do mysqldump $x > /migration/databases/$x.sql;done')
    
    # Move FTP users
    os.system('cp -r /ftp/pub /migration/ftp')
    
    # Checking Size
    os.system('du -h --max-depth=0 /migration')
    
    # Time to tar
    os.system('tar cvzf /migration.tar.gz /migration')
    
    # --------------------------------------------------------------------
    # Moving the Tarball
    # --------------------------------------------------------------------
    
    os.system("rsync -avz /migration.tar.gz "chi_user"@"chi_server":/home/")
    
    # --------------------------------------------------------------------
    # Ending Print
    # --------------------------------------------------------------------
    print("The migration has started and can take a few hours. Please reply to the client and let them know, and also copy the contents of the notes.txt into the ticket. You will still need to check their account for problem applications also!")
```