## MyBatis的学习记录

#### 1，mybatis的简介：略

### 二.入门案例的操作步骤

1mybatis的环境搭建

​             第一步：创建maven的工程并导入坐标.

~~~ java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>tech.aistar</groupId>
    <artifactId>mybatis_day01_wubin</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>


    <dependencies>
<!--        mybatis的依赖-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.5</version>
        </dependency>

<!--操作数据库，那么需要mysql的依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>

<!-- 日志记录的依赖（可选）-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>

<!--单元测试的依赖（可选）-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
~~~

第二步;创建实体类和dao的接口

~~~ java
package tech.aistar.entity;

import java.io.Serializable;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/1
 * @
 */
public class User implements Serializable {

    private Integer id;

    private String username;

    private String password;

    private  int power;

    public User() {
    }

    public User(Integer id, String username, String password, int power) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.power = power;
    }


    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public int getPower() {
        return power;
    }

    public void setPower(int power) {
        this.power = power;
    }


    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", power=" + power +
                '}';
    }
}

创建dao
    
    package tech.aistar.dao;

import tech.aistar.entity.User;

import java.util.List;

/**
 * 接口功能描述
 *
 * @Author: admin
 * @Date: 2020/2/1
 * @
 */
public interface IUserDao {


    /**
     * 查询所有的操作
     * @return
     */
    List<User> findAll();
}

~~~

第三步：创建mybatis的主配置文件SqlMapperConfig.xml

注意：maven的工程，我们需要将此配置文件存放在resource资源路径下

~~~ java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--环境的配置，mybatis的核心配置文件-->
<configuration>
<!-- 映射实体类的别名-->
    <typeAliases>
        <package name="tech.aistar"/>
    </typeAliases>
<!--起一个名字，没有强制的规定，但是这里使用什么环境配置，那么下面就需要有相应的对应-->
    <environments default="mysql">
<!-- 配置mysql的环境-->
        <environment id="mysql">
<!--  配置事务的类型，这里使用的是jdbc的-->
            <transactionManager type="JDBC"></transactionManager>
<!-- 配置数据源，也叫连接池-->
            <dataSource type="POOLED">
                <!-- 需要在数据源中配置四个基本的连接数据库的基本信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/chaohu"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>

        </environment>
    </environments>


<!-- 配置mybatis的映射文件的位置，映射文件指的是每一个dao独立的配置文件-->
    <mappers>
<!-- resource:指定配置文件的位置-->
        <mapper resource="tech/aistar/dao/IUserDao.xml"></mapper>
    </mappers>
</configuration>
~~~

第四步：创建映射配置文件IUserDao.xml

但是要注意，此文件也要写在resource资源文件夹下。并且这个文件所在的包也要一个一个的写

~~~ java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace里面填写的是你的方法所在的接口的全限定名-->
<mapper namespace="tech.aistar.dao.IUserDao">
<!-- 配置查询所有的方法的映射-->
    <select id="findAll" resultType="user">
        select * from user;
    </select>

</mapper>

~~~

单元测试

~~~ java
package tech.aistar.dao;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import tech.aistar.entity.User;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/2
 * @mybatis入门案例的测试类
 */
