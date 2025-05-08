## Install SQL Server 2022 on Ubuntu 24.04.01 LTS
For installing Microsoft SQL Server 2022 on Ubuntu 24.04, we still use the 22.04 Jammy MSSQL package. 
If Ubuntu 24.04 is a fresh installation, we need LDAP libraries installed for successfully starting mssql-server.service

> download & install libldap-2.5-0
```bash
curl -O http://debian.mirror.ac.za/debian/pool/main/o/openldap/libldap-2.5-0_2.5.13+dfsg-5_amd64.deb
sudo dpkg -i libldap-2.5-0_2.5.13+dfsg-5_amd64.deb
```
> download & install libldap-dev
```bash
curl -O http://debian.mirror.ac.za/debian/pool/main/o/openldap/libldap-dev_2.5.13+dfsg-5_amd64.deb
sudo dpkg -i libldap-dev_2.5.13+dfsg-5_amd64.deb
```
> download & run the script from this repository, install_sql_server_2022_on_Ubuntu_24_04.sh
```bash
chmod +x install_sql_2022_on_Ubuntu_24_04.sh
./install_sql_2022_on_Ubuntu_24_04.sh
```

