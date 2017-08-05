---
title: MongoDB总结
date: 2017-06-01
comments: false
categories: MongoDB
tags: MongoDB
---

## 一、操作数据库 ##

1. 创建和切换数据库 ：`use dbName`
2. 查看数据库 ：`show dbs`
3. 删除数据库 ：`db.dropDatabase()`

## 二、操作集合 ##

 1. 查看集合 ：`show collections`
 2. 删除集合 ：`db.集合名称.drop()`
 3. 插入数据 ：`db.集合名称.insert({"k1":"v1","k2":"v2"})`
 4. 插入多条数据 ：

	- 插入数组：
		`db.集合名称.insert([{"url":"www.sina.com","name":"新浪"},{"url":"www.baidu.com"}]);`

	- 循环插入
		`for(var i=0;i<10;i++){db.集合名称.insert({"url":"zyx-"+i};}`
 5. 查询数据：`db.集合名称.find({查询条件}[,{设置显示的字段}])`

	- 查询所有数据 ：`db.集合名称.find()`
		
	- 查询一条数据 ：`db.集合名称.findOne()`
		
	- 带条件查询：
		例：查询name为zyx 的记录 --- `db.infos.find({"name":"zyx"})`
		
	- 设置查询的字段(投影操作) ：要查的设为1，不查设为0  (不建议这么查)
	
		例：查询所有url的记录不带_id --- `db.infos.find({查询条件},{"_id":0,"url":1})`
		
	- 关系查询：大于($gt)，大于等于($gte)，小于($lt)，小于等于($lte)，不等于($ne)，等于(k:v,$eq)
		
		例1：查询分数大于等于60的
			`db.students.find({"score":{"$gte":60}},{"_id":0})`

		例2：查询性别是男的
			`db.students.find({"sex":"男"},{"_id":0})`

		例3：查询年龄小于20的
			`db.students.find({"age":{"$lt":20}},{"_id":0})`

		例4：查询姓名不是王五的
			`db.students.find({"name":{"$ne":"王五"}},{"_id":0})`
		
	- 逻辑查询：与($and),或($or),非($not,$nor)
	
		例1：查询分数大于60且小于80的记录
			`db.students.find({"score":{"$gt":60,"$lt":80}},{"_id":0}).pretty()`
				
		例2：查询年龄大于19且分数大于80的记录
			`db.students.find({"$or":[
				{"age":{"$gt":19}},
				{"score":{"$gt":80}}
			]}).pertyy()`

		例3：查询年龄小于等于19且分数小于等于80的记录(即是对例2取非)
			`db.students.find({"$nor":[
				{"age":{"$gt":19}},
				{"score":{"$gt":80}}
			]}).pertyy()`

		例4：查询年龄在10岁到20岁和30到40岁之间的记录
			`db.person.find({"$or":[{"age":{"$gt":10,"$lt":20}},
			{"age":{"$gt":30,"$lt":40}}]},{"_id":0}).pretty()`
		
	- 求模：`$mod`：语法  `{$mod:[数字,余数]}`
	
		例：查询年龄为21的记录
			`db.students.find({"age":{"$mod":[21,1]}}).pretty()`
		
	- 范围查询：
	
		在范围内：`$in` 语法：`{$in:["str1","str2","str3"]}`

		例：查询姓名是“张三”，“李四”，“王五”的信息
		`db.students.find({"name":{"$in":["张三","李四","王五"]}}).pretty()`

		不在范围内：`$nin` 语法：`{$nin:["str1","str2","str3"]}`

		例：查询姓名是“张三”，“李四”，“王五”的信息
		`db.students.find({"name":{"$nin":["张三","李四","王五"]}}).pretty()`
		
	- 数组： 几个运算符：`$all,$size,$slice,$elemMatch`
		
		***注：①$all不仅可以用于数组上，也可以用于一个数据上
			②可以用索引进行操作：key.index(index下标从0开始)
			③$slice可以对数组信息进行切片
				n:表示返回前n个
				-n:表示返回后n个
				[3,4]:表示跳过1个，返回4个***

		例1：查询报了课程语文、数学的学生信息
			`db.students.find({"course":{"$all":["语文","数学"]}}).pretty()`

		例2：查询地址在临潼区的学生(用$all)
			`db.students.find({"loc":{"$all":["临潼区"]}}).pretty()`

		例3：查询报了3门课程的学生信息
			`db.students.find({"course":{"$size":3}}).pretty()`

		例4：查询第一门课是语文的学生
			`db.students.find({"course.1":"数学"}).pretty()`

		例5：查询年龄是19岁且只要求显示两门课程

		(1)显示前两门
			`db.students.find({"age":19},{"course":{"$slice":2}}).pretty()`

		(2)显示后两门
			`db.students.find({"age":19},{"course":{"$slice":2}}).pretty()`
 
		(3)跳过2门,显示3门
			`db.students.find({"age":19},{"course":{"$slice":[2,3]}}).pretty()`

		嵌套查询：

		例：查询年龄大于等于19岁且父母有是工人的学生信息
				`db.students.find({"$and":[{"age":{"$gte":19}},{"parents":{"$elemMatch":{"job":"工人"}}}]}).pretty()`
		判断某个字段是否存在：$exists

		例1：查询有字段parents的学生信息
			`db.students.find({"parents":{"$exists":true}}).pretty()`
		例2：查询不具有字段course的学生信息
			`db.students.find({"course":{"$exists":false}}).pretty()`
		where查询
		例：查询年龄大于20的
		`db.students.find({"$where":"this.age>20"}).pretty()
		db.students.find("this.age>20").pretty()
		db.students.find(function(){
			return this.age>20;
		}).pretty()
		db.students.find({"$where":function(){
			return this.age>20;
		}}).pretty()`
		
		正则表达式:
			基础语法：`{key:正则标记}`
			完整语法：`{"$regex":正则标记,"$options":选项}`
			例1：查询名字已张开头的学生信息
				`db.students.find({"name":/^张/}).pretty()`
			例2：查询名字中含有a的学生信息
				`db.students.find({"name":/a/i}).pretty()`//  i:区分大小写
				`db.students.find({"name":{$regex:/a/i}}).pretty()`
				`db.students.find({"name":{"$regex":/a/,"$options":"$i"}}).pretty()`
			例3：查询coures带有“语”的学生信息
				`db.students.find({"course":/语/}).pretty()`
		排序：sort() 函数   1:升序   -1:降序   `$natural`：自然顺序(记录插入顺序)  
			例1：根据成绩降序排序 
				`db.students.find().sort({"score":-1}).pretty()`
				`db.students.find().sort({"$natural":-1}).pretty()`
		分页：
			`skip(n)`：表示跨过多少数据行
			`limit(n)`：取出的数据行的个数显示
			例1：分页显示(第一页：skip(0),limit(5)),并且score降序显示
				`db.students.find().skip(0).limit(5).sort({"score":-1}).pretty()`
			例1：分页显示(第二页：skip(0),limit(5)),并且score降序显示
				`db.students.find().skip(5).limit(5).sort({"score":-1}).pretty()`



 6. 数据更新
	`update()`函数
		语法：`db.集合.update(更新条件，新的对象数据，upsert，mutil);`
			|- upsert：如果要更新的数据不存在，则增加一条新的内容(true:增加，false:不增加)
			|- mutil：表示是否只更新满足条件的第一行数据(true:全更新，false：只更新第一条)
		例1：（更新存在的数据）将年龄为19岁的学生成绩更新为100
			`db.students.update({"age":19},{"$set":{"score":100}},false,false)`
		例2：（更新不存在的数据）将年龄为30岁的学生成绩更新为71
			`db.students.update({"age":30},{"$set":{"score":100}},false,false)`
	`save()`函数：根据_id更新，_id存在就更新，不存在就插入一条数据
		例：更新_id为58664e79749d012a9fb04022的学生年龄为50
			`db.students.save({"_id" : ObjectId("fff64e79749d012a9fb04022"),"age":90})`
	修改器：10种修改器
		(1) `$inc`：语法  `{"$inc":{"字段":内容}}`    (increase)
			例：将年龄为19岁的学生的score减30且age加1岁
				`db.students.update({"age":19},{"$inc":{"score":-30,"age":1}},false,true)`
 		(2) `$set`：语法  `{"$set":{"字段":"新内容"}}`
 			例：将年龄为20岁的score设为68分
 				`db.students.update({"age":20},{"$set":{"score":68}})`
 		(3) `$unset`: 删除某个字段   语法  `{"$set":{"字段":1}}`
 			例：将name为张三的age和score删除
 				`db.students.update({"name":"张三"},{"$unset":{"age":1,"score":1}})`
 		(4) `$push`: 追加内容到指定的成员之中(基本是数组)
 				语法：`{"$push":{"成员":"内容"}}`
 				例：给张三添加一门课程“语文”
 					`db.students.update({"name":"张三"},{"$push":{"course":"语文"}})`
 		(5) `$pushAll`:与`$push`是类似的，但可以追加多个内容到字段中
 				语法：`{"$pushAll":{"成员":数组内容}}`
 				例：给张三添加课程政治，美术，体育
 					`db.students.update({"name":"张三"},{"$pushAll":{"course":["政治","美术","体育"]}})`
 		(6) `$addToSet`:向数组里添加一个新的内容，只有当内容不存在时，才会添加
 				语法：`{"$addToSet":{"成员":内容}}`
 				例：给张三添加一门英语
 					`db.students.update({"nmae":"张三"},{"$addToSet":{"course":"英语"}})`
 		(7) `$pop`:删除数组内的数据
 				语法：`{"$pop":{"成员":内容}}` 内容为-1表示删除第一个，为1删除最后一个
 				例：删除张三第一门课程
 					`db.students.update({"name":"张三"},{"$pop":{"coures":-1}})`
 		(8) `$pull`:从数组内删除以一个指定内容的数据
 				语法：`{"$pull":{"成员":"数据"}}`，此数据为要删除的数据
 				例：删除张三 的英语课程
 					`db.students.update({"name":"张三"},{"$pull":{"course":"英语"}})`
 		(9) `$pullAll`:一次删除多个内容
 				语法：`{"$pullAll":{"成员":数组内容}}`
 				例：删除张三的政治体育课程
 					`db.students.update({"name":"张三"},{"$pullAll":{"course":["音乐","体育"]}})`
 		(10)`$rename`: 修改字段的名称
 				语法：`{"$rename":{"旧的成员名称":"新的成员名称"}}`
 				例：讲张三的“name”字段改名为“姓名”
 					`db.students.update({"name":"张三"},{"$rename":{"name":"姓名"}})`

 7. 数据删除
	`remove()`函数：有两个可选项
		删除条件：满足条件的数据被删除
		是否只删除第一个数据(默认删除全部满足条件的)：如果设为true或1,则删除第一条满足条件 的
		例1：删除集合的全部信息
			`db.infos.remove({})`    3.x
		例2：删除name有“张”的学生信息
			`db.students.remove({"name":/张/})`
		例2：删除name有“张”的学生信息
			`db.students.remove({"name":/王/},true)`

 8. 游标：类似于ResultSet
	是否有下一个：`hasNext()`
	下一个：`next()`
	例：逐行打印students的姓名
		`var cursor=db.students.find();
		while(cursor.hasNext()){
			var doc=cursor.next();
			print(doc.name);
		}`
	注：游标取出来的内容是Object类型的,如果需要将数据以json的形式打印，可以用`printjson()`函数。
	例：`var cursor=db.students.find();
		while(cursor.hasNext()){
			var doc=cursor.next();
			printjson(doc);
		}`

 9. 索引：提升数据检索性能
	查看集合默认索引状态：
		`db.集合.getIndexes()`
		`db.students.getIndexes()`
	- 创建索引：
		`db.集合.ensureIndex({字段:1})  ` 1表示索引按照升序排列，-1为降序
		注：索引的名称是自动命名的。命名规范：字段名称_索引排序模式
		例：`db.students.ensureIndex({"age":-1})`
	针对当前的age字段上的索引做一个分析
		`db.students.find({"age":19}).explain();`    扫描方式：IXSCAN
	针对score字段查询
		`db.students.find({"score":{"$gt":60}}).explain(); ` 扫描方式：COLLSCAN
	- 创建复合索引：
		`db.students.ensureIndex({"age":-1,"score":-1},{"name":"age_-1_score_-1_index"})`
	- 强制使用索引
		`hint({"age":-1,"score":-1})`

	- 删除索引
		删除一个索引：`db.students.dropIndex({"age":-1,"score":-1})`
		删除全部索引(除“_id”的索引)：`db.students.dropIndexes()`

	- 唯一索引：用在某个字段上，使该字段的内容不重复,如果有某个文档没有该字段，则会创建不成功
		创建：`db.students.ensureIndex({"name":1},{"unique":true})`
		`db.stduents.insert({"name":"张三"})`
		如果插入的文档的内容在该字段上相同则会报错：
			`E11000 duplicate key error collection: zyx.students index: name_1 dup key`
	- 过期索引：要实现过期索引，需要保存一个时间字段,但是这个时间往往不怎么准确
		创建：
		`db.集合.ensureIndex({"time":1},{expireAfterSeconds:10})`
		例：
			创建一个手机验证码集合，并设置10秒过期
			`db.phones.ensureIndex({"time":1},{expireAfterSeconds:10})
		    db.phones.insert({"tel":110,"code":000,"time":new Date()})
			db.phones.insert({"tel":111,"code":001,"time":new Date()})
			db.phones.insert({"tel":112,"code":002,"time":new Date()})
			db.phones.insert({"tel":113,"code":003,"time":new Date()})
			db.phones.insert({"tel":114,"code":004,"time":new Date()})
			db.phones.insert({"tel":115,"code":005,"time":new Date()})`
	- 全文索引：
		为某几个字段设置全文索引：
			创建：`{"字段1":"text","字段2":"text"}`   
			模糊查询：
				要实现全文检索，则使用`"$text"`判断符，而要想进行数据的查询则使用`"$search"`
				|- 查询指定关键字：`{"$search":"查询关键字"}`
				|- 查询多个关键字(或关系)：`{"$search":"查询关键字1 查询关键字2 ..."}`
				|- 查询多个关键字(与关系)：`{"$search":"\"查询关键字1\" \"查询关键字2\" 		..."}`
				|-查询多个关键字(排除某一个)：`{"$search":"查询关键字1 查询关键字2 ... -	排除关键字"}`
				例：`db.students.find({"$text":{"$search":"关键字"})`
			使用相似度的打分来判断检索结果：默认升序，分数越高越准确
				`db.students.find({"$text":{"$search":"关键字"}},{"score":{"$meta":"textScore"}})`
				排序操作：
					`db.students.find({"$text":{"$search":"关键字"}},{"score":{"$meta":"textScore"}}).sort({"score":{"$meta":"textScore"}})`
		为所有字段设置全文索引：
		`	db.集合.ensureIndex({"$**":"text"})	`
			注：尽可能别用。。。因为慢
	- 地理信息索引：分为两类 ①2D平面索引②2DShere球面索引
		为集合定义2D索引：
		`	db.集合.ensureIndex({"字段":"2d"})`
		例：
			创建2d索引：
				`db.shop.ensureIndex({"loc":"2d"})`
			创建集合：
			    `db.shop.insert({"loc":[10,10]})
				db.shop.insert({"loc":[11,10]})
				db.shop.insert({"loc":[11,11]})
				db.shop.insert({"loc":[13,12]})
				db.shop.insert({"loc":[50,60]})
				db.shop.insert({"loc":[90,95]})`
		实现位置查询：
			两种：(1) `"$near"`:查询距离某个点最近的坐标点
 				  (2) `"$geoWithin"`：查询某个形状内的点,可以设置范围:
 				  	 ① 矩形范围`（$box）`: `{"$box":[[x1,y1],[x2,y2]]}`
 				  	 ② 圆形范围`（$center）`: `{"$center":[[x,y],r]}`
 				  	 ③ 多边形范围`（$polygon）`: `{"$poltgon":[[x1,y1],[x2,y2],[x3,y3]....]}`
 				  (3) 使用`runCommand()`实现信息查询
 			例1：假设现在坐标:[11,11]   
 				`db.shop.find({"loc":{"$near":[11,11]}})`
 				注：实际上返回集合内的前100个点
 			例2：设置查询距离范围
 				`db.shop.find({"loc":{"$near":[11,11],"$maxDistance":5}})`
 			例3：查询矩形范围
 				`db.shop.find({"loc":{"$geoWithin":{"$box":[[9,9],[12,12]]}}})`
 			例4：查询圆形范围
 				`db.shop.find({"loc":{"$geoWithin":{"$center":[[11,11],2]}}})`
 			例5：查询多边形范围
 				`db.shop.find({"loc":{"$geoWithin":{"$polygon":[[4,6],[6,8],[8,9],[5,8]]}}})`
 			例6：使用`runCommand()`
 		    	`db.runCommand({"geoNear":"shop","near":[11,11],"maxDistance":3,"num":2})`

 10. 聚合：统计操作，称为聚合
	取得集合个数：`db.集合.count()`
		注：也可以加条件进行模糊查询，如：`db.students.count({"name":/王/})`
		例：`db.shop.count()`
	消除重复数据：没有直接的函数支持，只能使用runCommand()函数
		语法：`db.runCommand({"distinct":"集合","key":"字段"})`
		例：`db.runCommand({"distinct":"students","key":"name"})`
	- Group操作：使用group操作可以实现数据的分组操作，在MongoDB里面会将集合依据指定的key的不	         同进行分组操作，并且每一个组都会产生一个处理的文档结果 
		例：
			`db.runCommand({"group":{
				"ns" : "students",
				"key" : {"age":true},
				"initial" : {"count":0},
				"condition" : {"age":{"$gte":18}},
				"$reduce" : function(doc,prev){
					prev.count++;
					print("doc="+doc+" prev="+prev);
				}
			}})`

	- MapReduce：
	
		MapReduce是大数据的精髓所在，实际分为两步处理数据：
			Map：将数据分别取出
			Reduce：负责数据的最后处理
		建立一组雇员数据
		`db.emps.insert({"name":"张三","age":30,"sex":"男","job":"CLERK","salary":1000})
		db.emps.insert({"name":"李四","age":28,"sex":"女","job":"MANAGER","salary":4000})
		db.emps.insert({"name":"王五","age":31,"sex":"男","job":"CLERK","salary":1500})
		db.emps.insert({"name":"赵六","age":33,"sex":"女","job":"MANAGER","salary":5000})
		db.emps.insert({"name":"孙七","age":27,"sex":"男","job":"CLERK","salary":1000})
		db.emps.insert({"name":"王八","age":35,"sex":"男","job":"PRESIDENT","salary":7000})`

		使用MapReduce操作最终会将处理结果保存在一个单独的集合里，最终处理效果如下：
		例1：按照值为分组，取得每个职位的人名
				● 编写分组的定义
					`var jobMapFun = function(){
						emit(this.job,this.name); //按照job分组，取出name
					};`
					经过上述函数操作后结果如下：
						第一组：`{"key":"CLERK","values":[姓名1,姓名2...]}`
				● 编写Reduce操作
					`var jobReduceFun = function(key,values){
						return {"job":key,"names":values};
					};`
				● 针对MapReduce处理完成的数据进行处理
					`var jobFinalizeFun = function(key,values){
						if(key=="PRESIDENT"){
							return {"job":key,"names":values,"info":"公司老大"}
						}
						return {"job":key,"names":values};
					}`
				● 进行操作的整合
					第一种
						`db.runCommand({
							"mapreduce":"emps",
							"map":jobMapFun,
							"reduce":jobReduceFun,
							"out":"t_job_emp"
						})`
					第二种
						`db.runCommand({
							"mapreduce":"emps",
							"map":jobMapFun,
							"reduce":jobReduceFun,
							"out":"t_job_emp",
							"finalize": jobFinalizeFun
						})`
		例2：统计出各性别的人数，总工资，平均工资，最高工资，最低工资，雇员姓名
			●  编写Map
				`var sexMapFun = function(){
					//定义好分组条件，以及取出每个集合要取出的内容
					emit(this.sex,{"ccount":1,"csal":this.salary,"cmax":this.salary,"cmin":this.salary,"cname":this.name})
				};`
			● 编写Reduce操作
				`var sexReduceFun = function(key,values){
					var total = 0;//统计人数
					var sum = 0;//统计宗工资
					var max = values[0].cmax;
					var min = values[0].cmin;
					var name = new Array()
					for(var x in values){//循环取出values中的内容
						total += values[x].ccount;// 人数增加
						sum += values[x].csal;
						if(max < values[x].cmax){
							max = values[x].cmax;
						}	
						if(min > values[x].cmin){
							min = values[x].cmin;
						}
						name[x] = values[x].cname;
					}
					var avg = (sum/total).toFixed(2);//平均工资，保留两位小数
					//返回数据的处理结果
					return {"count":total,"avg":avg,"sum":sum,"max":max,"min":min,"name":name}
				};`
			● 进行操作的整合
				`db.runCommand({
					"mapreduce":"emps",
					"map":sexMapFun,
					"reduce":sexReduceFun,
					"out":"t_sex_emp"
				})`
	聚合框架：虽然MapReduce功能强大，但是复杂度也大，所以MongoDB从2.x开始引入了聚合函数:aggregate
			注：在聚合框架里面使用每行的数据引用："$字段名称"
		`$group` : 分组操作，是无序操作，并且在内存中完成的，所以不肯能支持大数据量
		例1：实现聚合查询功能----求出每个职位的雇员人数
			`db.emps.aggregate({"$group":{"_id":"$job","job_count":{"$sum":1}}})`

		例2：求出每个职位的总工资
			`db.emps.aggregate([{"$group":{"_id":"$job","job_count":{"$sum":"$salary"}}}])`
		例3：计算每个职位的平均工资
			`db.emps.aggregate([{"$group":{"_id":"$job","job_count":{"$sum":1},"job_avg":{"$avg":"$salary"}}}])`
		例3：计算每个职位的最高、最低工资
			`db.emps.aggregate([{"$group":{"_id":"$job","max_sal":{"$max":"$salary"},"min_sal":{"$min":"$salary"}}}])`
		例4：求出每个职位的工资数据(数组显示)
			`db.emps.aggregate([{"$group":{"_id":"$job","sal_data":{"$push":"$salary"}}}])`
		例5：求出每个职位的人员
			`db.emps.aggregate([{"$group":{"_id":"$job","sal_data":{"$push":"$name"}}}])`
		例5：求出每个职位的人员(去除重复数据)
			`db.emps.aggregate([{"$group":{"_id":"$job","sal_data":{"$addToSet":"$name"}}}])`
		例6：取出第一个雇员
			`db.emps.aggregate([{"$group":{"_id":"$job","sal_data":{"$first":"$name"}}}])`
		例7：取出最后一个雇员
			`db.emps.aggregate([{"$group":{"_id":"$job","sal_data":{"$last":"$name"}}}])`

		`$project`:来控制数据列的显示规则，可执行的规则为如下：
			● 普通列 `（{成员 : 1|true}）`: 表示要显示的内容
			● `_id列` `（{"_id":0|false}）`: 表示`“_id”`列是否显示
			● 条件过滤列`（{成员 : 表达式}）`：满足表达式后的数据可以进行显示
			注：①只有设置的列才可以进行显示，其他列则不显示，实际上这就是数据库的投影机制
				②四则运算：加法`（“$and”）`、减法`（“$subtract”）`、乘法`（“$mutilply”`）、除法`（“$divide”）`
				③关系运算：大小比较`（“$cmp”）`，等于`（“$eq”）`，大于`（“$gt”）`，大于等于`（“$gte”）`，
						   小于`（“$lt”）`，小于等于`（“$lte”）`，不等于`（“$ne”）`，判断NULL`（“$ifNull”）`
				④逻辑运算：与（`$and`），或`（“$or）`，非`（“$not”）`
				⑤字符串操作：连接`（“$concat”）`，截取`（“$substr”）`，转小写`（“$toLower”）`，转大写`（“$toUpper”）`
							 大小写比较`（“$strcasecmp”）`
			例1：只显示name、job列，不显示_id、salary列
			`db.emps.aggregate({"$project":{"_id":0,"name":1}})  `
			例2：计算雇员年薪
			`db.emps.aggregate([{"$project":{"_id":0,"name":1,"job":1,"salary":{"年薪":{"$multiply":["$salary",12]}}}}])`
			例3：找出所有工资大于等于2000的雇员姓名、年龄、工资
			`db.emps.aggregate([{"$project":{
				"_id":0,
				"name":1,
				"age":1,
				"工资":"$salary",
				"salary":{"$gte":["$salary",2000]}
			}}])`
			
		例4：查找职位是MANAGER的雇员信息
		`db.emps.aggregate([{"$project":{
			"_id":0,
			"name":1,
			"age":1,
			"职位":"$job",
			"job":{"$eq":["$job","MANAGER"]}
		}}])
		db.emps.aggregate([{"$project":{
			"_id":0,
			"name":1,
			"age":1,
			"职位":"$job",
			"job":{"$eq":["$job",{"$toUpper":"manager"}]}
		}}])
		db.emps.aggregate([{"$project":{
			"_id":0,
			"name":1,
			"age":1,
			"职位":"$job",
			"job":{"$strcasecmp":["$job","manager"]}
		}}])`
		
		例5：查询职位信息的前三位
		
		`db.emps.aggregate([{"$project":{
			"_id":0,
			"name":1,
			"age":1,
			"职位":"$job",
			"job":{"$substr":["$job",0,3]}
		}}])`

		`$match`：用于过滤数据，只输出符合条件的文档。
			例：查询工资大于的2000小于8000 的雇员信息
				`db.emps.aggregate([{"$match":{"salary":{"$gte":2000,"$lte":8000}}}])`

		例2：进行管道操作，将$match得到的文档送入$group操作
			`db.emps.aggregate([{"$match":{"salary":{"$gte":2000,"$lte":8000}}},{"$project":{"_id":0,"name":1,"age":1,"salary":1}}])`

		总结：`$project`相当于SELECT字句
			  `$match`相当于WHERE字句
			  `$group`相当于GROUP BY操作

		`$sort`：实现排序操作，1表示升序，-1表示降序
			例1：按照工资进行升序排序查询
				`db.emps.aggregate([{"$sort":{"salary":1}}])`
			例2：将所有操作一起使用
				`db.emps.aggregate([{"$match":{"salary":{"$gte":1000,"$lte":10000}}},{"$project":{"_id":0,"name":1,"salary":1,"job":1}},{"$group":{"_id":"$job","count":{"$sum":1},"avg":{"$avg":"$salary"}}},{"$sort":{"count":-1}}])`
		分页处理：`$limit`:负责数据的取出个数
				  `$skip`:数据的跨过个数
			例1：查询3条
			`	db.emps.aggregate([{"$project":{"_id":0,"name":1,"age":1,"salary":1}},{"$limit":3}])`
			例2：跨过3条，取出4条
			`	db.emps.aggregate([{"$project":{"_id":0,"name":1,"age":1,"salary":1}},{"$skip":3},{"$limit":4}])`
			例3：综合运用
				`db.emps.aggregate([
					{"$match":
						{"salary":{"$gte":1000,"$lte":10000}}},
					{"$project":{"_id":0,"name":1,"salary":1,"job":1}},
					{"$group":{"_id":"$job","count":{"$sum":1},"avg":{"$avg":"$salary"}}},
					{"$sort":{"count":-1}},
					{"$skip":1},
					{"$limit":1}
				])`
		`$unwind`：在查询数据的时候经常会返回数组信息，但是数组不方便信息的浏览，所以提供不`“$unwind”`的方式可		   以将数组变为独立的字符串
			注：将数据的数据变为了单行的数据
			创建depts集合
			`db.depts.insert({"dep":"技术部","bus":["研发","生产","培训"]})`
			`db.depts.insert({"dep":"财务部","bus":["工资","税收"]})`
			例：将信息进行转化
				`db.depts.aggregate([
				{"$project":{"_id":0,"dep":true,"bus":true}},
				{"$unwind":"$bus"}])`


		`$geoNear`：可以得到附近的坐标点
			准备测试数据：
				`db.shop.drop()
				db.shop.insert({"loc":[10,10]})
				db.shop.insert({"loc":[11,10]})
				db.shop.insert({"loc":[11,11]})
				db.shop.insert({"loc":[13,12]})
				db.shop.insert({"loc":[50,60]})
				db.shop.insert({"loc":[90,95]})`
			例：查找点[11,12]附近的两个点
				`db.shop.aggregate([
					{"$geoNear":{
						"near":[11,11],
						"distanceField":"loc",
						"maxDistance":1,
						"num":2,
						"spherical":true
					}}
				])`

		`$out`：利用此操作可以将查询结果输出到到指定的集合里面
		例：将投影信息输出到集合emp_out
	    `db.emps.aggregate([
			{"$project":{
				"_id":0,
				"name":1,
				"age":1}},
			{"$out":"emp_out"}
		])`

 11. 深入操作
	- 固定集合：规定集合大小，如果要保存的内容长度超过了集合的长度，那么会采用LRU算法（最近最少使用原则）
			  将最早的的数据移除，从而保存新的数据
		创建固定集合：
			`db.createCollection("collName",{"capped":true,"size":1024,"max":5}) `			
				"capped"：表示一个固定集合
				"size"：表示集合所占的空间大小
				"max"：表示最多能有几条记录
		例：`db.createCollection("users",{"capped":true,"size":1024,"max":5}) 
			db.users.insert({"name":"A","age":12})
			db.users.insert({"name":"B","age":34})
			db.users.insert({"name":"C","age":64})
			db.users.insert({"name":"D","age":43})
			db.users.insert({"name":"E","age":53})`
			此时已经达到了集合的上限，那么会继续保存新的内容
			`db.users.insert({"name":"F","age":24})
			db.users.insert({"name":"G","age":66})`
			最近最少使用的数据会消失，新数据则会保存，这种操作和缓存机制是非常相似度的
	- GridFS：在MongoDB里面支持大数据的存储（如图片、视频、音乐等二进制数据文件），在CMD使用“mongofiles”命令		  来完成
		（1）利用命令进入文件所在的路径下
		（2）将文件保存到数据库里
			`	mongofiles --port=27017 put 文件名   //保存数据到fs.files集合中`

		注：在MongoDB里面有一个fs.files系统集合，这个集合默认保存在test数据库里面，

		（3）查看保存的文件
				`mongofiles --port=27017 list`
		（4）查看保存的信息
				`use test
				db.fs.files.find()`
		（5）删除文件
				mongofiles --port=27017 delete 文件名 

