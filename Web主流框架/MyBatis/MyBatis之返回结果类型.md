# 1. resultMap和resultType区别
- MyBatis的每一个查询映射的返回类型都是ResultMap
- 当我们提供的返回类型属性是resultType的时候，MyBatis对自动的给我们把对应的值赋给resultType所指定对象的属性
- 当我们提供的返回类型是resultMap的时候，将数据库中列数据复制到对象的相应属性上，可以用于复制查询，两者不能同时用。

# 2. resultType

- 返回单个实例

```
<select id="selectUser" parameterType="int" resultType="User">
select * from user where id = #{id}
</select>
```
- 返回List集合

```
<select id="selectUserAll" resultType="User" > <!-- resultMap="userMap" -->
select * from user
</select>
```
 - resultType：适合使用返回值得数据类型是非自定义的，即jdk的提供的类型
# 3. resultMap

- 简单查询：

```
<resultMap type="User" id="userMap">
<id column="id" property="id"/>
<result column="name" property="name"/>
</resultMap>
column:数据库中列名称，property:类中属性名称
```

- resultMap：适合使用返回值是自定义实体类的情况

- resultMap ：映射实体类的数据类型，resultMap的唯一标识
