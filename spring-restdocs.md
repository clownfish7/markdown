## Spring REST Docs


> Spring.io 官方文档： https://docs.spring.io/spring-restdocs/docs/2.0.4.RELEASE/reference/html5/
>
> Asciidoc 官方文档（用户手册）：https://asciidoctor.org/docs/user-manual/#env-attributes
>
> Asciidoc 官方文档（Maven插件）：https://asciidoctor.org/docs/asciidoctor-maven-plugin/#built-in-attributes



**环境说明**：jdk-1.8.0_101、springboot-2.3.0-RELEASE、maven-3.5.4

### 1. 引入 pom 相关依赖 ###

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/>
    </parent>
    <groupId>com.clownfish7</groupId>
    <artifactId>springboot-restdoc</artifactId>
    <version>1.0.0</version>
    <name>springboot-restdoc</name>
    <description>springboot-restdoc</description>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <fastjson.version>1.2.58</fastjson.version>
        <maven.build.timestamp.format>yyyy-MM-dd HH:mm:ss</maven.build.timestamp.format>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-mockmvc</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- change maven build utc time  -->
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>build-helper-maven-plugin</artifactId>
                <version>1.9.1</version>
                <executions>
                    <execution>
                        <id>timestamp-property</id>
                        <goals>
                            <goal>timestamp-property</goal>
                        </goals>
                        <configuration>
                            <name>build.time</name>
                            <pattern>yyyy-MM-dd HH:mm:ss</pattern>
                            <locale>zh_CN</locale>
                            <timeZone>Asia/Shanghai</timeZone>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.asciidoctor</groupId>
                <artifactId>asciidoctor-maven-plugin</artifactId>
                <version>2.0.0</version>
                <configuration>
                    <attributes>
                        <!-- 使用节在生成中启用节编号 -->
                        <sectnums>true</sectnums>
                        <!-- 将版本和生成日期添加到标头 -->
                        <revnumber>${project.version}</revnumber>
                        <!-- 修改 maven.build.timestamp 为指定时区即上方插件定义的时间 -->
                        <!--<revdate>${maven.build.timestamp}</revdate>-->
                        <revdate>${build.time}</revdate>
                        <!-- asciidoctor 官网 copy 来的，似乎并没有什么用，没有在生成的 doc 中看到 organization -->
                        <organization>${project.organization.name}</organization>
                        <!-- default is the last update time with file which /src/main/asciidoc/*.adoc -->
                        <docdatetime>${build.time}</docdatetime>
                    </attributes>
                </configuration>
                <executions>
                    <!-- html -->
                    <execution>
                        <id>output-html</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>process-asciidoc</goal>
                        </goals>
                        <configuration>
                            <backend>html5</backend>
                            <doctype>book</doctype>
                        </configuration>
                    </execution>

                    <!-- pdf -->
                    <!--<execution>
                        <id>output-pdf</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>process-asciidoc</goal>
                        </goals>
                        <configuration>
                            <backend>pdf</backend>
                            <doctype>book</doctype>
                            &lt;!&ndash; 中文乱码 &ndash;&gt;
                            <attributes>
                                <pdf-fontsdir>${basedir}/pdf/fonts</pdf-fontsdir>
                                <pdf-stylesdir>${basedir}/pdf/themes</pdf-stylesdir>
                                <pdf-style>cn</pdf-style>
                            </attributes>
                        </configuration>
                    </execution>-->

                    <!-- xml -->
                    <!--<execution>
                        <id>output-docbook</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>process-asciidoc</goal>
                        </goals>
                        <configuration>
                            <backend>docbook</backend>
                            <doctype>book</doctype>
                        </configuration>
                    </execution>-->
                </executions>
                <dependencies>
                    <!-- maybe dont need - copy from spring-restdoc -->
                    <dependency>
                        <groupId>org.springframework.restdocs</groupId>
                        <artifactId>spring-restdocs-asciidoctor</artifactId>
                        <version>2.0.4.RELEASE</version>
                    </dependency>
                    <!-- pdf need -->
                    <dependency>
                        <groupId>org.asciidoctor</groupId>
                        <artifactId>asciidoctorj-pdf</artifactId>
                        <version>1.5.3</version>
                    </dependency>
                </dependencies>
            </plugin>
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.7</version>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>
                                ${project.build.outputDirectory}/static/docs
                            </outputDirectory>
                            <!-- 每个版本在单独的文件夹中生成文档 -->
                            <!--<outputDirectory>target/generated-docs/${project.version}</outputDirectory>-->
                            <resources>
                                <resource>
                                    <directory>
                                        ${project.build.directory}/generated-docs
                                    </directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>

```