## 三、用户管理 ：（针对数据库） ##

- 创建用户 ：
`db.createUser({
	"user":"root",
	"pwd":"123456",
	"roles":[{"role":"readWrite","db":"zyx"}]
})`

- 修改密码 ：
	db.changeUserPassword("用户名","新密码")    
	注：要在不需要授权验证的情况下修改(noauth=true)

## 四、使用Java操作MongoDB ##
（1）基于Mongo-Java-2.x
	连接数据库
```
	/*
	 * 连接数据库
	 */
	public class MongoDemo_conn {
		
		public static void main(String[] args) throws Exception {
			MongoClient client=new MongoClient("localhost",27017);
			DB db = client.getDB("zyx");//连接数据库
			//进行数据库的用户名和密码的验证
			boolean authenticate = db.authenticate("root", "123456".toCharArray());
			if(authenticate){
				Set<String> names = db.getCollectionNames();
				for (String name : names) {
					System.out.println("集合名称:"+name);
				}
			}
			client.close();//关闭数据库连接
		}
	}	
```
测试代码：
```
	/**
	 * 基于Mongo-Java-2.x
	 */
	public class TestMongoDB {

		private MongoClient client=null;
		private DB db=null; 
		private boolean authenticate=false;
		
		@Test
		public void testDelete(){
			if(authenticate){
				DBCollection dept = db.getCollection("deptcol");
				BasicDBObject cond=new  BasicDBObject();//准备删除 的过滤条件
				cond.put("deptno", new BasicDBObject("$gte",1020).append("$lte", 1050));
				WriteResult remove = dept.remove(cond);
				System.out.println(remove);
			}
		}
		
		/*
		 * 更新多条
		 */
		@Test
		public void testUdpateMore(){
			if(authenticate){
				DBCollection dept = db.getCollection("deptcol");
				
				BasicDBObject condA=new BasicDBObject();//准备要设置的过滤条件
				condA.put("deptno", new BasicDBObject("$gte",1020).append("$lte", 1050));
				
				BasicDBObject condB=new BasicDBObject();//修改器的设置
				condB.put("$set", new BasicDBObject("dname","修改后的"));
				
				WriteResult result = dept.updateMulti(condA, condB);
				System.out.println(result.getN());
			}
		}
		
		/*
		 * 更新一条
		 */
		@Test
		public void testUdpateOne(){
			if(authenticate){
				DBCollection dept = db.getCollection("deptcol");
				
				BasicDBObject condA=new BasicDBObject();//准备要设置的过滤条件
				condA.put("deptno", 1000);
				
				BasicDBObject condB=new BasicDBObject();//修改器的设置
				condB.put("$set", new BasicDBObject("dname","修改后的"));
				
				WriteResult result = dept.update(condA, condB);
				System.out.println(result);
			}
		}
		
		/*
		 * 模糊查询
		 */
		@Test
		public void testFuzzyQuery(){
			if(authenticate){
				DBCollection dept = db.getCollection("deptcol");
				BasicDBObject cond=new BasicDBObject();//准备设置查询过滤条件
				Pattern pattern=Pattern.compile("5"); 
				cond.put("dname", new BasicDBObject("$regex",pattern));
				
				DBCursor cursor=dept.find(cond);//得到全部数据
				while(cursor.hasNext()){
					DBObject doc = cursor.next();//得到每一行数据
					System.out.println("部门编号:"+doc.get("deptno")+" , 部门名称:"+doc.get("dname")+" , 部门地点:"+doc.get("loc"));
				}
			}
		}
		
		/*
		 * 设置范围查询---in
		 */
		@Test
		public void testRangeQuery(){
			if(authenticate){
				DBCollection dept = db.getCollection("deptcol");
				BasicDBObject cond=new BasicDBObject();//准备设置查询过滤条件
				
				cond.put("deptno", new BasicDBObject("$in",new int[]{1021,1053,1029}));
				
				DBCursor cursor=dept.find(cond);//得到全部数据
				while(cursor.hasNext()){
					DBObject doc = cursor.next();//得到每一行数据
					System.out.println("部门编号:"+doc.get("deptno")+" , 部门名称:"+doc.get("dname")+" , 部门地点:"+doc.get("loc"));
				}
			}
		}
		
		/*
		 * 带条件查询
		 */
		@Test
		public void testQueryWithCond(){
			if(authenticate){
				DBCollection dept = db.getCollection("deptcol");
				BasicDBObject cond=new BasicDBObject();//准备设置查询过滤条件
				//设置查询条件  deptno在1020到1050范围内
				cond.put("deptno", new BasicDBObject("$gte",1020).append("$lte", 1050));
				
				DBCursor cursor=dept.find(cond);//得到全部数据
				while(cursor.hasNext()){
					DBObject doc = cursor.next();//得到每一行数据
					System.out.println("部门编号:"+doc.get("deptno")+" , 部门名称:"+doc.get("dname")+" , 部门地点:"+doc.get("loc"));
				}
			}
		}
		
		/*
		 * 分页查询
		 */
		@Test
		public void testPage(){
			if(authenticate){
				DBCollection dept = db.getCollection("deptcol");
				DBCursor cursor=dept.find().skip(0).limit(20);//分页处理
				while(cursor.hasNext()){
					DBObject doc = cursor.next();//得到每一行数据
					System.out.println("部门编号:"+doc.get("deptno")+" , 部门名称:"+doc.get("dname")+" , 部门地点:"+doc.get("loc"));
				}
			}
		}
		
		/*
		 * 读取数据
		 */
		@Test
		public void testRead(){
			if(authenticate){
				DBCollection dept = db.getCollection("deptcol");
				DBCursor cursor=dept.find();//得到全部数据
				while(cursor.hasNext()){
					DBObject doc = cursor.next();//得到每一行数据
					System.out.println("部门编号:"+doc.get("deptno")+" , 部门名称:"+doc.get("dname")+" , 部门地点:"+doc.get("loc"));
				}
			}
		}
		
		/*
		 * 插入数据
		 */
		@Test
		public void testInsert(){
			if(authenticate){
				DBCollection dept = db.getCollection("deptcol");
				for(int x=0;x<1000;x++){
					BasicDBObject doc=new BasicDBObject();
					doc.append("deptno", 1000+x);
					doc.append("dname", "技术部-"+x);
					doc.append("loc", "临潼-"+x);
					dept.insert(doc);//保存数据
				}
			}
		}
		
		@Before
		public void connection() throws Exception{
			client=new MongoClient("localhost",27017);
			db = client.getDB("zyx");//连接数据库
			//进行数据库的用户名和密码的验证
			authenticate = db.authenticate("root", "123456".toCharArray());
		}
		
		@After
		public void close(){
			client.close();//关闭数据库连接
		}
		
	}
```
（2）基于Mongo-Java-3.x
	连接数据库
