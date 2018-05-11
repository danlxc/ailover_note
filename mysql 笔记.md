mysql 笔记

##### 模糊查询 有索引
`F_INFO_WINDCODE LIKE #{targetfundcode}"%"`

##### 批量插入
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