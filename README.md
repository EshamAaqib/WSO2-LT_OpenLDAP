# WSO2 OpenLDAP Assignment

## Graphical view of the LDAP Structure

![LDAP Structure](https://user-images.githubusercontent.com/75664650/127180429-b1e6f111-5aab-4e57-9cb8-27e4402b61a5.png)

### All the LDIF files are stored in the LDIF_Files folder (Link at the bottom of the document)
### Master and Slave config files are stored in the Master_Config and Slave_Config. (Link at the bottom of the document)


## 1) Creating an OpenLDAP server to store employee information

### 1.1) Installing OpenLDAP Server

```
sudo apt-get update
sudo apt-get install slapd
```

### 1.2) Now lets move the directory based configuration file and the ldap.conf file.

```
cd /etc/ldap
sudo mv slapd.d slapd.d-backup
sudo mv ldap.conf ldap.conf-backup
```

### 1.3) Now copy the slapd.conf and ldap.conf config file proivded in the Config_Files folder in this repository and restart slapd

```
sudo cp slapd.conf  /etc/ldap/slapd.conf
sudo cp ldap.conf   /etc/ldap/ldap.conf
sudo systemctl restart slapd.service
```
###### You can use ```slappasswd -h {SSHA}``` to generate a hashed password for the admin account. Default Admin password is 123 and its in clear text at the bottom of the slapd.conf file

### 1.4) Now as we have completed all the step lets create the suffix "dc=ltacademy,dc=com". To do so use the file "Suffix.ldif" provided in the "LDIF_Files" folder 

```
sudo ldapadd -D "cn=admin,dc=ltacademy,dc=com" -h 127.0.0.1  -W -x -f suffix.ldif
```

### 1.5) Now we have added the suffix. So lets create an Organizational Unit under this suffix so we can add the Employee information to it. To do so use the file "OU_Employee.ldif" provided in the "LDIF_Files" folder

```
sudo ldapadd -D "cn=admin,dc=ltacademy,dc=com" -h 127.0.0.1  -W -x -f OU_Employee.ldif
```

### 1.6) Now we created the Organizational Unit. So we can add the Employee Information to it. To do so use the file "Users.ldif" provided in the "LDIF_Files" folder

```
sudo ldapadd -D "cn=admin,dc=ltacademy,dc=com" -h 127.0.0.1  -W -x -f Users.ldif
```

###### All the userpasswords in the provided Users.ldif file are encrypted using SSHA (Password for all the users is 123). If you need to add more and encrypt the password use the command ```slappasswd -h {SSHA}``` to generated a hashed password.

## b) Creating the Organizational Unit called Books (ou=books) under dc=ltacademy,dc=com  and storing the given book information

### Creating Organizational Unit called Books (ou=books). To do so use the file "OU_Books.ldif" provided in the "LDIF_Files" folder

```
sudo ldapadd -D "cn=admin,dc=ltacademy,dc=com" -h 127.0.0.1  -W -x -f OU_Books.ldif
```

### Storing Book information in the Organizational Unit (ou=books). To do so use the file "Books.ldif" provided in the "LDIF_Files" folder

```
sudo ldapadd -D "cn=admin,dc=ltacademy,dc=com" -h 127.0.0.1  -W -x -f Books.ldif
```

## c) Making employee email and mobile attributes unique across LDAP tree structure (Using unique overlay)

### To do so first add the following below the line ```moduleload syncprov.la``` in the slapd.conf file.

```
moduleload      unique
```

### Then add the following below the line ```suffix "dc=ltacademy,dc=com"``` in the slapd.conf file

```
overlay unique
unique_base dc=ltacademy,dc=com
unique_attributes mail
unique_attributes mobile
```

## d) Configuring Audit Overlay so that administrator can track what are the changes that happened to the LDAP server

### To do so first add the following below the line ```moduleload unique``` in the slapd.conf file.

```
moduleload      auditlog
```

### Then add the following below the line ```unique_attributes mobile``` in the slapd.conf file

```
overlay auditlog
auditlog /var/tmp/OPENLDAPlog.ldif
```

###### Now whenever changes are made to the server the audit log will be stored in /var/tmp and the file name would be OPENLDAPlog.ldif

## e) Creating an Organizational Unit called Ou=System under the suffix and creating an user called replicationuser

### To do so use the file "OU_System.ldif" provided in the "LDIF_Files" folder

```
sudo ldapadd -D "cn=admin,dc=ltacademy,dc=com" -h 127.0.0.1  -W -x -f OU_System.ldif
```
###### By running the above command it will create an Organizational unit named System (ou=System) and create a user named replicationuser in it. The password given to the user is 123 and its hashed.

## f) Granting user: replicationuser read access to the entire LDAP tree structure

### To do so add the following to the bottom the the slapd.conf file

```
access to attrs=userPassword
	by * auth

access to dn.subtree="dc=ltacademy,dc=com"
	by dn="cn=replicationuser,ou=System,dc=ltacademy,dc=com" read
	by * none
 ```
 
 ## g) Setting a slave LDAP server ,so that it will replicate the Master LDAP Server in near real time
 
 ### Before creating the slave server first add the following below the line ```auditlog /var/tmp/OPENLDAPlog.ldif``` in the slapd.conf file in the MASTER server
 
 ```
overlay syncprov
syncprov-checkpoint 100 10
syncprov-sessionlog 100
```

### Now we can install openLDAP in the slave server. To do so follow the steps in 1.1), 1.2) and 1.3)

### Now we have configured and installed openLDAP in our slave server. Now we are going to add the following configurations to the slapd.conf file in our slave server to retrive data from our Master server in near realtime.

###### Add the following below the line ```suffix "dc=ltacademy,dc=com"```

```
syncrepl rid=123
		provider=ldap://192.168.88.129:389 # IP of the slave server
                type=refreshAndPersist
                searchbase="dc=ltacademy,dc=com"
                scope=sub
                attrs="*"
                schemachecking=off
                bindmethod=simple
		binddn="cn=replicationuser,ou=System,dc=ltacademy,dc=com"             
                credentials=123
```

###### Now restart slapd to reflect the changes ```sudo systemctl restart slapd.service```

###### Now we have configured the slave server completely and now its receivng data from the Master server in near realtime. To confirm everything is working in order run the following command in the salve server ```ldapsearch -x```


## This whole assignment was written and uploaded to a private github repository. All the files and the whole guide is accessible via the link below :
https://gitfront.io/r/eshamaaqib/ab9812f7798bc2445ed3a9cba1741da8716fcc28/WSO2-OpenLDAP-Assignment/



