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
        ```sh
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
        ```sh
        [ERROR] Terminal initialization failed; falling back to unsupported
        mv  $HADOOP_HOME/share/hadoop/yarn/lib/jline*.jar
        cp $HIVE_HOME/lib/jline*.jar $HADOOP_HOME/share/hadoop/yarn/lib/
        ``` 
# 创建表和加载数据

1. 创建表

    ```sql
    create database ilive;
    create table films (id int, name string)
    row format delimited
    fields terminated by ',';
    ```

2. 上传数据到hdfs

    - 数据
    ```
    1,f1
    2,f2
    3,f3
    4,f4
    5,f5
    6,f6
    ```
    - 上传
    ```sh
    hdfs dfs -put films.tb /user/hive/warehouse/ilive.db/films
    ```
    - 验证
    ```sql
    hive> select * from films;
    OK
    1       f1
    2       f2
    3       f3
    4       f4
    5       f5
    6       f6
    ```
    - 其他方式
    ```sql
    load data [local] inpath file_path into table table_name;
    ```
3. external 外部表
    ```sql
    create external table novels(id int, name string)
    row format delimited
    fields terminated by ' '
    location '/hivedata/ilive.db/novels';
    ```
    ```sql
    hive> select * from novels limit 10;
    OK
    1       乃木坂之诗-全集下载
    2       星心传说-全集下载
    3       灵道-全集下载
    4       楚门-全集下载
    5       火影忍者之七夜传说-全集下载
    6       异世之混元大道-全集下载
    7       一个俏妈三个爸-全集下载
    8       震惊玄学圈的吉祥物-全集下载
    9       快乐的变身生活-全集下载
    10      万年古尸-全集下载
    Time taken: 1.876 seconds, Fetched: 10 row(s)
    ```

> 外部表被`drop`，只是被metadata 被删除，实际数据不影响

4. 元数据
    ```sql
    mysql> select * from COLUMNS_V2;
    +-------+---------+-------------+-----------+-------------+
    | CD_ID | COMMENT | COLUMN_NAME | TYPE_NAME | INTEGER_IDX |
    +-------+---------+-------------+-----------+-------------+
    |     3 | NULL    | id          | int       |           0 |
    |     3 | NULL    | name        | string    |           1 |
    |     5 | NULL    | id          | int       |           0 |
    |     5 | NULL    | name        | string    |           1 |
    +-------+---------+-------------+-----------+-------------+
    mysql> select * from TBLS;
    +--------+-------------+-------+------------------+-------+-----------+-------+----------+----------------+--------------------+--------------------+
    | TBL_ID | CREATE_TIME | DB_ID | LAST_ACCESS_TIME | OWNER | RETENTION | SD_ID | TBL_NAME | TBL_TYPE       | VIEW_EXPANDED_TEXT | VIEW_ORIGINAL_TEXT |
    +--------+-------------+-------+------------------+-------+-----------+-------+----------+----------------+--------------------+--------------------+
    |      3 |  1546246469 |     6 |                0 | lpl   |         0 |     3 | films    | MANAGED_TABLE  | NULL               | NULL               |
    |      5 |  1546250158 |     6 |                0 | lpl   |         0 |     5 | novels   | EXTERNAL_TABLE | NULL               | NULL               |
    +--------+-------------+-------+------------------+-------+-----------+-------+----------+----------------+--------------------+--------------------+
    ```
5. 分区表
    ```sql
    create table music(id int, name string)
    partitioned by(artist string)
    row format delimited
    fields terminated by ',';

    hive> desc music;
    OK
    id                      int
    name                    string
    artist                  string
    # Partition Information
    # col_name              data_type               comment

    load data local inpath file_path into table table_name partition(field=field_value)

    /db_name/table_name/field=value/...  
    ```
6. STORED AS 
> SEQUENCEFILE|TEXTFILE|RCFILE

7. ddl -> alter table -> 修改表
    ```sql
    create table art(id int, name string)
    partitioned by(artist string)
    row format delimited
    fields terminated by ',';
    load data local inpath '/home/lpl/hive_data/art_01.dat' into table art partition(artist="a1");
    load data local inpath '/home/lpl/hive_data/art_02.dat' into table art partition(artist="a2");
    /user/hive/warehouse/ilive.db/art/artist=a1/art_01.dat
    /user/hive/warehouse/ilive.db/art/artist=a2/art_02.dat
    hive> show partitions art;
    OK
    artist=a1
    artist=a2

    alter table art drop partition(artist='a2');
    hive> show partitions art;
    OK
    artist=a1
    同时 /user/hive/warehouse/ilive.db/art/artist=a2/art_02.dat 也被删掉

    hive> alter table art add partition(artist='a3') partition(artist='a4');
    OK
    hive> show partitions art;
    OK
    artist=a1
    artist=a3
    artist=a4
    /user/hive/warehouse/ilive.db/art/artist=a3
    /user/hive/warehouse/ilive.db/art/artist=a4

    hive> alter table art rename to art_new;
    OK
    hive> show tables;
    OK
    art_new
    films
    music
    novels

    alter table art add columns(country string);
    hive> desc art;
    OK
    id                      int
    name                    string
    country                 string
    artist                  string
    # Partition Information
    # col_name              data_type               comment
    artist                  string
    hive> select * from art;
    OK
    1       中文 希崎ジェシカ 人妻寝取願望単体作品  NULL    a1
    2       內射白天午休的護士們    NULL    a1
    3       人妻花園劇場 1周年記念ス        NULL    a1
    4       「在我出差時…」出軌通姦 懷有惡意的妻子前男友所拍下的妻子        NULL    a1
    5       園田みおん作品259LUXU 685       NULL    a1
    6       快感風俗フルコース 松下紗栄子   NULL    a1

    ADD是代表新增一字段，字段位置在所有列后面，partition列前，REPLACE则是表示替换表中所有字段

    hive> alter table art change name art_name string first;
    OK
    hive> desc art;
    OK
    art_name                string
    id                      int
    country                 string
    artist                  string
    ```
8. 显示命令
    ```sql
    hive> show databases;
    OK
    default
    ilive
    hive> show tables;
    OK
    art
    films
    music
    novels
    hive> show partitions art;
    OK
    artist=a1
    artist=a3
    artist=a4
    hive> show functions;
    ...
    hive> desc formatted art;
    OK
    # col_name              data_type               comment

    type                    string
    art_name                string
    id                      int

    # Partition Information
    # col_name              data_type               comment

    artist                  string

    # Detailed Table Information
    Database:               ilive
    Owner:                  lpl
    CreateTime:             Sat Jan 05 22:39:04 CST 2019
    LastAccessTime:         UNKNOWN
    Protect Mode:           None
    Retention:              0
    Location:               hdfs://master:8020/user/hive/warehouse/ilive.db/art
    Table Type:             MANAGED_TABLE
    Table Parameters:
            last_modified_by        lpl
            last_modified_time      1546703197
            transient_lastDdlTime   1546703197

    # Storage Information
    SerDe Library:          org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
    InputFormat:            org.apache.hadoop.mapred.TextInputFormat
    OutputFormat:           org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat
    Compressed:             No
    Num Buckets:            -1
    Bucket Columns:         []
    Sort Columns:           []
    Storage Desc Params:
            field.delim             ,
            serialization.format    ,
    ```
    ```sh
    hive> dfs -cat /user/hive/warehouse/ilive.db/art/artist=a1/art_01.dat;
    1,中文 希崎ジェシカ 人妻寝取願望単体作品
    2,內射白天午休的護士們
    3,人妻花園劇場 1周年記念ス
    4,「在我出差時…」出軌通姦 懷有惡意的妻子前男友所拍下的妻子
    5,園田みおん作品259LUXU 685,20歳的AV女優！
    6,快感風俗フルコース 松下紗栄子

    hive> !cat /home/lpl/hive_data/art_02.dat;
    1,僕のねとられ話しを聞いてほしい
    2,まんチラの誘惑 熟成された女のカラダ
    3,M痴女 上山奈々
    4,市川里美 遭催眠光线控制气象女主播
    5,真白杏 六本木俱乐部真实中出乱交猛
    6,向部屋人妻 神宮寺奈緒
    ```
