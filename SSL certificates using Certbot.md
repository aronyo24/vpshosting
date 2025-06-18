# SSL certificates using Certbot

### Update the Package List
```sh
sudo apt update
sudo apt upgrade -y
```

### Install Certbot and Apache Plugin

```sh
sudo apt install certbot python3-certbot-apache -y

```

### Obtain SSL Certificates for Your Domain

```sh
syntex : sudo certbot --apache -d domain_name -d domain_name
Example: sudo certbot --apache -d 247school.org -d www.247school.org

```
