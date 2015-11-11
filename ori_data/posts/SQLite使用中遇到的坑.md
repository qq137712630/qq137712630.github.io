title: SQLite 和 greenDAO 使用中遇到的坑
---

	SQLite 因为是个精简的数据库，所以会缺省了一些方法

## SQLite 没法修改列名
所以要修改列名的话，还是直接新建一个列吧。


## SQLite 没有datediff日期时间相减的方法
如果要实现日期相减的话，那就用sqlite里有个julianday函数转化后可以直接相减,得到的结果是以天(day)为单位数值，如果不足一天会以小数表示。


## greenDAO -- SQLite 开源操作框架-遇到的坑
greenDAO 的确是很好用,但在我自己做项目 [【启程中转】](http://www.wandoujia.com/apps/com.qichengshike.qichengzhongzhuan) 时,还是遇到了坑    
介绍一下这项目 [【启程中转】](http://www.wandoujia.com/apps/com.qichengshike.qichengzhongzhuan) ,这app只为一个需求:    

- 查询12306的中转车票

### 坑1--没有同表联查
我创了个数据库,里面的只有一张表: ``` QUERY_ENTITY ```
结构为:

public class QueryEntity

    private Long id;//这是ID
    private String from_station;//出发地点
    private String to_station;//到达地点
    private String position;//排行第几次车,用于区分车次

中转说明:
	
	上一次的列车的到达地点(to_station)与下一次的列车的出发地点(from_station)是同一个地点,如:
	position=1的列车的到达地点(to_station)与position=2的列车的出发地点(from_station)相同


这种要对比的数据处于同一个表的情况, greendao 支持使用原生 sql 来查询,但并不是很好的, 开始我这样查的: 

	String tempSQL = "SELECT QUERY002.FROM_STATION FROM "
		+ "(SELECT * FROM QUERY_ENTITY WHERE POSITION = '1') QUERY001 JOIN "
		+ "(SELECT * FROM QUERY_ENTITY WHERE POSITION ='2') QUERY002 "
		+ "WHERE QUERY001.TO_STATION = QUERY002.FROM_STATION )";

	Query<QueryEntity> query = mQueryEntityDao.queryBuilder().where(new StringCondition("POSITION = '1' AND TO_STATION = " + "( " +tempSQL+" )")).build();

这样看起来是对的,也查出了 ``` POSITION = '1' ``` 的列车有多少个,但因为 ``` POSITION = '1' ``` 和 ``` POSITION = '2' ``` 的数量是不一定相同的。    
而现在只是查询出来了不重复的 ``` POSITION = '1' ``` 的列车有多少个     
只要执行了 tempSQL 就可以得到正确的列车数据了,而在使用 greendao 的mQueryEntityDao.queryBuilder().where()还是要再一次比较    
我也看了 greendao 的 join 接口, 但这好像是针对不同表的,而且在网上的中文资料也是很少,到后来也没用上,如有资料的,请指导一下        

后来我还是使用了以前的写法:

public static List<QueryEntity> getRawQueryList(SQLiteDatabase mDb, String queryStr, int position)

	String tempSQL = "SELECT QUERY001.* FROM "
		+ "(SELECT * FROM QUERY_ENTITY WHERE POSITION = '1') QUERY001 JOIN "
		+ "(SELECT * FROM QUERY_ENTITY WHERE POSITION ='2') QUERY002 "
		+ "WHERE QUERY001.TO_STATION = QUERY002.FROM_STATION )";
	Cursor mCursor = mDb.rawQuery(sql, null);

## 坑2--SQLite 中,Cursor.getColumnIndex(String)不支持:"表名.列名"
这个本来是放上面的,但在这讲会更容易理解。   

mCursor.getColumnNames()中的数据是这样的:

	0 = {HashMap$HashMapEntry@4557} "QUERY001.FROM_STATION" -> "0"
	1 = {HashMap$HashMapEntry@4560} "QUERY001.TO_STATION" -> "1"

然后这样执行:

	while (mCursor.moveToNext()) {
	    try {
	        int i = mCursor.getColumnIndex("QUERY001.FROM_STATION");
	        String[] temp= mCursor.getColumnNames();
	
	        String s1 = mCursor.getString(0);
	        String s2 = mCursor.getString(mCursor.getColumnIndex(temp[0]));
	        System.out.println(i + s2 + "");
	    } catch (Exception e) {
	        e.printStackTrace();
	    }
	}

执行到 String s2 = mCursor.getString(mCursor.getColumnIndex(temp[0])); 就报错了
所以我就直接根据 mCursor.getColumnNames()排序来获得指定的数据了

---

### 参考资料
[SQLite时间/日期函数](http://blog.csdn.net/raykenio/article/details/7026691)    
[【转】实现Sqlite datediff日期时间相减的方法](http://www.bubuko.com/infodetail-1000541.html)
[ greendao 官网 ](http://greendao-orm.com/)
