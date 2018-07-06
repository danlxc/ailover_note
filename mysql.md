## 数据去重,group\_concat,DISTINCT

```
SELECT
    *, group_concat(DISTINCT user_id)
FROM
    cjqfweb_fund_castsurely_scene
GROUP BY
    user_id
```

##### 

## 模糊查询 有索引

`F_INFO_WINDCODE LIKE #{targetfundcode}"%"`

## 批量插入

```
<update id="updateBatchByList" parameterType="java.util.List"  >
    <foreach collection="list" separator=";" item="item">
      UPDATE
          cjqfweb_smart_beta_proportion_everytime
      SET
          position_proportion = #{item.positionProportion}
      WHERE
          fundcode = #{item.fundcode}
          AND transfer_date > #{item.transferDate}
    </foreach>
</update>
```

## 获得奇偶数

```
获取偶数的方法

select * from pos_info_report_tmp_20110712 r

where mod(r.id,2) = 0;

获取奇数的方法

select * from pos_info_report_tmp_20110712 r

where mod(r.id,2) = 1;
```



