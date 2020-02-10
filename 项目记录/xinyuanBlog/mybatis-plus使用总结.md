# Mybatis-plus使用总结

## 代码生成器

> AutoGenerator 是 MyBatis-Plus 的代码生成器，通过 AutoGenerator 可以快速生成 Entity、Mapper、Mapper XML、Service、Controller 等各个模块的代码，极大的提升了开发效率。

### 说明

之前使用的是mybatis-generator，感觉不好用。所以想试试mybatis-plus提供的这个代码自动生成功能，目前看来效果还是很好的。

根据网上的建议，我单独创建了一个项目去专门生成代码，然后再手动地复制进项目中。这样做虽然麻烦了手动复制，但是避免了文件从新生成的覆盖问题。



### 添加依赖

MyBatis-Plus 从 `3.0.3` 之后移除了代码生成器与模板引擎的默认依赖，需要手动添加相关依赖：

```xml
<dependencies>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-generator</artifactId>
        <version>3.0.1</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
        <version>1.18.4</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.11</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.1.2.RELEASE</version>
    </dependency>
</dependencies>
```

上面的所有依赖都需要添加。

### 创建生成类

CodeGenerator.java

```java
package cn.xinyuan.blog;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

/**
 * @ClassName: CodeGenerator
 * @Description: java类作用描述
 * @Author: xinyuan
 * @CreateDate: 2020/2/9 18:58
 */
// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class CodeGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        final String projectPath = System.getProperty("user.dir");//意思是读取当前用户的工作目录
        gc.setOutputDir(projectPath + "/src/main/java/");
        gc.setAuthor("xinyuan");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/xyblog?allowMultiQueries=true&useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=GMT");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        //pc.setModuleName(scanner("模块名"));
        pc.setParent("cn.xinyuan.blog");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/" + "xml"
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
    /*
    cfg.setFileCreate(new IFileCreate() {
        @Override
        public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
            // 判断自定义文件夹是否需要创建
            checkDir("调用默认方法创建的目录");
            return false;
        }
    });
    */
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        // 配置自定义输出模板
        //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        //strategy.setSuperEntityClass("你自己的父类实体,没有就不用设置!");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 公共父类
//        strategy.setSuperControllerClass("你自己的父类控制器,没有就不用设置!");
        // 写于父类中的公共字段
//        strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }

}
```

上面配置的具体解释还是要看官方文档的说明。这里只是把一些有歧义的说明一下。

- 全局配置

输出路径

```java
gc.setOutputDir(projectPath + "/src/main/java/");
```

这个是输出路径，不要写全，跟后面的包路径连接起来就是最后文件将会生成的地点。

包路径

```
pc.setParent("cn.xinyuan.blog");
```



```java
// 自定义配置会被优先输出
focList.add(new FileOutConfig(templatePath) {
    @Override
    public String outputFile(TableInfo tableInfo) {
        // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
        return projectPath + "/src/main/resources/mapper/" + "xml"
                + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
    }
});
```

xml的输出路径也要从新配置。



```java
strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
```

这句注释掉就是将数据库的所有表都生成，没有注释就需要在命令窗口手动输入需要生成的数据库表。

- 最后放上一张文件结构图，便于前面几个路径设置的理解。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E5%99%A8%E6%96%87%E4%BB%B6%E7%BB%93%E6%9E%84%E5%9B%BE.PNG)



## 分页

### 两种分页方式

- 物理分页：相当于执行了limit分页语句，返回部分数据。物理分页只返回部分数据占用内存小，能够获取数据库最新的状态，实施性比较强，一般适用于数据量比较大，数据更新比较频繁的场景。

- 逻辑分页：一次性把全部的数据取出来，通过程序进行筛选数据。如果数据量大的情况下会消耗大量的内存，由于逻辑分页只需要读取数据库一次，不能获取数据库最新状态，实施性比较差，适用于数据量小，数据稳定的场合。
  

### 配置分页插件

```java
//Spring boot方式
@EnableTransactionManagement
@Configuration
@MapperScan("com.baomidou.cloud.service.*.mapper*")
public class MybatisPlusConfig {

    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        // 开启 count 的 join 优化,只针对部分 left join
        paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
        return paginationInterceptor;
    }
}
```

### 具体分页实现

#### Wrapper构造查询条件

MP的Wrapper提供了两种分页查询的方式，源码如下：

```java
/**
 * 根据 entity 条件，查询全部记录（并翻页）
 *
 * @param page         分页查询条件（可以为 RowBounds.DEFAULT）
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
 
/**
 * 根据 Wrapper 条件，查询全部记录（并翻页）
 *
 * @param page         分页查询条件
 * @param queryWrapper 实体对象封装操作类
 */
IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```


```java
public void selectPage(){
    //构造查询条件
    QueryWrapper<User> queryWapper = new QueryWrapper<User>();
    queryWrapper.ge("age",26);
    
    Page<User> page = new Page<User>(1,2);//第一页，每页2条
    
    //返回的IPage是实体类
    IPage<User> iPage = userMapper.selectPage(page, queryWrapper);
    System.out.println("总页数",iPage.getPages());
    System.out.println("总记录数",iPage.getTotal());
    List<User> userList = iPage.getRecords();
    
    //返回的IPage是Map
    IPage<Map<String, Object>> iPage = userMapper.selectMapsPage(page, queryWrapper);
    System.out.println("总页数",iPage.getPages());
    System.out.println("总记录数",iPage.getTotal());
    List<Map<String, Object>> userList = iPage.getRecords();    
}
```



#### 自定义SQL分页查询

UserMapper.java

```java
public interface UserMapper extends BaseMapper<User>{
    IPage<User> selectPages(IPage<User> page, @Param(Constants.WRAPPER) Wrapper<User> queryWrapper);//第一个参数是设置好的Page，第二个参数也可以是作为查询条件的任何参数
}
```

UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.demo.mapper.UserMapper">
 
    <select id="selectMyPage" resultType="com.example.demo.model.User">
        SELECT * FROM user ${ew.customSqlSegment}
    </select>
 
</mapper>
```

测试

```java
   /**
     * 自定义sql分页查询
     */
    @Test
    public void selectByMyPage() {
        QueryWrapper<User> wrapper = new QueryWrapper();
        wrapper.like("name", "雨").lt("age", 40);
        Page<User> page = new Page<>(1,2);
        IPage<User> mapIPage = userMapper.selectMyPage(page, wrapper);
 
        System.out.println("总页数"+mapIPage.getPages());
        System.out.println("总记录数"+mapIPage.getTotal());
        List<User> records = mapIPage.getRecords();
        records.forEach(System.out::println);
    }
```





