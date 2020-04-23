---
layout: post
title:  "DB"
cover:  "/assets/instacode.png"
name: jsp_db.md
---

JDBC
===

- JAVA에서 DB를 사용하기 위한 인터페이스. RDBMS에 접근하고 SQL문을 실행하는 방법을 제공
- **java.sql** 에 구현되어 있음. 여러 DB에 이 API를 사용하여 접근
- 이클립스를 사용한다면 등록해야 함

- JDBC 사용순서
  1. import java.sql.* ;
  2. 드라이버 load
  3. **Connection** 객체를 이용하여 DB와 연결
  4. **Statement** 객체를 이용하여 query 수행(;는 빼고 쓴다)
  5. **ResultSet** 객체를 생성하여 query 결과 저장
  6. 후 처리하고 JDBC연결을 끊기 위하여 **close()** 메서드 실행

#### SELECT

```java
import java.sql.*;

public class SelectTest {
    public static void main(String[] args) {
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;

        try{
            //드라이버 로드
            Class.forName("com.mysql.jdbc.Driver");

            //연결
            String url = "jdbc:mysql://localhost/dev";
            conn = DriverManager.getConnection(url, "id", "passwd");

            // Statement 객체 생성
            stmt = conn.createStatement();

            //쿼리 수행
            String sql = "SELECT name, owner, date_format(birth, '%Y년%m월%d일' date FROM pet";
            rs = stmt.executeQuery(sql);

            //실행결과 출력하기
            while(rs.next()){
                // 레코드의 칼럼은 배열과 달리 0부터 시작하지 않고 1부터 시작
                String name = rs.getString(1);
                String owner = rs.getString(2);
                String date = rs.getString(3);

                System.out.println(name + " " + owner + " " + date);
            }
        }
        catch( ClassNotFoundException e){
            System.out.println("Fail : loading a driver");
        }
        catch( SQLException e){
            System.out.println("Error : " + e);
        }
        finally{
            try{
              //역순으로
                if( stmt != null && !stmt.isClosed()){
                    stmt.close();
                }
                if( conn != null && !conn.isClosed()){
                    conn.close();
                }
                if( rs != null && !rs.isClosed()){
                    rs.close();
                }
            }
            catch( SQLException e){
                e.printStackTrace();
            }
        }
    }
}
```

#### INSERT

- 동적으로 할당하는 값이 있는 query(insert, update ,delete)등은 **prepareStatement()** 를 이용한다
  - prepareStatement는 **PreparedStatement** 객체를 반환하고, 이 객체의 **setString()**, **setInt()** 등의 메서드를 이용하여 동적으로 값을 할당
  - 할당 후 **executeUpdate()** 메서드를 호출하여 query 수행. 반환 값은 영향을 받은 row의 수
  - 결과를 얻고자 하면(select에 사용한 경우), **executeQuery()** 를 수행한다. 반환 값은 해당 row들

```java
import java.sql.*;

public class InsertTest {
    public static void main(String[] args) {
        // pet 테이블에는 이름/소유자/종/성별/출생일 칼럼이 있습니다.
        insert("봄이", "victolee", "페르시안", "m", "2010-08-21", null);
    }

    public static void insert(String name, String owner, String species,
                              String gender, String birth, String death){
        Connection conn = null;
        PreparedStatement pstmt = null;

        try{
            //드라이버 로드
            Class.forName("com.mysql.jdbc.Driver");

            //연결하기
            String url = "jdbc:mysql://localhost/dev";
            conn = DriverManager.getConnection(url, "dev", "dev");


            //SQL
            String sql = "INSERT INTO pet VALUES (?,?,?,?,?,?)";
            pstmt = conn.prepareStatement(sql);

            //?에 순서대로 들어간다 0이 아닌 1부터 시작
            pstmt.setString(1, name);
            pstmt.setString(2, owner);
            pstmt.setString(3, species);
            pstmt.setString(4, gender);
            pstmt.setString(5, birth);
            pstmt.setString(6, death);


            //쿼리 수행
            int count = pstmt.executeUpdate();
        }
        catch( ClassNotFoundException e){
            System.out.println("Fail : loading a driver");
        }
        catch( SQLException e){
            System.out.println("Error : " + e);
        }
        finally{
            try{
                if( pstmt != null && !pstmt.isClosed()){
                    pstmt.close();
                }
                if( conn != null && !conn.isClosed()){
                    conn.close();
                }
            }
            catch( SQLException e){
                e.printStackTrace();
            }
        }
    }
}
```

