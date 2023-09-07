### Usual steps

Install PHP8.2 with Modules
```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.2 -y
sudo apt-get install -y php8.2-cli php8.2-common php8.2-fpm php8.2-mysql php8.2-zip php8.2-gd php8.2-mbstring php8.2-curl php8.2-xml php8.2-bcmath
```

Installed with following modules:
* `php8.2-cli` - command interpreter, useful for testing PHP scripts from a shell
* `php8.2-common` - documentation, examples, and common modules for PHP
* `php8.2-mysql` - for working with MySQL databases
* `php8.2-zip` - for working with compressed files
* `php8.2-gd` - for working with images
* `php8.2-mbstring` - used to manage non-ASCII strings
* `php8.2-curl` - lets you make HTTP requests in PHP
* `php8.2-xml` - for working with XML data
* `php8.2-bcmath` - used when working with precision floats PHP configurations related to Apache are stored in `/etc/php/8.2/apache2/php.ini`. You can list all loaded PHP modules with the following command:

List the installed modules
```bash
php -m
```

