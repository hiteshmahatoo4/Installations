sudo snap install docker

mkdir docker && cd docker

### Clone the github repository
https://github.com/oracle/docker-images.git

cd ~/docker/docker-images/OracleDatabase/SingleInstance/dockerfiles

### Download the Oracle version and place it in the respective version folder
https://www.oracle.com/in/database/technologies/oracle19c-linux-downloads.html#license-lightbox

./buildContainerImage.sh -v 19.3.0 -e

docker run --name oracle19c -p 1521:1521 -p 5500:5500 -e ORACLE_PDB=orcl -e ORACLE_PWD=liferay123 -v /opt/oracle/oradata -d oracle/database:19.3.0-ee

PDB(Plugable Database) and system password

docker logs -f oracle19c

docker exec -it <CONTAINER ID> /bin/bash

connect sys/oracle as sysdba

show con_name;

If CDB$ROOT then change it to ORCL (PDB)

ALTER SESSION SET container = ORCL;

show con_name;

SQL> CREATE TABLESPACE liferay_data logging DATAFILE

'/opt/oracle/oradata/ORCLCDB/liferay_data.dbf' SIZE 64m

autoextend ON NEXT 32m maxsize 4096m blocksize 8k EXTENT management local;
  
 SQL> CREATE TEMPORARY TABLESPACE liferay_temp tempfile
 
  2  '/opt/oracle/oradata/ORCLCDB/liferay_temp.dbf'
  
  3  SIZE 64m autoextend ON NEXT 32m maxsize 2048m blocksize 8k
  
  4  EXTENT management local; 
  
SQL> CREATE USER lportal IDENTIFIED  by lportal DEFAULT TABLESPACE

  2  liferay_data TEMPORARY TABLESPACE liferay_temp PROFILE

3  DEFAULT account unlock;

SQL> GRANT CONNECT TO lportal;

SQL> GRANT RESOURCE TO lportal;

SQL> GRANT UNLIMITED TABLESPACE TO lportal;

show user;

bash-4.2$ show sqlplus /nolog

SQL> sqlplus lportal@orcl
  
show user;
  
select table_name from all_tables;
  
## Liferay Configs
  
docker inspect <container id> | grep "IP  Adress"

cp -r ~/Downloads/liferay-dxp-7.4.13.u54/ .
cd ~/liferay-dxp-7.4.13.u54/tomcat-9.0.68/webapps/ROOT/WEB-INF/shielded-container-lib

cp -r ~/Downloads/ojdbc8.jar .
  
cd liferay-dxp-7.4
  
portal-ext.properties

jdbc.default.driverClassName=oracle.jdbc.OracleDriver 
jdbc.default.url=jdbc:oracle:thin:@//172.17.0.2:1521/ORCL 
jdbc.default.username=lportal
jdbc.default.password=lportal
