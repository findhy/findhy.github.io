---
layout: post
title: "HBase通过Mysql建立二级索引"
date: 2014-12-06 16:28:39 +0800
comments: true
categories: HBase
keywords: HBase通过Mysql建立二级索引
description: HBase通过Mysql建立二级索引
---
HBase查询数据有三种模式：Rowkey查询、Rowkey范围scan和全表扫描，HBase的Rowkey查询效率很高，毫秒级的，所以我们应该尽量对HBase使用Rowkey查询，而避免范围和全表扫描，思路有很多种，一种是Rowkey的巧妙设计，例如Rowkey是userId#timestamp，这样就可以支持用户和时间戳的查询了，这种方式最简单但也比较局限，例如无法支持较复杂的查询条件，还有Rowkey设计与region数据分布的问题，我们总是希望数据在多个region上均匀分布，又希望经常被一起查的数据在同一个region上，所以全靠Rowkey设计来解决还是比较局限的，本文介绍通过Mysql来建立HBase二级索引的方式，来实现HBase高效率的分页查询、复杂条件组合查询。

## 思路 ##

- 在Mysql中设计HBase的索引表，Rowkey作为主键，其它查询条件作为其它字段，由于数据量很大，Mysql索引表需要做分表
- 通过Python编码实现定时将HBase的Rowkey扫描到Mysql中，这里注意一个点，每次扫描完将最后的Rowkey记录在redis中，下次扫描从这个Rowkey开始
- 前端分页查询先查询索引表，这样就能够实现和Mysql一样的分页效果，总行数还有直接跳转到具体页，HBase是实现不了直接跳转到具体页的
- 从Mysql中根据查询条件查询到具体的Rowkey，再用这些Rowkey到HBase中批量查询，每一个都是Rowkey直接定位，批量查询效率也是毫秒左右返回，整体查询效率是在秒级响应的

<!--more-->