public class UserDaoTest {
    public static void main(String[] args) {
        //1.读取配置文件
        try {
            InputStream in = Resources.getResourceAsStream("SqlMapperConfig.xml");
            //2.创建SqlSessionFactory工厂来创建出连接的会话对象
            SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
            //3.使用工厂生产出SqlSession对象
            SqlSession sqlSession = factory.openSession();
            //4.使用SqlSession创建Dao接口的代理对象，使用代理对象执行方法
            IUserDao dao = sqlSession.getMapper(IUserDao.class);
            //调用dao中的方法
           List<User> list = dao.findAll();
            for (User user : list) {
                System.out.println(user);

            }
             //关闭资源
            sqlSession.close();
            in.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

~~~

### 拓展:自定义Mybatis的分析与是实现：

​              mybatis在使用代理dao的方式实现增删改查时候做了什么事情?

​              两件事情: 

​                           第一：创建了代理对象

​                           第二：在代理对象中调用selectList的方法

下面我们来自己手写一个自定义的MyBatis

首先，明确出现的几个类：class Resources

​                                              class SqlSessionFactoryBuilder

​                                               interface SqlSessionFactory

​                                               interface SqlSession

#### 第一步删除原来的pom.xml文件中的mybatis的依赖

~~~ java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>tech.aistar</groupId>
    <artifactId>mybatis_day01_wubin</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>


    <dependencies>
<!--        mybatis的依赖-->
        //<dependency>
          //  <groupId>org.mybatis</groupId>
           // <artifactId>mybatis</artifactId>
           // <version>3.4.5</version>
        //</dependency>

<!--操作数据库，那么需要mysql的依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>

<!-- 日志记录的依赖（可选）-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>

<!--单元测试的依赖（可选）-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
            <scope>test</scope>
        </dependency>
    </dependencies>


</project>
~~~

下面开始重塑上面的那些类

~~~ java
RESOURCES这个类
    package tech.aistar.MybatisConfigUtils;

import java.io.InputStream;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/2
 * @这个是复原类加载器读取配置文件的一个类
 */
public class Resources {

    //根据传入的参数获取一个字节输入流
    public static InputStream getResourceAsStream(String fileName){
        //直接返回当前这个类的字节码的类加载器读取流
        return Resources.class.getClassLoader().getResourceAsStream(fileName);


    }

}

~~~

第二个：SqlSessionFactoryBuilder

~~~ java
package tech.aistar.MybatisConfigUtils;

import tech.aistar.MybatisConfigUtils.Impl.DefaultSqlSessionFactory;
import tech.aistar.utils.Configuration;
import tech.aistar.utils.XMLConfigBuilder;

import java.io.InputStream;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/2
 * @用于创建一个SqlSessionFactory对象
 */
public class SqlSessionFactoryBuilder {

    /**
     * 根据参数的字节输入流来构建出一个SqlSessionFactory工厂
     * @param in
     * @return
     */
    public SqlSessionFactory build(InputStream in){

        //创建配置文件的类
        Configuration cfg = XMLConfigBuilder.loadConfiguration(in);

        return new DefaultSqlSessionFactory(cfg);
    }

}

~~~

因为我们不在是使用mybatis的依赖的管理了，那么那些配置文件中的mybatis的头部信息就不需要了

IUserDao.xml的配置

~~~ java
<?xml version="1.0" encoding="UTF-8" ?>

<!--namespace里面填写的是你的方法所在的接口的全限定名-->
<mapper namespace="tech.aistar.dao.IUserDao">
<!-- 配置查询所有的方法的映射-->
    <select id="findAll" resultType="user">
        select * from user;
    </select>

</mapper>

~~~

mybatis的核心配置文件

~~~ java
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
<!-- 映射实体类的别名-->
    <typeAliases>
        <package name="tech.aistar"/>
    </typeAliases>
<!--起一个名字，没有强制的规定，但是这里使用什么环境配置，那么下面就需要有相应的对应-->
    <environments default="mysql">
<!-- 配置mysql的环境-->
        <environment id="mysql">
<!--  配置事务的类型，这里使用的是jdbc的-->
            <transactionManager type="JDBC"></transactionManager>
<!-- 配置数据源，也叫连接池-->
            <dataSource type="POOLED">
                <!-- 需要在数据源中配置四个基本的连接数据库的基本信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/chaohu"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>

        </environment>
    </environments>


<!-- 配置mybatis的映射文件的位置，映射文件指的是每一个dao独立的配置文件-->
    <mappers>
<!-- resource:指定配置文件的位置-->
        <mapper resource="tech/aistar/dao/IUserDao.xml"></mapper>
    </mappers>
</configuration>
~~~

##### 那么此时我们需要构建SqlSessionFactory对象时需要读取的配置文件就需要我们手动的编写xml的读取来获取到我们的所有的xml文件中配置的内容了。

建一个utils包，里面存放的时配置文件的相关类Mapper类，Configuration类，XMLConfigBuilder类

#### XMLConfigBuilder类：用来解析我们写的xml文件

~~~ java
package tech.aistar.utils;

import org.dom4j.Attribute;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;
import tech.aistar.MybatisConfigUtils.Resources;

import javax.print.DocFlavor;
import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/2
 * @用于解析xml文件的
 */
public class XMLConfigBuilder {

    /**
     * 解析主配置文件，把里面的内容填充到DefaultSqlSession所需要的地方
     * 使用的技术：dom4j+xpath
     */

    public static Configuration loadConfiguration(InputStream config){

        try {
            //定义封装信息的配置对象(mybatis的配置对象)
            Configuration cfg = new Configuration();
            //1.获取SAXReader对象
            SAXReader reader = new SAXReader();
            //根据字节输入流获取文档对象Document对象
            Document document = reader.read(config);
            //3.获取根节点
            Element root = document.getRootElement();
            //4.使用xpath中选择指定节点的方式，获取所有property节点
            //肯定是一个的集合
            List<Element> propertyElements = root.selectNodes("//property");
            //5.遍历节点
            for (Element propertyElement : propertyElements) {
                //判断每个节点是连接数据库的哪些部分的信息
                //然后取出每个对应的节点的value的值
                String name = propertyElement.attributeValue("name");
                if ("driver".equals(name)){
                    //表示的是驱动的节点
                    //那么就获取该节点的值
                    String driver = propertyElement.attributeValue("value");
                    //将该值存放到配置对象中
                    cfg.setDriver(driver);
                }
                if ("url".equals(name)){
                    //表示的是url的节点
                    //那么就获取该节点的值
                    String url = propertyElement.attributeValue("value");
                    //将该值存放到配置对象中
                    cfg.setUrl(url);
                }
                if ("username".equals(name)){
                    //表示的是用户名的节点
                    //那么就获取该节点的值
                    String username = propertyElement.attributeValue("value");
                    //将该值存放到配置对象中
                    cfg.setUsername(username);
                }
                if ("password".equals(name)){
                    //表示的是密码的节点
                    //那么就获取该节点的值
                    String password = propertyElement.attributeValue("value");
                    //将该值存放到配置对象中
                    cfg.setPassword(password);
                }

            }
            //取出mappers中所有的mapper标签，判断他们使用的是resource还是class属性
            List<Element> mapperElements = root.selectNodes("//mappers/mapper");
            //遍历集合
            for (Element mapperElement : mapperElements) {
                //判断mapperElement使用的是哪个属性
                //获取resource属性
                Attribute attribute = mapperElement.attribute("resource");
                if (attribute!=null){
                    //不为null,表示使用的就是resource属性，就是xml的配置方式
                    System.out.println("使用的是xml的配置方式");
                    //取出属性的值
                    String mapperPath = attribute.getValue();//获取到的结果为tech.aistar.dao.IUserDao.xml
                    //把映射文件的内容获取出来，封装成一个map
                    Map<String,Mapper> mappers = loadMapperConfiguration(mapperPath);
                    //给configuration中的mappers赋值
                    cfg.setMappers(mappers);
                }else {
//                    System.out.println("使用的式注解的开发方式");
//                    //表示没有resource属性，使用的是注解
//                    //获取class属性的值
//                    String daoClassPath = mapperElement.attributeValue("class");
//                    //根据daoClassPath获取封装的必要信息
//                    Map<String,Mapper> mappers = loadMapperAnnotation(daoClassPath);
//                    //给configuration中的mappers赋值
//                    cfg.setMappers(mappers);
                }

            }
            //返回Configuration对象
            return cfg;
        } catch (Exception e) {
         throw  new RuntimeException(e);
        }finally {
            try {
                config.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 根据传入的参数，解析xml,并且封装到Map中
     * @param mapperPath  映射配置文件的位置
     * @return  返回值中的map包含了获取到的唯一标识(key是由dao的全限定类名和方法名组成)
     *           以及执行所需的必要信息(value是一个Mapper对象，里面存放的是执行的SQL语句和要封装的实体类的全限定名)
     */
  private static Map<String,Mapper> loadMapperConfiguration(String mapperPath) throws IOException {
        InputStream in = null;

      try {
          //定义返回值对象
          Map<String,Mapper> mappers = new HashMap<String, Mapper>();
          //1.根据路径获取字节流
          in = Resources.getResourceAsStream(mapperPath);
          //2.根据字节输入流获取Document对象
          SAXReader reader = new SAXReader();
          Document document = reader.read(in);
          //3.获取根节点
          Element root = document.getRootElement();
          //4.获取节点的namespace的属性的值
          String namespace = root.attributeValue("namespace");//这个是组成返回值map中的key的部分
          //5.获取所有的select(查询的)节点集合
          List<Element> selectElements = root.selectNodes("//select");
          //6.遍历select节点集合
          for (Element selectElement : selectElements) {
              //取出id属性的部分    和namespace一起组成key
              String id = selectElement.attributeValue("id");
              //取出resultType属性的值  组成value的部分
              String resultType = selectElement.attributeValue("resultType");
              //取出文本内容
              String queryString = selectElement.getText();
              //创建key
              String key = namespace+"."+id;
              //创建value,它是一个对象，该对象中封装了我们需要的文本信息
              Mapper mapper = new Mapper();
              mapper.setQueryString(queryString);
              mapper.setResultType(resultType);
              //将key和value存放到mappers中
              mappers.put(key,mapper);
          }
          return mappers;
      } catch (Exception e) {
          throw new RuntimeException(e);
      }finally {
          in.close();
      }

  }


    /**
     * 根据传入的参数，得到所有被select注解标注的方法
     * @param mapperPath  映射配置文件的位置
     * @return  根据方法名和类名，以及方法上的注解value属性的值，组成Mapper的必要信息
     *
     */
//    private static Map<String,Mapper> loadMapperAnnotation(String daoClassPath) throws Exception {
//
//
//            //定义返回值对象
//            Map<String,Mapper> mappers = new HashMap<String, Mapper>();
//            //1.得到dao接口的字节码对象
//        Class daoClass = Class.forName(daoClassPath);
//           //2.得到dao接口中的方法组
//        Method[] methods = daoClass.getMethods();
//        //遍历方法的数组
//        for (Method method : methods) {
//            //取出每一个方法，判断是否有select注解
//            boolean isAnnotated = method.isAnnotationPresent(Select.class);
//            if (isAnnotated){
//                //说明有该注解
//                //创建Mapper对象
//                Mapper mapper = new Mapper();
//                //取出注解的value的属性值
//                Select selectAnno = method.getAnnotation(Select.class);
//                String queryString = selectAnno.value();
//                mapper.setQueryString(queryString);
//                //获取当前的方法的返回值，还必须要求带有泛型信息
//                Type type = method.getGenericReturnType();//List<User>
//                //判单type是不是参数化类型
//                if (type instanceof ParameterizedType){
//                 //强转
//                 ParameterizedType ptype =(ParameterizedType) type;
//                 //得到参数化类型中的实际参数
//                    Type[] types = ptype.getActualTypeArguments();
//                    //取出第一个
//                    Class domainClass = (Class) types[0];
//                    //获取domainClass类名
//                    String resultType = domainClass.getName();
//                    //给Mapper赋值
//                    mapper.setResultType(resultType);
//                }
//
//                //组装key的信息
//                //获取方法名称
//                String methodName = method.getName();
//                String calssName = method.getDeclaringClass().getName();
//                String key = calssName+"."+methodName;
//                //给map赋值
//                mappers.put(key,mapper);
//            }
//
//        }
//
//        return mappers;
//
//    }


}

~~~

#### Mapper类

~~~ java
package tech.aistar.utils;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/2
 * @用于封装执行的sql语句和结果类型的全限定类名
 */
public class Mapper {

    private String queryString;//SQL

    private String resultType;//实体类的全限定类名

    public String getQueryString() {
        return queryString;
    }

    public void setQueryString(String queryString) {
        this.queryString = queryString;
    }

    public String getResultType() {
        return resultType;
    }

    public void setResultType(String resultType) {
        this.resultType = resultType;
    }
}

~~~

#### Configuration类

~~~ java
package tech.aistar.utils;

import java.util.Map;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/2
 * @自定义mybatis的配置类
 */
public class Configuration {
    //配置类中就是定义一些数据库连接的信息
    private String driver;

    private String url;

    private  String username;

    private String password;

    private Map<String,Mapper> mappers;



    public String getDriver() {
        return driver;
    }

    public void setDriver(String driver) {
        this.driver = driver;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void setMappers(Map<String, Mapper> mappers) {
        //注意此处的赋值，需要使用追加的方式，不能使用传统的赋值方式，不然之前的值会被覆盖掉
        this.mappers.putAll(mappers);
    }

    public Map<String, Mapper> getMappers() {
        return mappers;
    }
}
~~~

..........详情请见项目：zidingyi_mybatis



#### 注解的mybatis的配置

首先将我们的mybatis的文件中的mappers的标签中的resource属性换成class属性配置

~~~ java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--环境的配置，mybatis的核心配置文件-->
<configuration>
<!-- 映射实体类的别名-->
    <typeAliases>
        <package name="tech.aistar"/>
    </typeAliases>
<!--起一个名字，没有强制的规定，但是这里使用什么环境配置，那么下面就需要有相应的对应-->
    <environments default="mysql">
<!-- 配置mysql的环境-->
        <environment id="mysql">
<!--  配置事务的类型，这里使用的是jdbc的-->
            <transactionManager type="JDBC"></transactionManager>
<!-- 配置数据源，也叫连接池-->
            <dataSource type="POOLED">
                <!-- 需要在数据源中配置四个基本的连接数据库的基本信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/chaohu"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>

        </environment>
    </environments>


<!-- 配置mybatis的映射文件的位置，映射文件指的是每一个dao独立的配置文件-->
    <mappers>
<!-- resource:指定配置文件的位置-->
<!-- <mapper resource="tech/aistar/dao/IUserDao.xml"></mapper>-->
        <mapper class="tech.aistar.dao.IUserDao"></mapper>
    </mappers>
</configuration>
~~~

然后在我们的dao的接口中的方法上打上我们需要的sql的注解

~~~ java
package tech.aistar.dao;

import org.apache.ibatis.annotations.Select;
import tech.aistar.entity.User;

import java.util.List;

/**
 * 接口功能描述
 *
 * @Author: admin
 * @Date: 2020/2/1
 * @
 */
public interface IUserDao {


    /**
     * 查询所有的操作
     * @return
     */
    @Select("select * from user")
    List<User> findAll();
}
~~~

然后我们就不需要再写接口的xml文件了。IUserDao.xml



### 下面正片开始，开始使用mybatis实现我们的CRUD的相关操作

首先我们再次的回顾下，我们是如何创建出mybatis的相关配置的

配置pom.xml依赖

~~~ java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    
    //配置我们的取别名
    <typeAliases>
        <package name="tech.aistar"/>
    </typeAliases>

<!-- 配置环境-->
    <environments default="mysql">
<!-- 配置mysql的环境-->
        <environment id="mysql">
          <!--配置事务的管理-->
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/chaohu"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>


    <mappers>
        <mapper resource="tech/aistar/dao/IUserDao.xml"></mapper>
    </mappers>
</configuration>
~~~

创建实体类user

~~~ java
package tech.aistar.entity;

import java.io.Serializable;
import java.util.Date;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/3
 * @
 */
public class User implements Serializable {
    private Integer id;

    private String username;

    private String password;

    private int power;

    private Date birthday;

    public User() {
    }

    public User(Integer id, String username, String password, int power, Date birthday) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.power = power;
        this.birthday = birthday;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public int getPower() {
        return power;
    }

    public void setPower(int power) {
        this.power = power;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", power=" + power +
                ", birthday=" + birthday +
                '}';
    }
}
~~~

创建测试的接口IUserDao

~~~ java
package tech.aistar.dao;

import tech.aistar.entity.User;

import java.util.List;

/**
 * 接口功能描述
 *
 * @Author: admin
 * @Date: 2020/2/3
 * @
 */
public interface IUserDao {


    /**
     * 查询所有
     * @return
     */
    List<User> findAll();


    /**
     * 保存的方法
     * @param user
     */
    void save(User user);

    /**
     * 根据id删除的方法
     * @param id
     */
    void delById(Integer id);

    /**
     * 根据id查询用户
     * @param id
     * @return
     */
    User findById(Integer id);


    /**
     * 更新用户
     * @param user
     */
    void updateUser(User user);

}

~~~

那么方法写好之后，我们要为该接口中的每一个方法都要写一个xml的标签

~~~ java
IUserDao.xml:

    <?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="tech.aistar.dao.IUserDao">
    <select id="findAll" resultType="tech.aistar.entity.User">
        select  * from user;
    </select>


    <select id="findById" parameterType="int" resultType="user">
        select * from user where id = #{id}
    </select>

    <delete id="delById" parameterType="int">
        delete from user where id = #{id}
    </delete>


    <insert id="save" parameterType="user">
        insert  into user (username, password, power, birthday) values (#{username},#{password},#{power},#{birthday})
    </insert>

    <update id="updateUser" parameterType="user">
        update user set username=#{username},password = #{password},power = #{power},birthday=#{birthday} where id = #{id}
    </update>
</mapper>
~~~

编写测试类

~~~java
package tech.aistar;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import tech.aistar.dao.IUserDao;
import tech.aistar.entity.User;

import java.io.IOException;
import java.io.InputStream;
import java.util.Date;
import java.util.List;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/3
 * @
 */
public class UserDaoTest {

    private SqlSessionFactory factory;

    private SqlSession sqlSession;

    private IUserDao dao;

    private InputStream in;

    @Before//在执行测试方法之前执行
    public void init(){
        try {
             in = Resources.getResourceAsStream("MybatisConfig.xml");
            factory = new SqlSessionFactoryBuilder().build(in);
            sqlSession = factory.openSession();
            dao = sqlSession.getMapper(IUserDao.class);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    @After//在执行测试方法之后执行
    public void close(){
       //因为mybatis在执行操作时，默认的是需要手动提交的
        sqlSession.commit();
        sqlSession.close();
        try {
            in.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    @Test
    public void testFindAll(){

        List<User> list = dao.findAll();
        for (User user : list) {
            System.out.println(user);

        }


    }

    @Test
    public void testSave(){
        User user = new User();
        user.setUsername("剑豪");
        user.setPassword("qwer");
        user.setPower(1);
        user.setBirthday(new Date());
        dao.save(user);

    }

}


~~~



获取新的一次插入的数据的id的值

~~~ java
使用的就是这个语句:select last_insert_id();
~~~

```xml
<insert id="save" parameterType="user">  
    <!--获取最新一次插入的记录的id的值-->   
    
    <!--使用的就是selectKey的标签，里面有几个属性：keyProperty:实体类属性的要获取的属性的值-->  
    <!--keyColumn是数据库表中字段的对应的名称    order:指的是执行的时机，此处是在插入语句执行之后执行    resultType:代表的是结果的类型-->     
    <selectKey keyProperty="id" keyColumn="id" order="AFTER" resultType="int">
    select last_insert_id();
    </selectKey>  
    insert  into user (username, password, power, birthday) values (#{username},#{password},#{power},#{birthday})
</insert>
```

##### mybatis中参数的深入，使用实体类的包装对象作为查询的条件进行查询。

使用的知识点是OGNL(对象导航语言)

OGNL:Object(对象)  Graphic(图)   Navigation(导航)  Language(语言)

它是通过对象的取值方法来获取数据，在写法上将get给省略掉了

​                                       比如：获取用户名称      类中的写法：user.getUserName();

​                                                                                OGNL表达式的写法：user.username

在开发中，我们可能在查询的时候，根据多个条件来进行查询，这些多个的查询的条件，我们可能会封装到一个具体的实体类中，然后将这个实体类作为一个查询的参数条件进行查询。

~~~ java 
   /**
     * 使用OGNL的思想来进行查询
     * 还是按照名称来进行模糊查询
     */

    List<User> findByVO(QueryVo queryVo);



<!--根据queryVo来进行查询-->
    <select id="findByVO" parameterType="QueryVo" resultType="user">
        select * from user where username like concat('%',#{user.username},'%')
    </select>
    
    
    测试：
    @Test
    public void testFindByVo(){
        QueryVo queryVo = new QueryVo();
        User user = new User();
        user.setUsername("小");
        queryVo.setUser(user);
        List<User> list = dao.findByVO(queryVo);
        for (User user2 : list) {
            System.out.println(user2);

        }
    }

~~~



#### 以上的操作都是基于类中的属性的值和数据库中的字段名称保持一致的时候完成的，那么如果实体类中的属性和表中的字段名称不一致怎么操作呢？

修改之后的实体类：

~~~ java
package tech.aistar.entity;

import java.io.Serializable;
import java.util.Date;

/**
 * 类功能描述
 *
 * @Author: admin
 * @Date: 2020/2/3
 * @
 */
public class User implements Serializable {
    private Integer uid;

    private String uname;

    private String pwd;

    private int userPower;

    private Date userBirthday;

    public User() {
    }

    public User(Integer uid, String uname, String pwd, int userPower, Date userBirthday) {
        this.uid = uid;
        this.uname = uname;
        this.pwd = pwd;
        this.userPower = userPower;
        this.userBirthday = userBirthday;
    }

    public Integer getUid() {
        return uid;
    }

    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public String getUname() {
        return uname;
    }

    public void setUname(String uname) {
        this.uname = uname;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }

    public int getUserPower() {
        return userPower;
    }

    public void setUserPower(int userPower) {
        this.userPower = userPower;
    }

    public Date getUserBirthday() {
        return userBirthday;
    }

    public void setUserBirthday(Date userBirthday) {
        this.userBirthday = userBirthday;
    }

    @Override
    public String toString() {
        return "User{" +
                "uid=" + uid +
                ", uname='" + uname + '\'' +
                ", pwd='" + pwd + '\'' +
                ", userPower=" + userPower +
                ", userBirthday=" + userBirthday +
                '}';
    }
}

~~~



#####  查询的语句进行封装的时候，你的实体类中的属性和表中的字段名称不一致的时候，就不能使用resultType属性了,找不到封装不了，需要使用resultMap属性来进行手动的封装.

~~~ java
<!--根据姓名来进行模糊查询-->
    <select id="findByName" parameterType="string" resultType="userMap">

        select * from user where username like concat('%',#{name},'%')
    </select>
    
    <resultMap id="userMap" type="user">
        <id property="uid" column="id"></id>
        <result property="uname" column="username"></result>
        <result property="pwd" column="password"></result>
        <result property="userPower" column="power"></result>
        <result property="userBirthday" column="birthday"></result>
        
    /resultMap>

~~~

#### mybatis中的连接池的相关操作

在日常的开发中，偶们都会使用到连接池，因为它会减少我们获取链接的时间

mybatis中的链接池：

​        mybatis中为我们提供了三种连接池的配置方式：

​      配置位置：在我们的mybatis的核心配置文件中的dataSources标签，type属性就表示我们采取的是何种链接方式。

   type的类型：属性的取值：

​                        POOLED:采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对它的实现

​                        UNPOLLED:采用传统的获取链接的方式，虽然也实现了上面的接口，但是没有采用池的思想

​                         JNDI：采用服务器提供的JNDI技术实现的。只能在web或者maven的war工程中使用，我们课程使用的是tomcat服务器，使用的是dbcp的连接池。

