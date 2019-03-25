### SPI
#### Java SPI
    SPI 全称为 Service Provider Interface，是一种服务发现机制。SPI 的本质是将接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类。这样可以在运行时，动态为接口替换实现类。正因此特性，我们可以很容易的通过 SPI 机制为我们的程序提供拓展功能。
    比如mysql-connector-java-5.1.18.jar，就有一个/META-INF/services/java.sql.Driver里面内容是 com.mysql.jdbc.Driver。

    ![](https://raw.githubusercontent.com/null23/picture/master/Dubbo/Java-MySQL-SPI.png)


#### Java SPI Demmo
    https://dubbo.incubator.apache.org/zh-cn/docs/source_code_guide/dubbo-spi.html
    参考以上Dubbo官方文档的Demo。
    这里有一个需要注意的地方，就是创建META-INF目录的时候，需要以下路径META-INF/services，记得在META-INF后创建services文件，并且把配置文件写在里边