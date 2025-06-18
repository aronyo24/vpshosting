**To host a React frontend and Django backend on an Apache server in a VPS.
---

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

### 1. **Prepare Your VPS**
   - Update your VPS:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```
   - Install necessary dependencies:
```bash
sudo apt install apache2
sudo apt update
sudo apt install python
sudo apt install libapache2-mod-wsgi-py3
sudo apt install python3-pip
sudo apt install git
 ```

---

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

### 2. **Set Up Django Backend**
   1. **Clone Your Django Project**
      ```bash
	  cd /var/www
      git clone <your-django-repo-url>
      cd <your-django-project-directory>
      ```

   2. **Set Up Virtual Environment**
      ```bash
      python3 -m venv env_name
      source venv/bin/activate
      pip install -r requirements.txt
      ```

   3. **Collect Static Files**
      Edit `settings.py` to configure your `STATIC_ROOT`:
      ```python
      STATIC_ROOT = BASE_DIR / "static"
      ```
      Then run:
      ```bash
      python manage.py collectstatic
      ```

   4. **Run Database Migrations**
      ```bash
      python manage.py migrate
      ```

   5. **Test Your Django App**
      Start Django locally to ensure it's working:
      ```bash
      python manage.py runserver
      ```

---

### 3. **Set Up React Frontend**
   1. **Build Your React App**
      Navigate to your React project directory:
      ```bash
      cd <your-react-project-directory>
      npm install
      npm run build
      ```
      The production build will be in the `build/` folder.



### 4. **Configure Apache**
   1. **Set Up a Virtual Host for Django**
      Create a new Apache configuration file:
      ```bash
      sudo nano /etc/apache2/sites-available/django.conf
	  Ex: sudo nano /etc/apache2/sites-available/admin.247school.conf
      ```
      Add the following configuration:
      ```
      <VirtualHost *:80>
          ServerName yourdomain.com
          DocumentRoot /var/www/html

          Alias /static /path/to/your-django-project/static
          <Directory /path/to/your-django-project/static>
              Require all granted
          </Directory>

          <Directory /path/to/your-django-project>
              <Files wsgi.py>
                  Require all granted
              </Files>
          </Directory>

          WSGIDaemonProcess django_app python-home=/path/to/your-django-project/venv python-path=/path/to/your-django-project
          WSGIProcessGroup django_app
          WSGIScriptAlias / /path/to/your-django-project/wsgi.py

          ErrorLog ${APACHE_LOG_DIR}/django_error.log
          CustomLog ${APACHE_LOG_DIR}/django_access.log combined
      </VirtualHost>
      ```
      Replace paths and `yourdomain.com` with your actual project paths and domain.

   2. **Set Up a Virtual Host for React**
      Create another configuration file:
      ```bash
      sudo nano /etc/apache2/sites-available/react.conf
      ```
      Add the following:
 ```

<VirtualHost *:80>
    ServerName 247school.org
    ServerAlias www.247school.org
    DocumentRoot /var/www/247school/frontend/dist

    <Directory /var/www/247school/frontend/dist>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/247school.org-error.log
    CustomLog ${APACHE_LOG_DIR}/247school.org-access.log combined

    # Redirect all HTTP to HTTPS
    RewriteEngine on
    RewriteCond %{SERVER_NAME} =www.247school.org [OR]
    RewriteCond %{SERVER_NAME} =247school.org
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName 247school.org
    ServerAlias www.247school.org
    DocumentRoot /var/www/247school/frontend/dist

    <Directory /var/www/247school/frontend/dist>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted

        # ✅ Rewrite all non-file requests to index.html for React Router
        RewriteEngine On
        RewriteBase /
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule ^ index.html [L]
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/247school.org-error.log
    CustomLog ${APACHE_LOG_DIR}/247school.org-access.log combined

    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/247school.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/247school.org/privkey.pem
</VirtualHost>
</IfModule>

  ```

   3. **Enable the Sites**
      Enable the Django and React sites:
      ```bash
      sudo a2ensite django.conf
      sudo a2ensite react.conf
      sudo systemctl restart apache2
      ```

---

### 5. **Secure Your Server with SSL (Optional)**
   Install Certbot and get a free Let's Encrypt SSL certificate:
   ```bash
   sudo apt install certbot python3-certbot-apache -y
   sudo certbot --apache
   ```

---

### 6. **Test Your Setup**
   - Django backend: Visit `http://yourdomain.com`.
   - React frontend: Visit `http://react.yourdomain.com`.

---

