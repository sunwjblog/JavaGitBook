# Mybatis遇到的问题

#### 根据时间范围查询记录未查到的情况

根据时间查询数据时，传入的是Date类型的参数时，查询不到数据，此情况与SQL的字符串拼接或日期的格式转换存在关系。

在写sql的时候，需要注意时间字段的特殊处理。

问题sql：

```xml
 <select id="getTfpayBaddebtOrdersByTransTime" resultMap="BaseResultMap">

    select
    <include refid="Base_Column_List" />
    from TF_PAYBADDEBT_ORDERS
    <where>
      <if test="startTime!=null">
         TRANSTIME &gt;= #{startTime,jdbcType=TIMESTAMP}
      </if>
      <if test="endTime!=null">
        and TRANSTIME &lt;= #{endTime,jdbcType=TIMESTAMP}
      </if>
    </where>

  </select>
```

修改后：

```xml
<select id="getTfpayBaddebtOrdersByTransTime" resultMap="BaseResultMap">

    select
    <include refid="Base_Column_List" />
    from TF_PAYBADDEBT_ORDERS
    <where>
      <if test="startTime!=null">
        <![CDATA[ TRANSTIME >= #{startTime,jdbcType=TIMESTAMP} ]]>
      </if>
      <if test="endTime!=null">
        <![CDATA[ and TRANSTIME >= #{endTime,jdbcType=TIMESTAMP} ]]>
      </if>
    </where>

  </select>
```

注意：使用> = <符号，需要使用 <![CDATA[ 文本 ]] 将内容包围以告诉XML解析时不对文本进行解析。

