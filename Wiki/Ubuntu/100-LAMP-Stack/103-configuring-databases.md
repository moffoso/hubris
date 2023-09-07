
### Create a database
Replace 'database' with the corresponding name
```bash
mysql -u root -p
CREATE DATABASE database;
```

### Create a db user
When creating a user follow the format shown below replacing 'name' and 'password' respectivly
```bash
mysql -u root -p
CREATE USER 'name'@'localhost' IDENTIFIED BY 'password';
```

### Grant user acces to a database
Replace database.* with the name of the database
```bash
mysql -u root -p
GRANT ALL PRIVILEGES ON database.* TO 'name'@'localhost';
FLUSH PRIVILEGES;
```