- - -

DAO & DTO
=========

DAO
----
- DAO(Data Access Object)
- DB질의를 통해 데이터에 접근하는 객체.
- DB한 테이블에 하나의 **DAO 클래스** 를 만들어 두면 유지보수가 편해짐
- CRUD 지원(inset, select, update, delete)
- MVC의 Model

DTO
----
- DTO(Data Transfer Object)
- 계층(layer)간 데이터 교환을 위한 javaBeans
- DB의 한 테이블에 존재하는 column들을 멤버 변수로 작성하여 레코드를 java객체로 다룸
- 순수한 데이터 객체로 메서드는 **getter, setter** 만을 가진다
- VO : DTO와 같은 개념이지만 READ-ONLY이다

예제
----

- UserDTO 클래스

```java
public class UserDTO{
    private String name;
    public UserDTO(String name){
      this.name=name;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

- UserDAO 클래스

```java
import java.sql.*;

public class UserDAO {
    private Connection getConnection() throws SQLException {
        Connection conn = null;
        try {
            Class.forName("com.mysql.jdbc.Driver");
            String url = "jdbc:mysql://localhost/test";
            conn = DriverManager.getConnection(url, "id", "passwd");
        }
        catch (ClassNotFoundException e) {
            System.out.println("Fail : loading a driver");
        }
        return conn;
    }

    public boolean insert(UserDTO dto) {
        boolean result = false;
        Connection conn = null;
        PreparedStatement pstmt = null;

        try {
            conn = getConnection();

            String sql = "INSERT INTO user VALUES (null, ?, ?, ?);";
            pstmt = conn.prepareStatement(sql);

            pstmt.setString(1, dto.getName());
            int count = pstmt.executeUpdate();
            result = (count == 1);
        }
        catch (SQLException e) {
            e.printStackTrace();
        }
        finally {
            try {
                if( pstmt != null )pstmt.close();
                if( conn != null )conn.close();
            }
            catch(SQLException e) {
                e.printStackTrace();
            }
        }
        return result;
    }
    public ArrayList<UserDTO> select(UserDTO dto){
      Connection conn = null;
      PreparedStatement pstmt = null;
      ResultSet rSet = null;

      ArrayList<UserDTO> retList = new ArrayList<UserDTO>();  

      try {
          conn = getConnection();

          String sql = "SELECT * from user where name=?";
          pstmt = conn.prepareStatement(sql);

          pstmt.setString(1, dto.getName());
          rSet = pstmt.executeQuery();
          while(rSet.next()){
            String name =rSet.getString("name");
            UserDTO record = new UserDTO(name);
            retList.add(record);
          }
      }
      catch (SQLException e) {
          e.printStackTrace();
      }
      finally {
          try {
              if(rSet!=null)rSEt.close();
              if(pstmt!=null)pstmt.close();
              if(conn!=null)conn.close();
          }
          catch(SQLException e) {
              e.printStackTrace();
          }
      }
      return retList.isEmtpy()?null:retList;
    }
}
```

- servlet

```java
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/add")
public class UserServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("utf-8");

        UserDTO dto = new UserDTO();
        vo.setName(request.getParameter("name"));

        UserDAO dao = new UserDAO();
        dao.insert(dto);
        ArrayList<UserDTO> recordList = dao.select(dto);
        request.setAttribute("recordList",recordList);
        request.getRequestDispatcher("newUrl").forward(request,response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```

- [How to pass data from a Servelt to a jsp](https://stackoverflow.com/questions/51608047/sending-data-from-servlet-to-jsp)
  1. getRequestDispatcher 사용
  2. session 사용

- response.sendRedirect vs request.getRequestDispatcher
  - sendRedirect : HTTP 헤더에 리다이렉트 정보를 담아 보냄. 브라우저가 URL로 리다이렉트. sendRedirect 이후에도 서버는 동작함(그러므로 return 시켜주자). 경로는 절대 경로를 사용함. request, response 객체가 새로 생긴다
  - getRequestDispather: web container가 페이지 이동을 시키기 때문에 브라우저는 알 수 없음(브라우저의 URL 변화 없음). 이후의 코드가 무시. request, response 객체 재사용 가능

- - -

참고자료
=======

- <https://victorydntmd.tistory.com/145>
- <https://rwd337.tistory.com/21>
- <https://victorydntmd.tistory.com/149>
