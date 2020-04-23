---
layout: post
title:  "Redirect & Interceptor"
name: spring_db.md
category: spring
---

JDBC
=======

- [JDBC of JSP](https://21lva.github.io/gbWeb/posts/jsp/jsp_db.html)
- [참고자료](https://preamtree.tistory.com/88)
- [참고자료2](https://gmlwjd9405.github.io/2018/12/19/jdbctemplate-usage.html)
- @Repository : XML 설정없이도 DAO와 리파지토리를 찾고 설정이 가능하게 해줌

JDBC template
---------

- JDBC의 load, connection, query 등의 작업을 spring이 자동으로 하게 해주는 기능
- JdbcTemplate <----> SQL 작성 및 전송

- bean 설정

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driver}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>
```
- 활용
```java
public class myDao {
    private JdbcTemplate jdbcTemplate;
    @AutoWired
    public myDao (DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void execute(String sql){
      //DDL에 사용
      this.jdbcTemplate.execute(sql);
    }

    //update : delete, insert, update에 사용
    public int update(String searchName,String tgtName){
      this.jdbcTemplate.update("UPDATE tbl1 set name=? where name=?",new Object[]{tgtName,searchName});

    }
    public int insert(String name){
      this.jdbcTemplate.update("INSERT INTO tbl1 (name) values(?)",new Object[]{name});
    }
    public int delete(String name){
      this.jdbcTemplate.update("DELETE FROM tbl1 where name=?",new Object[]{name});
    }

    //query
    public List<Ball> findByName(String name){
      String sql = "SELECT id,name FROM tbl1"
      return this.jdbcTemplate.query(sql,
        (row,rowNum)->new Ball(row.getLong("id"),row.getString("name")),
      name);
    }
}
```

- query 메서드들
  - List<T> query(String sql, RowMapper<T> rowMapper)
  - List<T> query(String sql, Object [] args, RowMapper<T> rowMapper)
  - List<T> query(String sql, RowMapper<T> rowMapper, Object... args)