## Mysql索引表设计 ##
索引表设计尽量小，只存Rowkey、时间还有必要的查询字段，Rowkey为主键，并且索引表一定要分区，因为HBase的数据量很大，而Mysql只能撑住单表几百万的数据量，考虑到我们目前的数据量不是很大，做了按月分区：
    DROP TABLE IF EXISTS INDEX_HBASE_CLICK_201412;
    CREATE TABLE INDEX_HBASE_CLICK_201412
    (
       ROW_KEY VARCHAR(100) NOT NULL PRIMARY KEY,
       CLICK_TIME DATETIME NOT NULL COMMENT 'CLICK_TIME',
       PUBLISHER_ID INT(11) COMMENT '媒体ID',
       OFF_ID INT(11) COMMENT '订单ID',
       BUSINESS_ID INT(11) COMMENT '商务ID',
       DATABASE_TIME DATETIME COMMENT '入mysql时间'
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='HBASE_CLICK索引表 ';

## 扫描Rowkey入Mysql ##
这部分是通过Python来实现的，无法提供完整代码，思路就是定时每隔30秒去扫描HBase，然后记住最后一个Rowkey，下次扫描的时候从这个Rowkey开始。其中HbaseDb是通过thrift2来访问HBase。  
	from adunion.db.mysql_db import Mysql
	from adunion.db.hbase_db2 import HbaseDb
	import time
	from adunion.redis.redis_count import redisCount
	from core.logger import logUtil
	from adunion.msg.msg_obj import msgObj
	
	class AdunionIndex(object):
	    hbaseDb=None
	    work_status=False
	    def __init__(self):
	        self.hbaseDb=HbaseDb()
	    def start_work(self):
	        if self.work_status:
	            msgObj.addRedisCount("work_status is True!!!")
	            return
	        self.work_status=True
	        start_time=time.time()
	        msgObj.addHbaseIndexLog("AdunionIndex.AdunionIndex.start_work:begin")
	        last_row_key=redisCount.getHbaseLastKey()
	        new_last_row_key=self.hbaseScan(last_row_key)
	        redisCount.setHbaseLastKey(new_last_row_key)
	        self.work_status=False
	        used=time.time()-int(start_time)
	        msgObj.addHbaseIndexLog("AdunionIndex.AdunionIndex.start_work:end,used:"+str(used))
	    def hbaseScan(self,last_row_key):
	        try:
	            self.hbaseDb.begin()
	            hbase_index_list_click={}
	            hbase_index_list_postback={}
	            row_family_qualifier_list=[]
	            row_family_qualifier_list.append(("c","b"))
	            row_family_qualifier_list.append(("c","o"))
	            row_family_qualifier_list.append(("c","p"))
	            row_family_qualifier_list.append(("p","b"))
	            row_family_qualifier_list.append(("p","o"))
	            row_family_qualifier_list.append(("p","p"))
	            scannerId=self.hbaseDb.openScanner("adunion_log",last_row_key,row_family_qualifier_list)
	            result = self.hbaseDb.getScannerRows(scannerId, 10)
	            new_last_row_key=None
	            if not result:
	                msgObj.addHbaseIndexLog("AdunionIndex.AdunionIndex.hbaseScan:result is null!!!")
	                return last_row_key
	            last_res=result.pop()
	            new_last_row_key=last_res.row
	            for res in result:
	                row_key=res.row
	                time_str=row_key[row_key.index('_')+1:row_key.index('_')+11]
	                mon_key=time.strftime('%Y%m', time.localtime(float(time_str)))
	                if not hbase_index_list_click.has_key(mon_key):
	                    hbase_index_list_click[mon_key]={}
	                if not hbase_index_list_postback.has_key(mon_key):
	                    hbase_index_list_postback[mon_key]={}
	                for col in res.columnValues:
	                    if col.family=="c":
	                        if not hbase_index_list_click[mon_key].has_key(row_key):
	                            hbase_index_list_click[mon_key][row_key]=[]
	                        hbase_index_list_click[mon_key][row_key].append(col.qualifier+"_"+col.value)
	                        hbase_index_list_click[mon_key][row_key].append("timestamp"+"_"+str(col.timestamp))
	                    elif col.family=="p":
	                        if not hbase_index_list_postback[mon_key].has_key(row_key):
	                            hbase_index_list_postback[mon_key][row_key]=[]
	                        hbase_index_list_postback[mon_key][row_key].append(col.qualifier+"_"+col.value)
	                        hbase_index_list_postback[mon_key][row_key].append("timestamp"+"_"+str(col.timestamp))
	            self.hbaseIndexToMysqlClick(hbase_index_list_click)
	            self.hbaseIndexToMysqlPostback(hbase_index_list_postback)
	            return new_last_row_key
	        except Exception,e:
	            logUtil.log("AdunionIndex.hbase_scan"+",error:"+str(e))
	            import traceback
	            msgObj.addErrorLog("AdunionIndex.hbase_scan error:"+str(traceback.format_exc()))
	            logUtil.log(str(traceback.format_exc()))
	        finally:
	            self.hbaseDb.end()
	    def hbaseIndexToMysqlReal(self,hbase_index_param_list,sql):
	        mysql=Mysql()
	        mysql.begin()
	        try:
	            row_size=len(hbase_index_param_list) 
	            prop_size=5
	            mysql.insertManyWithDup(sql,"@@".join(hbase_index_param_list),row_size,prop_size)       
	            msgObj.addHbaseIndexLog("success insert into mysql,len:"+str(row_size)) 
	        except Exception,e:
	            import traceback
	            logUtil.log(traceback.format_exc())
	            msgObj.addErrorLog("AdunionIndex.hbaseIndexToMysqlReal error:"+str(traceback.format_exc()))
	        finally:
	            mysql.end();
	    def hbaseIndexToMysqlPostback(self,hbase_index_list):
	        sql="INSERT INTO INDEX_HBASE_POSTBACK_#mon#(ROW_KEY,POSTBACK_TIME,PUBLISHER_ID,OFF_ID,BUSINESS_ID,DATABASE_TIME) VALUES ('%0%','%1%',%2%,%3%,%4%,NOW()) on duplicate key update PUBLISHER_ID=PUBLISHER_ID"
	        for mon_key in hbase_index_list.keys():
	            sql=sql.replace('#mon#', mon_key)
	            new_hbase_index_list=self.processParamList(hbase_index_list[mon_key])
	            self.hbaseIndexToMysqlReal(new_hbase_index_list,sql)
	    def hbaseIndexToMysqlClick(self,hbase_index_list):
	        sql="INSERT INTO INDEX_HBASE_CLICK_#mon#(ROW_KEY,CLICK_TIME,PUBLISHER_ID,OFF_ID,BUSINESS_ID,DATABASE_TIME) VALUES('%0%','%1%',%2%,%3%,%4%,NOW()) on duplicate key update PUBLISHER_ID=PUBLISHER_ID"
	        for mon_key in hbase_index_list.keys():
	            sql=sql.replace('#mon#', mon_key)
	            new_hbase_index_list=self.processParamList(hbase_index_list[mon_key])
	            self.hbaseIndexToMysqlReal(new_hbase_index_list,sql)
	    def processParamList(self,hbase_index_list):
	        new_hbase_index_list=[]
	        for row_key in hbase_index_list.keys():
	            col_list=hbase_index_list[row_key]
	            publisher_id=None
	            business_id=None
	            offer_id=None
	            data_time=None
	            for col in col_list:
	                qualifier=col.split('_')[0]
	                qualifier_value=col.split('_')[1]
	                if qualifier=='p':
	                    publisher_id=qualifier_value
	                elif qualifier=='b':
	                    business_id=qualifier_value
	                elif qualifier=='o':
	                    offer_id=qualifier_value
	                elif qualifier=='timestamp':
	                    data_time=qualifier_value
	            data_time_str=time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(float(data_time)/1000))
	            new_hbase_index_list.append(",".join([str(row_key),str(data_time_str),str(publisher_id),str(offer_id),str(business_id)]))
	        return new_hbase_index_list
	adunionIndex=AdunionIndex()

