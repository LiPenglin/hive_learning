# env
- centos
- mysql
    - [download](https://dev.mysql.com/downloads/mysql/)
    - sudo rpm -ivh MySQL*.rpm
    - xxx 已安装的msyql
    - sudo rpm -e xxx --nodeps 
    - service mysql start
    - service status 
    - sudo mysql_install_db --user=mysql
    - mysql_secure_installation
        - Remove anonymous users
        - Allow root login remotely
    - create user
        ```sql
        create user plli identified by '123';
        select user,host,password from user where user='plli';
        create database plli_db;
        grant all privileges on plli_db.* to plli@'%' identified by '123';
        flush privileges;
        select user,db,host,select_priv,insert_priv,update_priv,delete_priv from mysql.db where user='plli';
        update mysql.user set password=password('new password.') where user='plli' and host='%';
        flush privileges;
        drop user plli@'%';
        ```
    - dml
        - select ... from ...;
        - update ... set ...=..., where ...;
        - insert into ... (...,) values (...,);
        - delete from ... where ...;
    - ddl 
        - create
        - alter
        - drop
    -dcl 
        - grant
- hive1.0
    - 解压
    - /etc/profile
    - conf/hive-env.sh
        ```shell
        export HADOOP_HOME=/data/hadoop-2.6.5
        ```
    - conf/hive.xml
        ```xml
        <configuration>
        <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://ip:3306/hive?createDatabaseIfNotExist=true</value>
        <description>JDBC connect string for a JDBC metastore</description>
        </property>

        <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
        <description>Driver class name for a JDBC metastore</description>
        </property>

        <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
        <description>username to use against metastore database</description>
        </property>

        <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123</value>
        <description>password to use against metastore database</description>
        </property>
        </configuration>
        ```
    - mysql connector
        - [download](https://dev.mysql.com/downloads/connector/j/5.1.html)
        - `cp jarfile $HIVE_HOME/lib`
    - Attention
        ```shell
        [ERROR] Terminal initialization failed; falling back to unsupported
        mv  $HADOOP_HOME/share/hadoop/yarn/lib/jline*.jar
        cp $HIVE_HOME/lib/jline*.jar $HADOOP_HOME/share/hadoop/yarn/lib/
        ``` 