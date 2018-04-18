本版本采用TPC-H_Tools_v2.17.3版本，可能不是最新版本，请自行到以下地址，寻找最新版本。</br>
版本文件位置在dbgen/release.h文件中<br>
[TPC下载页](http://www.tpc.org/tpc_documents_current_versions/current_specifications.asp)</br>
   
TPC-H是一个数据库测试工具，如果对此工具不熟悉，请阅读官方文档 </br>
[TPCH文档](http://www.tpc.org/TPC_Documents_Current_Versions/pdf/tpc-h_v2.17.3.pdf) </br>   

目前TPCH22个SQL，如需要了解，请查看<br> 
[TPCH-查询SQL解析](doc/QuerySQLExplain.MD)<br>

项目在centos7上运行，需要有mysql客户端和GCC，请自行安装。<br>

目录介绍
----
dbgen:工具目录（标准包提供）<br>
dev-tools:作用不明（标准包提供）<br>
ref_data:使用工具包创建好的数据（标准包提供，由于太大目前已删除，请自行从tpc网站下载）<br>

doc:扩展内容，主要是收集的相关资料<br>
test-tools：扩展内容，主要是为了方便测试，制作的脚本和工具<br>
JavaExecuter:一个使用java制作的sql查询执行工具(未完成)<br>

使用说明
----
提供了test-tools目录，增加了一些测试脚本,所有的测试脚本统一使用config配置文件进行参数配置<br>

在mysql下测试需要在my.cnf中设置mysql库表名不区分大小写lower_case_table_names=1<br>

* config.properties
````
# ============通用配置参数-数据库配置信息==============
target_db_ip=172.18.100.62
target_db_port=4000
target_db_user=root
target_db_password=
target_db_name=tpch

# ============dataimport工具参数==============
# 异步导入 0=是，1=否，tidb才可以进行异步导入
target_db_asynchronous=0
# 先删除数据库,0=是，1=否
target_db_delete=0

# ============dataMake工具配置参数==============
# 生成的数据仓库的数量，决定了目标库数据的数量级
target_db_databaseNum=1
````

* dataMake.sh 提供测试数据生成 

* dataImport.sh 创建库表以及导入数据库

* queryMake.sh 在test-tools/queries下生成查询语句

* queryExec.sh 批量运行查询语句，并输出每个查询语句的执行时间

* JavaExecuter 提供基于java的执行器
  安装JDK后执行，java -jar JavaExecuter.jar 即可<br>
  程序会自动读取config配置文件，然后执行queries目录下的22个SQL文件，并记录执行时间<br>
  


================================================================================
标准包转换为mysql实现
----
* 在dbgen/tpcd.h中添加宏定义<br>
````
#ifdef MYSQL
#define GEN_QUERY_PLAN ""
#define START_TRAN "START TRANSACTION"
#define END_TRAN "COMMIT"
#define SET_OUTPUT ""
#define SET_ROWCOUNT "limit %d;\n"
#define SET_DBASE "use %s;\n"
#endif
````
* 复制dbgen/makefile.suite到makefile，同时填充对应的项目<br>
````
CC      = gcc
DATABASE= MYSQL
MACHINE = LINUX
WORKLOAD = TPCH
````

* 制作表初始化脚本<br>
  dbgen/dss.ddl 为创表脚本<br>
  dbgen/dss.ri  为外键脚本<br>
  需要将他们改造为mysql可支持的脚本，为简便起见，将其合并为一个test-tools/dss.sql脚本，后续升级时，需要再次进行制作。表名最好改为小写<br>

* 调整查询脚本<br>
   参数脚本在qgen/queries目下，需要进行修改才能适合与mysql
   
  


TPCH提供了数据生成工具以及SQL语句生成工具,可以在dbgen目录下使用make进行编译</br>
---
* dbgen：数据生成工具可以生成对应的数据，ref_data目录中为提前生成好的数据</br>
使用如下命令进行生成，$1需要替换成数字，即生成的数据的数量，运行后会生成多个.tbl文件，可用于数据导入<br>
 ```
 ./dbgen -s $1
 ```
  
* qgen：SQL语句工具会以qgen/queries下的sql为模版，替换出可执行的sql语句</br>
使用如下命令进行生成，sql模版必须在同一目录下,$1需要替换为数字，代表生成对应的1-22条SQL
````
./qgen -d $1
````


  

         