## Java实现查询 ##
对应业务层来说，他只需要查询Mysql的索引表就可以了，查出来Rowkey的list，再用这些Rowkey来批量查询HBase就可以了：  
	public List<Map<String,String>> getRowByRowKeyFromHBase(List<String> row_key_list,String dataType){
			HConnection con = null;
			HTable table = null;
			List<Map<String,String>> resultList=new ArrayList<Map<String,String>>();
			try {
				con = HConnectionManager.createConnection(configuration);
				table=(HTable)con.getTable(ADUNION_TABLE_NAME);
				List<Get> gets=new ArrayList<Get>();
				String family=null;
				if(dataType!=null&&dataType.equals(DATA_TYPE_CLICK)){
					family="c";
				}else if(dataType!=null&&dataType.equals(DATA_TYPE_POSTBACK)){
					family="p";
				}
				for(int i=0;i<row_key_list.size();i++){
					Get get=new Get(Bytes.toBytes(row_key_list.get(i)));
					get.addColumn(Bytes.toBytes(family), Bytes.toBytes("v"));
					gets.add(get);
				}
				Result[] result=table.get(gets);
				for(Result res:result){
					Map<String,String> map = new HashMap<String,String>();
					for(KeyValue keyValue:res.raw()){
						String familyCol=new String(keyValue.getFamily());
						byte[] value=keyValue.getValue();
						//map.put("rowkey", new String(result.getRow()));
						Map<Object,Object> jsonMap=JsonUtil.getMapFromJsonObjStr(new String(value));
						for(Object obj:jsonMap.keySet()){
							map.put(obj.toString(), jsonMap.get(obj).toString());
						}
					}
					resultList.add(map);
				}
			}catch(IOException ioe){
				log.error(ioe.getMessage());
				ioe.printStackTrace();
			} catch (Exception e) {
				log.error(e.getMessage());
				e.printStackTrace();
			}finally{
				try {
					table.close();
				} catch (Exception e) {
					log.error(e.getMessage());
				}
				try {
					con.close();
				} catch (Exception e) {
					log.error(e.getMessage());
				}
			}
			return resultList;
		}