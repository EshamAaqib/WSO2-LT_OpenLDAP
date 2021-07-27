# WSO2 OpenLDAP Assignment

## Graphical view of the LDAP Structure

![LDAP Structure GUI](https://user-images.githubusercontent.com/75664650/127172711-4d04e208-ae12-498b-906f-87745e4550f7.png)

## Creating a OpenLDAP server to store employee information

### Installing OpenLDAP Server

```
sudo apt-get update
sudo apt-get install slapd
```

### Now lets move the directory based configuration file and the ldap.conf file.

```
cd /etc/ldap
sudo mv slapd.d slapd.d-backup
sudo mv ldap.conf ldap.conf-backup

```

### Now copy the slapd.conf and ldap.conf config file proivded in the Config_Files folder in this repository

```
sudo cp slapd.conf  /etc/ldap/slapd.conf
sudo cp ldap.conf   /etc/ldap/ldap.conf
```
###### You can use ```slappasswd -h {SSHA}``` to generate a hashed password for the admin account. Default password is 123 and its in clear text at the bottom of the slapd.conf file

### Now as we have completed all the step lets create the suffix "dc=ltacademy,dc=com". To do so use the file "Suffix.ldif" provided in the "LDIF_Files" folder 

```
sudo ldapadd -D "cn=admin,dc=ltacademy,dc=com" -h 127.0.0.1  -W -x -f suffix.ldif
```

### Now we have added the suffix. So lets create an Organizational under this suffix so we can add the Employee information to it. To do so use the file "OU_Employee.ldif" provided in the "LDIF_Files" folder

```
sudo ldapadd -D "cn=admin,dc=ltacademy,dc=com" -h 127.0.0.1  -W -x -f OU_Employee.ldif
```

### Now we created the Organizational Unit. So we can add the Employee Information to it. To do so use the file "Users.ldif" provided in the "LDIF_Files" folder

```
sudo ldapadd -D "cn=admin,dc=ltacademy,dc=com" -h 127.0.0.1  -W -x -f Users.ldif
```

###### All the userpasswords in the provided Users.ldif file are encrypted using SSHA (Password for all the users is 123). If you need to add more and encrypt the password use the command ```slappasswd -h {SSHA}``` to generated a hashed password.



