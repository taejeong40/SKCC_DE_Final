# hdfs

*로컬 파일을 /usr/terry 위치에 hdfs 저장

`hdfs dfs -put file.txt /user/terry`

*파일 블럭id로 저장 위치 확인하기
`sudo find / -name "*1073743537*" ls`

* -D dfs.block.size=1048576 (1MB) 블럭 사이즈 설정하여 저장 (기본 128MB)

`hdfs dfs -D dfs.block.size=1048576 -put big.dat`

* 최소 블럭 size는 dfs.namenode.fs-limits.min-block-size 설정보다 작으면 오류 발생

`hdfs dfs -D dfs.block.size=1024 -put big.dat big3.dat`

* hdfs 권한
```
hdfs dfs -chmod 644 /user/terry
hdfs dfs -chmod -R 644 /user/terry # -R 하위 폴더까지
```

* 파일조회
```
hdfs dfs -ls -R /usr/terry # 경로 하위 모든 파일/디렉토리 조회
hdfs dfs -help ls
```

* 폴더 만들기
```
hdfs dfs -mkdir test3
hdfs dfs -mkdir -p user/terry # -p user 디렉토리 없으면 만들어서 저장
```

* 디렉토리 없으면 home 경로에 저장됨, working 디렉토리 개념 없음

* 삭제
```
hdfs dfs -rmdir /user/terry/test1 (test1 빈폴더야함)
hdfs dfs -rm -r /user/terry/test3 (-r 하위 포함 삭제)
hdfs dfs -rm -skipTrash file1 file2 (휴지통 이동 없이 완전 삭제)
```

* 삭제된 파일/폴더 보관 $HOME/.Trash/Current 
`hdfs dfs -expunge #휴지통 비우기`

* 파일 이동
```
hdfs dfs -mv /user/terry/.Trash/Current/user/terry/file1
hdfs dfs -mv /user/terry/file3 R_file3 #파일이름 변경
hdfs dfs -mv re_file3 file4 /tmp  # 파일 여러개 이동
```

* 복사
```
hdfs dfs -cp file.txt file2.txt #file2.txt 생성
hdfs dfs -cp work work2 #폴더 복사
```

# yarn

* yarn hue에서 jab browser 확인 가능
```
yarn application -list
yarn application -status <app-id>
yarn application -kill <app-id>
```

* RM WEB UI 8088
* JobHistory Server 19888
* spark history server 18088
* log NodeManager directory : /var/log/hadoop-yarn/container
* log Default HDFS directory : /var/log/hadoop-hdfs

* application id 검색

`yarn application -list -appStates FINISHED | grep 'word count'`

* application id로 log 불러오기

`yarn logs -applicationId <application_id>`

------------
* mapreduce - wordcount 프로그램 수행 // -D mapred.reduce.tasks=3 리듀스 갯수 지정 // input : file.txt  // output : output3

`hadoop jar /user/xxxx/hadoop-examples.jar wordcount -D mapred.reduce.tasks=3 file.txt output3`

----------
#hadoop 설정 정보 수정
```
cd /var/run/cloudera-scm-agent/process/
sudo tree
sudo ls
sudo vi /var/run/cloudera-scm-agent/process/118-hdfs-NAMENODE/core-site.xml
#client 에서 사용하는 설정 정보
cd /etc/hadoop
ls
sudo tree
# NAME 노드 조회
sudo ls | grep NAME
```
------
###Distcp
```
haddop distcp hdfs://source hdfs://tagrget #클러스터간 복사 MR 사용됨
haddop distcp /temp/terry /user/aa #동일 클러스터 내에서 복사 (대량 작업에서 MR 이용하므로 일반 CP보다 속도가 괜찮아)
haddop distcp -m3 file /temp/terry #-m3 mapper 3개 지정하여 file를 복사하여 디렉토리 하위에 저장
```

###REST API - Web HDFS REST API
```
curl -i -L "http://<host>:<port>/webhdfs/v1/<path>?op=OPENxxxx" #open and read a file
curl -i -X PUT "http://<host>:<port>/webhdfs/v1/<path>?op=CREATExxxx" #Create and Write a file
curl -i -X PUT "http://<host>:<port>/webhdfs/v1/<path>?op=MKDIRSxxxx" #Make a Directory
```
-------
#FLUME 실행

`flume-ng agent -name agent1 --conf-file flume.conf #flume.conf 파일에 agent1으로 이름 명시되어 있음`

