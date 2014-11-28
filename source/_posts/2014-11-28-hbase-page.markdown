---
layout: post
title: "HBase分页"
date: 2014-11-28 09:45:23 +0800
comments: true
categories: HBase
keywords: HBase分页
description: HBase分页
---
HBase分页的难点在于两点：获取总行数困难、没有行标号。根据HBase的特点获取数据只有三种模式：RowKey唯一定位、Rowkey范围扫描和全表扫描。那么分页可以采用Rowkey的范围扫描，每次扫描一定范围内的数据并且通过PageFilter过滤来限定扫描的行数，如果我们每页10条，那么每次就扫描11条，记住第11条的Rowkey，当点击下一页的时候，将startRowkey设置为第11条的Rowkey，就实现了分页的效果，下面是相关的核心代码，具体项目可以参考[github-HBase-page](https://github.com/findhy/hbase-page) 

    public class PageHBase {
    
    	private Integer currentPageNo=1;//当前页码
    	private Integer pageSize=5;//每页显示行数
    	private Integer totalCount;//总行数
    	private Integer totalPage;//总页数
    	private Integer direction;//下一页：1 上一页2
    	private Boolean hasNext=false;//是否有下一页
    	private String nextPageRowkey;//下一页起始rowkey
    	private List<Map<String, String>> resultList;//结果集List
    	private Map<String,String> paramMap=new HashMap<String,String>();//分页查询参数
    	private Map<Integer,String> pageStartRowMap=new HashMap<Integer,String>();//每页对应的startRow，key为currentPageNo，value为Rowkey
    	private Scan scan=new Scan();
    	
    	public Scan getScan(String startRowkey,String endRowkey){
    		scan.setCaching(100);
    		if(direction==1&&hasNext){
    			scan.setStartRow(Bytes.toBytes(startRowkey));
    			scan.setStopRow(Bytes.toBytes(endRowkey));
    		}else{
    			if(pageStartRowMap.get(currentPageNo)!=null){
    				scan.setStartRow(Bytes.toBytes(pageStartRowMap.get(currentPageNo)));
    				scan.setStopRow(Bytes.toBytes(endRowkey));
    			}else{
    				scan.setStartRow(Bytes.toBytes(startRowkey));
    				scan.setStopRow(Bytes.toBytes(endRowkey));
    			}
    		}
    		this.hasNext=false;
    		this.nextPageRowkey=null;
    		return scan;
    	}
    }


    public class AdunionHbaseService {
    
    	private static final Logger log = LoggerFactory
    			.getLogger(AdunionHbaseService.class);
    	public static final String CLICK_COLUMN_NAME="c";
    	public static final String CLICK_COLUMN_KEY="v";
    	public static final String POSTBACK_COLUMN_NAME="p";
    	public static final String POSTBACK_COLUMN_KEY="v";
    	public static final String ADUNION_TABLE_NAME="adunion_active";
    	private Configuration configuration;
    	private String clientPort;
    	private String retriesNumber;
    	private String zookeeperQuorum;
    	
    	public AdunionHbaseService(String zookeeeper,String port,String retries){
    		try {
    			configuration = HBaseConfiguration.create();
    			this.setClientPort(port);
    			this.setRetriesNumber(retries);
    			this.setZookeeperQuorum(zookeeeper);
    			configuration.set("hbase.zookeeper.property.clientPort", this.getClientPort());
    			configuration.set("hbase.client.retries.number", this.getRetriesNumber());
    			configuration.set("hbase.zookeeper.quorum",this.getZookeeperQuorum());
    		} catch (Exception e) {
    			log.error(e.getMessage());
    		}
    	}
    	
    	public PageHBase getPageHBaseData(String tableName, String columnName,String columnKey,String businessId,String publisherId,String offerId,
    			Date startDate,Date endDate, PageHBase pager) {
    		HConnection con = null;
    		HTable table = null;
    		ResultScanner rs=null;
    		List<Map<String,String>> resultList=new ArrayList<Map<String,String>>();
    		try {
    			if(startDate==null||endDate==null) return pager;
    			
    			con = HConnectionManager.createConnection(configuration);
    			table=(HTable)con.getTable(tableName);
    			
    			StringBuffer startRow=new StringBuffer();
    			StringBuffer endRow=new StringBuffer();
    			startRow.append(startDate.getTime());
    			endRow.append(endDate.getTime());
    			if(pager.getNextPageRowkey()==null) pager.setNextPageRowkey(startRow.toString());
    			Scan scan=pager.getScan(pager.getNextPageRowkey(),endRow.toString());
    			
    			List<Filter> filters=new ArrayList<Filter>();
    			
    			if(StringUtils.isNotBlank(publisherId)){
    				SingleColumnValueFilter filter=new SingleColumnValueFilter(
    						Bytes.toBytes(columnName),
    						Bytes.toBytes("p"),
    						CompareFilter.CompareOp.EQUAL,
    						Bytes.toBytes(publisherId));
    				filters.add(filter);
    			}
    			if(StringUtils.isNotBlank(offerId)){
    				SingleColumnValueFilter filter=new SingleColumnValueFilter(
    						Bytes.toBytes(columnName),
    						Bytes.toBytes("o"),
    						CompareFilter.CompareOp.EQUAL,
    						Bytes.toBytes(offerId));
    				filters.add(filter);
    			}
    			if(StringUtils.isNotBlank(businessId)){
    				SingleColumnValueFilter filter=new SingleColumnValueFilter(
    						Bytes.toBytes(columnName),
    						Bytes.toBytes("b"),
    						CompareFilter.CompareOp.EQUAL,
    						Bytes.toBytes(businessId));
    				filters.add(filter);
    			}
    			Filter pageFilter=new PageFilter(pager.getPageSize()+1);
    			Filter familyFilter=new FamilyFilter(CompareFilter.CompareOp.EQUAL,
    					new BinaryComparator(Bytes.toBytes(columnName)));
    			Filter qualifierFilter=new QualifierFilter(CompareFilter.CompareOp.EQUAL,
    					new BinaryComparator(Bytes.toBytes(columnKey)));
    			filters.add(familyFilter);
    			filters.add(qualifierFilter);
    			filters.add(pageFilter);
    			FilterList filterList=new FilterList(filters);
    			scan.setFilter(filterList);
    			rs = table.getScanner(scan);
    			int totalRow=0;
    			if(rs!=null){
    				for(Result result : rs){
    					totalRow++;
    					if(totalRow==1){
    						pager.getPageStartRowMap().put(pager.getCurrentPageNo(),Bytes.toString(result.getRow()));
    	pager.setTotalPage(pager.getPageStartRowMap().size());
    	}
    					if(totalRow>pager.getPageSize()){
    						pager.setNextPageRowkey(new String(result.getRow()));
    						pager.setHasNext(true);
    					}else{
    						Map<String,String> map = new HashMap<String,String>();
    						for(KeyValue keyValue:result.raw()){
    							String family=new String(keyValue.getFamily());
    							byte[] value=keyValue.getValue();
    							//map.put("rowkey", new String(result.getRow()));
    							Map<Object,Object> jsonMap=JsonUtil.getMapFromJsonObjStr(new String(value));
    							for(Object obj:jsonMap.keySet()){
    								map.put(obj.toString(), jsonMap.get(obj).toString());
    							}
    						}
    						resultList.add(map);
    					}
    				}
    			}
    			pager.setResultList(resultList);
    		}catch(IOException ioe){
    			log.error(ioe.getMessage());
    		} catch (Exception e) {
    			log.error(e.getMessage());
    		}finally{
    			try {
    				rs.close();
    			} catch (Exception e) {
    				log.error(e.getMessage());
    			}
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
    		return pager;
    	}
    }