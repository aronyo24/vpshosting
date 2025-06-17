### How to Point Domain and Host Django Project using Github on Apache Web Server VPS Hosting
##### Note - If you get Permission Denied Please use sudo
- Login to Your Domain Provider Website
- Navigate to Manage DNS
- Add Following Records:

| Type | Host/Name | Value |
| :---: | :---: | :--- |
| A     | @     | Your Remote Server IP |
| A     | www   | Your Remote Server IP |
| AAAA  | @     | Your Remote Server IPv6 |
| AAAA  | www   | Your Remote Server IPv6 |


- To Access Remote Server via SSH
```sh
Syntax:- ssh -p PORT USERNAME@HOSTIP
Example:- ssh -p 22 root@222.11.22.111
```
#### Note:- Run Below Commands on Your Remote Server Linux Machine or VPS, Not on Your Local Windows Machine
- Verify that all required softwares are installed
```sh
  apache2 -v
  python --version
  apache2ctl -M
  pip --version
- SQLite is Included with Python
  git --version
```
- If Required Softwares and Modules are not Installed then Install them:
```sh
sudo apt install apache2
sudo apt update
sudo apt install python
sudo apt install libapache2-mod-wsgi-py3
sudo apt install python3-pip
sudo apt install git
```
- Install virtualenv
```sh
pip list
sudo apt install python3 python3-venv
```
- Verify Apache2 is Active and Running
```sh
sudo service apache2 status
```
- Verify Web Server Ports are Open and Allowed through Firewall
```sh
sudo ufw status verbose
```
- if not active
```sh
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw reload

```
- Make Connection between Remote Server and Github via SSH Key
- Generate SSH Keys
```sh
ssh-keygen -t ed25519 -C "githubs"
```

```
- Open Public SSH Keys then copy the key
```sh
cat ~/.ssh/id_ed25519.pub
```
- Go to Your Github Repo
- Click on Settings Tab
- Click on Deploy Keys option from sidebar
- Click on Add Deploy Key Button and Paste Remote Server's Copied SSH Public Key then Click on Add Key
- Verify the Connection, Go to Your Server Terminal then run below
```sh
ssh -T git@github.com
// OR
ssh -vT git@github.com
```
- You may get an error git @ github.com: Permission denied (publickey) If you will try to clone it directly on Web Server Public Folder /var/www So we will clone github repo in User's Home Directory then Move it to Web server Public Directory
- Clone Project from your github account
```sh


- Using SSH Path It requires to setup SSH Key on Github
-go to :
Syntax:- cd /var/www/

-clone
Syntax:- git clone ssh_repo_path
Example:- git clone git@github.com:sujitedu/sujit_biswas_w.git
```
- Run ls command to verify that the project is present
```sh
ls
```

- Go to Your Project Directory
```sh
Syntax:- cd /var/www/project_folder_name
Example:- cd /var/www/sujit_biswas_w
```
- Create Virtual env
```sh
Syntax:- python3 -m venv env_name
Example:- python3 -m venv mvs
```
- Activate Virtual env
```sh
Syntax:- source virtualenv_name/bin/activate
Example:- source mb/bin/activate
```
- Install Dependencies
```sh
pip install -r requirements.txt
```

-Go to
```sh
/etc/apache2/sites-available
```
- Create Virtual Host File
```sh
Syntax:-sudo nano your_domain.conf
Example:-sudo nano sujitbiswas.info.conf
```

- Add Following Code in Virtual Host File
```sh
<VirtualHost *:80>
    ServerName www.example.com
    ServerAdmin contact@example.com
    #Document Root is not required
    #DocumentRoot /var/www/project_folder_name
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    
    Alias /static /var/www/project_folder_name/static
    <Directory /var/www/project_folder_name/static>
        Require all granted
    </Directory>
    
    Alias /media /var/www/project_folder_name/media
    <Directory /var/www/project_folder_name/media>
        Require all granted
    </Directory>
    
    <Directory /var/www/project_folder_name/Inner_project_folder_name>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
    
    WSGIDaemonProcess any_name python-home=/var/www/project_folder_name/myprojectenv python-path=/var/www/project_folder_name
    WSGIProcessGroup any_name
    WSGIScriptAlias /  /var/www/project_folder_name/inner_project_folder_name/wsgi.py
</VirtualHost>
```
- Check Configuration is correct or not
```sh
sudo apache2ctl configtest
```
- Enable Virtual Host
```sh
cd /etc/apache2/sites-available/
sudo a2ensite sujitbiswas.info.conf
```
Disable default Apache configuration
```sh
sudo a2dissite 000-default.conf
sudo systemctl reload apache2

```
- Restart Apache2
```sh
sudo service apache2 restart
```
- Open Django Project settings.py
```sh
cd /var/www/miniblog/miniblog
sudo nano settings.py
```
- Make below changes
```sh
ALLOWED_HOST = ["your_domain"]
DEBUG = False
STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'static'
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```
- Restart Apache2
```sh
sudo service apache2 restart