------
#데이터생성
```
mysql -u root -p
create database airbnb;
show databases;
use airbnb;
source createTable.sql #table 생성
load data local infile "ab_nyc.txt" into table AB_NYC fields terminated by "|" ignore 1 lines; #데이터 로드
select * from AB_NYC limit 10;
desc AB_NYC;
```

#sqoop
```
sqoop list-databases --username root --password cloudera --connect jdbc:mysql://localhost #database조회
sqoop list-tables --username root --password cloudera --connect jdbc:mysql://localhost/airbnb #airbnb database의 table조회
sqoop import-all-tables --username root --password cloudera --connect jdbc:mysql://localhost/airbnb -m 1 #import tables / -m 1 : mapper 1개 지정(미 지정시 4개 기본)
hdfs dfs -ls AB_NYC

sqoop import --username root --password cloudera --connect jdbc:mysql://localhost/airbnb / 
--columns "id, name" --table AB_NYC /
--delete-target-dir --target-dir /user/cloudera/airbnb -m 1  
```

-----
#hive
```
CREATE TABLE products(
	id int,
	name string,
	price int
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE
LOCATION '/user/cloudera/products';

LOAD DATA LOCAL_INPATH '/home/xxxx/xx.csv' OVERWRITE INTO TABLE produts;

ALTER TABLE products SET TABLEPROPERTIES("skip.header.line.count"="1"); //첫줄 제외

CREATE EXTERNAL TABLE products(
	id int,
	name string,
	price int
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE
LOCATION '/user/cloudera/products'
tblproperties(
	'skip.header.line.count'='1'); //첫줄 제외

DESCRIBE FORMATTED products; # 테이블 상세

ALTER TABLE products RENAME products_2019; //table 명 변경
ALTER TABLE products CHANGE id prod_id int; //컬럼 변경
ALTER TABLE products ADD COLUMNS(last_modified date); //컬럼 추가
```

###hive 연결
```
beeline -u jdbc:hive2://localhost:10000 -n cloudera

beeline -u jdbc:hive2://localhost:10000 -n cloudera -f external_products.sql #외부에서 파일 수행
```

-----
#pig

```
#relation으로 데이터 load
raw_products =
LOAD '/user/xxx/products.csv' USING PigStorage(',') AS id:int, name:chararray, price:int);

#group - 특정 key를 기준으로 하나 이상의 relation 내 data grouping
purchases_groupdata = GROUP purchases BY product_id;
purchases_groupdata = GROUP purchases BY (product_id, purchase_data);

#union 
purchase_JUL_2019 = LOAD '/home/xxxxx/xx.txt' USING PigStorage(',') AS (order_id int,...,purchase_date);
purchase_AUG_2019 = LOAD '/home/xxxxx/xx.txt' USING PigStorage(',') AS (order_id int,...,purchase_date);
purchase_union = UNION purchase_JUL_2019, purchase_AUG_2019;

#split - 2개 이상의 relation으로 쪼갤 때 사용
SPLIT raw_products INTO t_shirts if id < 5, pants if id >= 5;

#fiter - tuple(ROW) 추출
group_order = FILTER raw_purchases BY quantity >= 5;

#foreach - fields 추출
purchases_compact = FOREACH raw_purchases GENERATE order_id, product_id, quantity;

#distinct - 중복 tuple 제거
products_purchased = FOREACH raw_purchases GENERATE product_id;
products_purchased_list = DISTINCT products_purchased;

#order by
purchases_orderby_date = ORDER raw_purchases BY purchase_date ASC;

#Limit
sample_purchases = limit raw_purchases 10;

#STORE - 결과 저장
STORE sample_purchases INTO '/usr/xxx/sample' USING PigStorage(',');

#dump - 화면 출력
DUMP raw_products;

#테이블 확인
DESCRIBE raw_products;

#함수
purchases_groupdata = GROUP purchases BY product_id;
tot_products_purchased = FOREACH purchases_groupdata GENERATE group, SUM(raw_products.quantity);
SUM()
AVG()
SUBSTRING(ToString(purchase_date, 'yyyy-MM-dd'), 5, 10);
UPPER()
LOWER()
TRIM()
RTRIM()
LTRIM()

pig # grunt 모드로 변경
fs -ls # hdfs에서 조회
sh ls; # local 조회

fs -getmerge report/ sales_report.txt; # report 하위를 로컬에 sales_report로 저장

pig good.pig #script 실행 / -- 주석
 
pig -x local #local 모드 - MR이 아니므로 속도가 빨라 

```





