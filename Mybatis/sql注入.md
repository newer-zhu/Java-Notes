## 如何防止sql注入

**1. 代码层防止sql注入攻击的最佳方案就是sql预编译**

```java
public List<Course> orderList(String studentId){
    String sql = "select id,course_id,student_id,status from course where student_id = ?";
    return jdbcTemplate.query(sql,new Object[]{studentId},new BeanPropertyRowMapper(Course.class));
}
```

这样我们传进来的参数 `4 or 1 = 1`就会被当作是一个`student_id`，所以就不会出现sql注入了。

**2. 确认每种数据的类型，比如是数字，数据库则必须使用int类型来存储**

**3. 规定数据长度，能在一定程度上防止sql注入**

**4. 严格限制数据库权限，能最大程度减少sql注入的危害**

**5. 避免直接响应一些sql异常信息，sql发生异常后，自定义异常进行响应**

**6. 过滤参数中含有的一些数据库关键词**