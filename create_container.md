# Creation and execution of a MA SQL server in docker

https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver16&pivots=cs1-bash (Accessed 06-2023).  

https://hub.docker.com/_/microsoft-mssql-server  

**Bash instructions**

## Pull and run the SQL Server Linux container image

* Pull the image from docker hub

```bash
sudo docker pull mcr.microsoft.com/mssql/server:2022-latest
```

* Run the container. In this case the password is MyStrong@Passw0rd  

```bash
sudo docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD= MyStrong@Passw0rd" \
   -p 1433:1433 --name sql1 --hostname sql1 \
   -d \
   mcr.microsoft.com/mssql/server:2022-latest
```

* View the container with `sudo docker ps -a`  

* If needed, the error log of the server is in  

```bash
docker exec -t sql1 cat /var/opt/mssql/log/errorlog | grep connection
```

## Change System Admin password

* Because of security reasons. `echo $MSSQL_SA_PASSWORD` displays the password.  

```bash
# For now, this is optional because it is for a tutorial. 
sudo docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd \
-S localhost -U SA \
 -P "$(read -sp "Enter current SA password: "; echo "${REPLY}")" \
 -Q "ALTER LOGIN SA WITH PASSWORD=\"$(read -sp "Enter new SA password: "; echo "${REPLY}")\""
 ```

## Connect to the SQL server

* Enter to the container `sudo docker exec -it sql1 "bash"`

* Inside the container, connect locally using **sqlcmd**

```sqlcmd
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "MyurNewStrong@Passw0rd"
```

## Create and query data

* Create a new database  

```sql
-- Creates a new database
CREATE DATABASE TestDB;
-- Lists the existing databases. The natabase abose should appear
SELECT Name from sys.databases;
GO
```

* Insert data  

```sql
USE TestDB;
CREATE TABLE Inventory (id INT, name NVARCHAR(50), quantity INT);
INSERT INTO Inventory VALUES (1, 'banana', 150); INSERT INTO Inventory VALUES (2, 'orange', 154);
GO
```

* Select the data

```sql
SELECT * FROM Inventory WHERE quantity > 152;
GO
```

* Exit the **sqlcmd** promt

```sql
QUIT
``` 

## Connect from outside the container 

```bash
sqlcmd -S <ip_address>,1433 -U SA -P "MyNewStrong@Passw0rd"
```

## Remove the container

```bash
sudo docker stop sql1
sudo docker rm sql1
```

