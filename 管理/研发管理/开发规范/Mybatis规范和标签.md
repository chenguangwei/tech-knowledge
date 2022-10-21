# Mybatis规范和标签

> 本文主要是介绍 Mybatis规范和标签 。

- [一文搞懂mybatis规范和标签](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#%E4%B8%80%E6%96%87%E6%90%9E%E6%87%82mybatis%E8%A7%84%E8%8C%83%E5%92%8C%E6%A0%87%E7%AD%BE)
  - [resultMap](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#resultmap)
  - [if标签](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#if%E6%A0%87%E7%AD%BE)
  - [where标签](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#where%E6%A0%87%E7%AD%BE)
  - [choose标签](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#choose%E6%A0%87%E7%AD%BE)
  - [foreach标签遍历数组](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#foreach%E6%A0%87%E7%AD%BE%E9%81%8D%E5%8E%86%E6%95%B0%E7%BB%84)
- [参考文章](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0)

## [#](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#%E4%B8%80%E6%96%87%E6%90%9E%E6%87%82mybatis%E8%A7%84%E8%8C%83%E5%92%8C%E6%A0%87%E7%AD%BE)一文搞懂mybatis规范和标签

### [#](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#resultmap)resultMap

resultType实际上是将数据库表字段与JavaBean中的属性建立映射关系，在编写SQL的时候使用resultMap，mybatis会依据resultMap中的映射关系将数据库返回的记录映射到对应的字段上。

修改TravelRouteMapper.xml增加一个resultMap

```
 <resultMap type="TravelRoute" id="TravelRouteType" >

      <result column = "travel_route_id"property="travelRouteId"/>
      <result column = "travel_route_name"property="travelRouteName"/>
      <result column = "travel_route_price"property="travelRoutePrice"/>
      <result column = "travel_route_introduce"property="travelRouteIntroduce"/>
      <result column = "travel_route_flag"property="travelRouteFlag"/>
      <result column = "travel_route_date"property="travelRouteDate"/>
      <result column = "isThemeTour"property="isThemeTour"/>
      <result column = "travel_route_count"property="travelRouteCount"/>
      <result column = "travel_route_cid"property="travelRouteCid"/>
      <result column = "travel_route_image"property="travelRouteImage"/>
      <result column ="travel_route_seller_id"property="travelRouteSellerId"/>
</resultMap>
```

我们将之前使用as 语法的queryTravelByPage和queryTravelById的as xxx去掉,将resultType改为TravelRoute。留下一个queryTravelByName当作复习语法的例子就好，就不赶尽杀绝了。

```
  <select id="queryTravelByPage"resultMap="TravelRouteType" parameterType="java.util.Map">

     select
       travel_route_id ,
       travel_route_name ,
       travel_route_price ,
       travel_route_introduce ,
       travel_route_flag ,
       travel_route_date ,
       isThemeTour ,
       travel_route_count ,
       travel_route_cid ,
       travel_route_image ,
       travel_route_seller_id

      from   travel_route order by travel_route_id desc limit #{startRow},#{endRow}

    </select>

     <select id="queryTravelById"resultMap="TravelRouteType" parameterType="Long">

     select
       travel_route_id ,
       travel_route_name ,
       travel_route_price ,
       travel_route_introduce ,
       travel_route_flag ,
       travel_route_date ,
       isThemeTour ,
       travel_route_count,
       travel_route_cid ,
       travel_route_image ,
       travel_route_seller_id 

      from   travel_route where travel_route_id =#{travelRouteId}

    </select>
```

运行测试程序，保证代码修改正确！

之前编写的例子中，我们都自己实现了dao接口，其实mybatis提供了动态代理的方式，无需由开发人员编写实现类。使用mapper的动态代理，可以简化冗余代码的编写，也是实际开发中推荐的写法。接下来，我们就一起来看下怎样使用动态代理。

1.在TravelRouteMapper.xml中的mapper标签中使用namespace属性：

```
<mapper namespace="com.pz.route.dao.TravelRouteDao">
```

这样编写mybatis mapper文件与具体的接口com.pz.route.dao.TravelRouteDao建立映射关系。

2.编写的Dao必须遵守一个命名规范——方法名和mapper文件的id值保持一一对应的关系。

3.使用sqlSession创建dao的代理对象执行数据库操作：

```
@Test
       public void testQueryTravelByIdProxy(){
              SqlSession sqlSession=null;
              try{
               sqlSession=MyBatisUtil.getSqlSession();

              TravelRouteDaotravelRouteDao=sqlSession.getMapper(TravelRouteDao.class);

              TravelRoute travelRoute=travelRouteDao.queryTravelById(515L);
              sqlSession.commit();
              System.out.println(travelRoute);
              }catch(Exception e){
                     e.printStackTrace();
              }finally{
                     if(null!=sqlSession){
                            sqlSession.close();
                     }
              }

       }
```

通过SqlSession对象的getMapper方法，将要获取的dao接口的class属性传入，会返回MyBatis会使用动态代理的方式创建对象，该对象是有jdk的动态代理自动生成的。由于我们的Sqlsession默认的是手动提交的事务方式，所以我们还是需要在程序中commit。不过，将dao的实现类去除掉之后，代码更加简洁。

我们之前编写的updateById方法其实存在一个问题——如果传入的数据是null或者是空字符串，对应的字段也会被设置为null或者是空字符串，其实这样提供方法比较危险，在实际开发的过程中，往往希望只对传入的数据做更新。还有一个场景，我们上网的时候发现页面上有多个组合查询条件，输入某一个查询条件就匹配一项查询结果，输入多个查询条件，就要求查询结果匹配多个查询条件。针对这样的场景，mybatis提供了动态sql技术来减少开发人员的开发工作。

### [#](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#if%E6%A0%87%E7%AD%BE)if标签

我们使用if标签来解决数据更新的控制问题——对于一些字段，必须有值传入才做更新:

```
<update id="updateById">
      update   
      travel_route
       set
       <if test="travelRouteName!= null and travelRouteName != ''"> 
       travel_route_name=#{travelRouteName},
       </if>
        <if test="travelRoutePrice!= null and travelRoutePrice >0 "> 
       travel_route_price=#{travelRoutePrice},
       </if>
        <if test="travelRouteIntroduce!= null and travelRouteIntroduce != ''"> 
      travel_route_introduce=#{travelRouteIntroduce},
       </if>
        <if test="travelRouteFlag!= null "> 
       travel_route_flag=#{travelRouteFlag},
       </if>
       <if test="travelRouteFlag!= null "> 
       travel_route_date=#{travelRouteDate},
       </if>
        <if test="travelRouteFlag!= null "> 
       isThemeTour=#{isThemeTour},
       </if>
        <if test="travelRouteCount!= null and travelRouteCount > 0 "> 
       travel_route_count=#{travelRouteCount},
       </if>
        <if test="travelRouteCid!= null and travelRouteCid > 0 "> 
       travel_route_cid=#{travelRouteCid},
       </if>
        <if test="travelRouteImage!= null and travelRouteImage != ''"> 
       travel_route_image=#{travelRouteImage},
       </if>
        <if test="travelRouteSellerId!= null and travelRouteSellerId >0 "> 
      travel_route_seller_id=#{travelRouteSellerId}
       </if>
      where travel_route_id =#{travelRouteId}


</update>
```

运行测试用例，确保代码没有问题！

### [#](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#where%E6%A0%87%E7%AD%BE)where标签

```
 我们来实现一个新的功能，实现复合查询：如果输入了线路名称，我们就按线路名称模糊查询，如果输入了价格我们就查询大于输入价格的线路。我们可以通过where标签来实现它。
```

在mapper中编写sql：

```
<select id="queryTravelByQuery"resultMap="TravelRouteType" parameterType="TravelRoute">

     select
       travel_route_id ,
       travel_route_name ,
       travel_route_price ,
       travel_route_introduce ,
       travel_route_flag ,
       travel_route_date ,
       isThemeTour ,
       travel_route_count,
       travel_route_cid ,
       travel_route_image ,
       travel_route_seller_id 

      from travel_route
       <where>
          <if test="travelRouteName!= null and travelRouteName != ''"> 
          and travel_route_name like  '%' #{travelRouteName} '%'
         </if>
         <if test="travelRoutePrice!= null and travelRoutePrice>0 "> 
          andtravel_route_price>#{travelRoutePrice}
         </if>
        </where>

    </select>
编写方法接口：/
     * 模糊查询TravelRoute列表
     * @param travelRoute
     * @return
     */
List<TravelRoute>queryTravelByQuery(TravelRoute travelRoute);
```

由于项目为了展示之前的例子，解决编译问题 实现类实现一个空方法就好，使用动态代理的方式是无需编写实现类的：

```
@Override
       public List<TravelRoute>queryTravelByQuery(TravelRoute travelRoute) {
              // TODO Auto-generated method stub
              returnnull;
       }
编写测试方法：
@Test
       publicvoid testQueryTravelRouteByQuerydProxy(){
              SqlSession sqlSession=null;
              try{
               sqlSession=MyBatisUtil.getSqlSession();
              TravelRouteDaotravelRouteDao=(TravelRouteDao) sqlSession.getMapper(TravelRouteDao.class);
              TravelRoute travelRoute = new TravelRoute();
              travelRoute.setTravelRouteName("春节");
              travelRoute.setTravelRoutePrice(100d);
              travelRoute.setTravelRouteDate("2019-10-26");
              List<TravelRoute> list=travelRouteDao.queryTravelByQuery(travelRoute);
              System.out.println(list);
              }catch(Exception e){
                     e.printStackTrace();
              }finally{
                     if(null!=sqlSession){
                            sqlSession.close();
                     }
              }

       }
```

运行测试用例，看看效果！

### [#](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#choose%E6%A0%87%E7%AD%BE)choose标签

```
       我们再提一个小要求，如果用户没有传入travelRouteName也没有传入travelRoutePrice,程序就不返回数据。当然我们可以在程序中做出判断，不过mybatis提供了choose标签也可以实现上述要求。在mapper中编写sql：
```

```
  <select id="queryTravelByChooseQuery"resultMap="TravelRouteType" parameterType="TravelRoute">

     select
       travel_route_id ,
       travel_route_name ,
       travel_route_price ,
       travel_route_introduce ,
       travel_route_flag ,
       travel_route_date ,
       isThemeTour ,
       travel_route_count,
       travel_route_cid ,
       travel_route_image ,
       travel_route_seller_id 

      from travel_route
       <where>
         <choose>
          <when test="travelRouteName!= null and travelRouteName != ''"> 
          and travel_route_name like  '%' #{travelRouteName} '%'
         </when>
         <when test="travelRoutePrice!= null and travelRoutePrice >0 "> 
          andtravel_route_price>#{travelRoutePrice}
         </when>
         <otherwise>
              1 >2
          </otherwise>
        </choose>

        </where>

    </select>
```

编写dao接口和实现类

```
/
     * 模糊查询TravelRoute列表
     * @param travelRoute
     * @return
     */
   List<TravelRoute> queryTravelByChooseQuery(TravelRoutetravelRoute);
编写测试方法
@Test
       publicvoid testQueryTravelRouteByChooseQuerydProxy(){
              SqlSession sqlSession=null;
              try{
               sqlSession=MyBatisUtil.getSqlSession();
              TravelRouteDao travelRouteDao=(TravelRouteDao)sqlSession.getMapper(TravelRouteDao.class);
              TravelRoute travelRoute = new TravelRoute();
              List<TravelRoute> list=travelRouteDao.queryTravelByChooseQuery(travelRoute);
              System.out.println(list);
              }catch(Exception e){
                     e.printStackTrace();
              }finally{
                     if(null!=sqlSession){
                            sqlSession.close();
                     }
              }

       }
```

```
      运行程序，我们发现返回了一个空的list.
```

choose标签有多个when子标签，但是只能有一个otherwise子标签，有点类似java中的switch语句，我们在子标签中使用了1>2这种错误条件，在sql层面就屏蔽了数据返回的可能性。

### [#](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#foreach%E6%A0%87%E7%AD%BE%E9%81%8D%E5%8E%86%E6%95%B0%E7%BB%84)foreach标签遍历数组

```
  我们再来完成一个小功能，传入一组id号，批量放回线路数据。我们需要使用到sql语句中的in语句，当然，也可以用拼接字符串的方式实现它，但是mybatis提供了foreach标签，实现了循环拼接的功能。

 foreach标签的属性
```

§ collection 表示要遍历的集合类型，可以传入List和数组。

§ open、close、separator 用于 SQL 拼接，open表示由什么字符开始，colse表示由什么字符结束，separator表示拼接字符串的分隔符。

我们来编写一个例子使用下foreach标签，在mapper文件中编写下面内容：

```
<select id="queryTravelByForEach"resultMap="TravelRouteType" >

     select
       travel_route_id ,
       travel_route_name ,
       travel_route_price ,
       travel_route_introduce ,
       travel_route_flag ,
       travel_route_date ,
       isThemeTour ,
       travel_route_count,
       travel_route_cid ,
       travel_route_image ,
       travel_route_seller_id 

      from travel_route
       <where>
         <choose>
       <when test="list != nulland list.size>0">
        travel_route_id IN
        <foreach collection="list"open="(" close=")"item="id" separator=",">
            #{id}
        </foreach>
        </when>
          <otherwise>
              1 >2
          </otherwise>
        </choose>
       </where>

    </select>
```

编写dao接口方法和接口实现：

```
 /
     * 根据travelRouteIdList返回列表
     * @param travelRouteIdList
     * @return
     */
   List<TravelRoute> queryTravelByForEach(List<Long>travelRouteIdList);
编写测试方法：
@Test
       publicvoid testQueryTravelRouteByForEachProxy(){
              List<Long> idList= Arrays.asList(1L,2L,3L);
              SqlSession sqlSession=null;
              try{
               sqlSession=MyBatisUtil.getSqlSession();
              TravelRouteDaotravelRouteDao=(TravelRouteDao) sqlSession.getMapper(TravelRouteDao.class);
              TravelRoute travelRoute = new TravelRoute();
              List<TravelRoute> list=travelRouteDao.queryTravelByForEach(idList);
              System.out.println(list);
              }catch(Exception e){
                     e.printStackTrace();
              }finally{
                     if(null!=sqlSession){
                            sqlSession.close();
                     }
              }

       }
```

sql标签

```
我们发现编写select语句时需要每次都写很多字段名，每写一条语句就需要写一次，这样很累，当然你要是不怕dba地狱的怒吼和你的Leader的天堂的神罚的话，可以写select * 来解决。Mybatis提供sql标签来解决sql的复用问题，使用sql标签可以把公共的sql语句片段提取出来，在需要使用的时候写上include标签就可以引入公用的sql语句片段了。我们将公共的查询字段使用sql标签提取出来：
```

```
  <sql id = "commonselect">

     travel_route_id ,
       travel_route_name ,
       travel_route_price ,
       travel_route_introduce ,
       travel_route_flag ,
       travel_route_date ,
       isThemeTour ,
       travel_route_count,
       travel_route_cid ,
       travel_route_image ,
       travel_route_seller_id 

   </sql>
```

为了验证一下，我们修改下之前的queryTravelByForEach语句

```
<select id="queryTravelByForEach"resultMap="TravelRouteType" >

     select
        <include refid="commonselect"/>

      from travel_route
       <where>
         <choose>
       <when test="list != nulland list.size>0">
        travel_route_id IN
        <foreach collection="list"open="(" close=")"item="id" separator=",">
            #{id}
        </foreach>
        </when>
          <otherwise>
              1 >2
          </otherwise>
        </choose>
       </where>

</select>
```

运行下测试用例，看看效果

注意噢，由于mybatis使用的时xml,在xml中有些符号，是不能直接使用的,遇到这种情况，我们可以使用实体符号代替。下面就是一些数学符号和实体符号之间的关系。

原符号 实体符号

< <

<= <=

> > 

> = >=

& &

" "

' '

```
 这里要强调一下<,>=,<=,&,一定不能直接在xml中出现。
```

我们修改下queryTravelByQuery的程序要求，要求返回大于等于输入价格的线路，示例：

```
<select id="queryTravelByQuery"resultMap="TravelRouteType" parameterType="TravelRoute">

     select
       travel_route_id ,
       travel_route_name ,
       travel_route_price ,
       travel_route_introduce ,
       travel_route_flag ,
       travel_route_date ,
       isThemeTour ,
       travel_route_count,
       travel_route_cid ,
       travel_route_image ,
       travel_route_seller_id 

      from travel_route
       <where>
          <if test="travelRouteName!= null and travelRouteName != ''"> 
          and travel_route_name like  '%' #{travelRouteName} '%'
         </if>
         <if test="travelRoutePrice!= null and travelRoutePrice >0 "> 
          and travel_route_price>=#{travelRoutePrice}
         </if>
        </where>

    </select>
```

除了使用特殊符号之外，我们还可以将特殊符号放到里面，这里面的内容xml是不会转义的。

```
<select id="queryTravelByQuery"resultMap="TravelRouteType" parameterType="TravelRoute">

     select
       travel_route_id ,
       travel_route_name ,
       travel_route_price ,
       travel_route_introduce ,
       travel_route_flag ,
       travel_route_date ,
       isThemeTour ,
       travel_route_count,
       travel_route_cid ,
       travel_route_image ,
       travel_route_seller_id 

      from travel_route
       <where>
          <if test="travelRouteName!= null and travelRouteName != ''"> 
          and travel_route_name like  '%' #{travelRouteName} '%'
         </if>
         <if test="travelRoutePrice!= null and travelRoutePrice >0 "> 
          and <![CDATA[travel_route_price >= #{travelRoutePrice}]]>
         </if>
        </where>

    </select>
```

本文分享自微信公众号 - 猿人工厂（gh_deca5a88e287），作者：山旮旯的胖子

原文出处及转载信息见文内详细说明，如有侵权，请联系 yunjia_community@tencent.com 删除。

原始发表时间：2020-06-05

本文参与[腾讯云自媒体分享计划 (opens new window)](https://cloud.tencent.com/developer/support-plan)，欢迎正在阅读的你也加入，一起分享。

想学习

## [#](https://www.yijiyong.com/dev/standard/03-mybatisstandard.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0)参考文章

- https://cloud.tencent.com/developer/article/1670067