```
	public class MongoDB_conn {

		public static void main(String[] args) {
			MongoClientURI uri=new MongoClientURI("mongodb://root:123456@localhost:27017/zyx");
			MongoClient client=new MongoClient(uri);
			MongoDatabase database = client.getDatabase("zyx");
			client.close();
		}
		
	}
```
	（3）测试代码
```
	public class TestMongoDB {

		private MongoClientURI uri=null;
		private MongoClient client=null;
		private MongoDatabase database =null;
		
		/*
		 *统计查询,根据job分组，统计平均工资
		 *
		 *	db.emps.aggregate([
		 *		{"$group":{
		 *					"_id":"$job",
		 *					"job_count":{"$sum":1},
		 *					"job_avg":{"$avg":"$salary"}
		 *				}
		 *		}
		 *	])
		 */
		@Test
		public void testAggregate(){
			MongoCollection<Document> emps = database.getCollection("emps");
			List<BasicDBObject> list=new ArrayList<BasicDBObject>();
			BasicDBObject cond=new BasicDBObject("$group",new BasicDBObject("_id","$job").append("job_count", new BasicDBObject("$sum",1)).append("job_avg", new BasicDBObject("$avg","$salary")));
			list.add(cond);
			MongoCursor<Document> cursor = emps.aggregate(list).iterator();
			while (cursor.hasNext()) {
				Document doc = (Document) cursor.next();
				System.out.println("职位:"+doc.getString("_id")+",人数:"+doc.getInteger("job_count")+",平均工资:"+doc.getDouble("job_avg"));
			}
		}
		
		/*
		 *删除数据
		 */
		@Test
		public void testDelete(){
			MongoCollection<Document> student = database.getCollection("student");
			BasicDBObject cond=new BasicDBObject("sid",0);//删除条件
			DeleteResult result = student.deleteOne(cond);
			System.out.println(result);
		
		}
		
		/*
		 *更新操作
		 */
		@Test
		public void testUpdate(){
			MongoCollection<Document> student = database.getCollection("student");
			BasicDBObject condA=new BasicDBObject("sid",0);//查询条件
			BasicDBObject condB=new BasicDBObject("$set",new BasicDBObject("sname","zyx"));
			UpdateResult result = student.updateOne(condA, condB);
			System.out.println(result);
		
		}
		
		/*
		 *分页查询 
		 */
		@Test
		public void testQueryPage(){
			MongoCollection<Document> student = database.getCollection("student");
			BasicDBObject cond=new BasicDBObject();
			Pattern pattern=Pattern.compile("5");
			cond.put("sname", new BasicDBObject("$regex",pattern).append("$options", "i"));
			
			MongoCursor<Document> cursor = student.find(cond).skip(5).limit(5).iterator();
			while (cursor.hasNext()) {
				Document doc = (Document) cursor.next();
				System.out.println("学号:"+doc.get("sid")+",姓名:"+doc.get("sname")+",性别:"+doc.get("sex"));
				//System.out.println(doc);
			}
		}
		
		
		/*
		 *模糊查询 
		 */
		@Test
		public void testFuzzyQuery(){
			MongoCollection<Document> student = database.getCollection("student");
			BasicDBObject cond=new BasicDBObject();
			Pattern pattern=Pattern.compile("5");
			cond.put("sname", new BasicDBObject("$regex",pattern).append("$options", "i"));
			
			MongoCursor<Document> cursor = student.find(cond).iterator();
			while (cursor.hasNext()) {
				Document doc = (Document) cursor.next();
				System.out.println("学号:"+doc.get("sid")+",姓名:"+doc.get("sname")+",性别:"+doc.get("sex"));
				//System.out.println(doc);
			}
		}
		
		/*
		 *范围查询 
		 */
		@Test
		public void testRangeQuery(){
			MongoCollection<Document> student = database.getCollection("student");
			BasicDBObject cond=new BasicDBObject();
			cond.put("sid", new BasicDBObject("$gt",10).append("$lt", 30));
			
			MongoCursor<Document> cursor = student.find(cond).iterator();
			while (cursor.hasNext()) {
				Document doc = (Document) cursor.next();
				System.out.println("学号:"+doc.get("sid")+",姓名:"+doc.get("sname")+",性别:"+doc.get("sex"));
				//System.out.println(doc);
			}
		}
		
		/*
		 *数据查询 
		 */
		@Test
		public void testQuery(){
			MongoCollection<Document> student = database.getCollection("student");
			MongoCursor<Document> cursor = student.find().iterator();
			while (cursor.hasNext()) {
				Document doc = (Document) cursor.next();
				System.out.println("学号:"+doc.get("sid")+",姓名:"+doc.get("sname")+",性别:"+doc.get("sex"));
				//System.out.println(doc);
			}
		}
		
		/*
		 * 数据插入
		 */
		@Test
		public void testInsert(){
			MongoCollection<Document> student = database.getCollection("student");
			
			List<Document> documents=new ArrayList<Document>(); 
			for(int x=0;x<100;x++){
				Document doc=new Document();
				doc.put("sid", x);
				doc.put("sname", "姓名-"+x);
				doc.put("sex", x%2==0?"男":"女");
				documents.add(doc);
			}
			student.insertMany(documents);
		}
		
		@Before
		public void connection() throws Exception{
			uri=new MongoClientURI("mongodb://root:123456@localhost:27017/zyx");
			client=new MongoClient(uri);
			database = client.getDatabase("zyx");
		}
		
		@After
		public void close(){
			client.close();//关闭数据库连接
		}
		
	}
```

