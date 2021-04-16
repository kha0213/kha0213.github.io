---
title:  "JPA persistence.xml Setting"

categories: 
  - jpa
tags:
  - jpa
  - study
last_modified_at: 2021-04-04T00:40:00-00:00
---

## What is the [persistence.xml] file?
[참고 https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/undejdev003.htm#CHDIDBBF](https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/undejdev003.htm#CHDIDBBF)   
persistence.xml은 엔티티를 사용하는 EJB 3.0 application 프로그램에서 (하나 이상의) persistence-units 설정하는 파일이다.    
(JPA는 EJB entity에서 시작하여 그 설정정보를 그대로 가져온다.)    

**Note:** 
persistence.xml 은 resources/META-INF 경로에 있으면 jpa가 자동으로 인식한다.
{: .notice--info}   

* persistence-units은 4가지 구성으로 이루어져 있다.

### Entity manager
기본 설정으로 Entity manager를 얻을 수 있다. (orm.xml로 매핑이 되어 @Entity 사용 가능)
* Data source   
[참고 : https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/servdats.htm#BEICDGCC](https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/servdats.htm#BEICDGCC)   

``` xml
<persistence-unit name="default" transaction-type="JTA">
    <provider>
        oracle.toplink.essentials.PersistenceProvider
    </provider>
    <jta-data-source>
        jdbc/MyDataSource
    </jta-data-source>
    <properties>
        <property name="toplink.logging.level" value="INFO"/>
    </properties>
</persistence-unit>
```

### Vendor extensions   
property를 통해 설정 정보를 변경할 수 있다.   
[변경 property 설명 : https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/cfgdepds005.htm#TopLinkJDBC](https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/cfgdepds005.htm#TopLinkJDBC)   
  
ex)  
``` xml
<persistence-unit name="default" transaction-type="RESOURCE_LOCAL">
    <provider>
        oracle.toplink.essentials.PersistenceProvider
    </provider>
    <exclude-unlisted-classes>false</exclude-unlisted-classes>
    <properties>
        <property name="toplink.logging.level" value="INFO"/>
        <property name="toplink.jdbc.driver" value="oracle.jdbc.OracleDriver"/>
        <property name="toplink.jdbc.url" value="jdbc:oracle:thin:@myhost:l521:MYSID"/>
        <property name="toplink.jdbc.password" value="tiger"/>
        <property name="toplink.jdbc.user" value="scott"/>
    </properties>
</persistence-unit>
```
### Persistent managed classes   
Entity manager를 사용하는 여러 기능들에 대한 정의이다.   
namely, entity classes, embeddable classes, and mapped superclasses...     

### Mapping metadata   
영속관리되는 class와 database와의 mapping 설정이다. 대표적으로 orm.xml이 있다.   

**Note:**   
orm.xml은 객체와 관계형데이터베이스 mapping을 지정하는 파일이다.   
{: .notice--info}

## Template
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             version="2.1">
    <persistence-unit name="simple-jpa-application">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/jpastudy"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <!-- 콘솔에 하이버네이트가 실행하는 SQL문 출력 -->
            <property name="hibernate.show_sql" value="true"/>
            <!-- SQL 출력 시 보기 쉽게 정렬 -->
            <property name="hibernate.format_sql" value="true"/>
            <!-- 쿼리 출력 시 주석(comments)도 함께 출력 -->
            <property name="hibernate.use_sql_comments" value="true"/>
            <!-- JPA 표준에 맞춘 새로운 키 생성 전략 사용 -->
            <property name="hibernate.id.new_generator_mappings" value="true"/>
            <!-- 애플리케이션 실행 시점에 데이터베이스 테이블 자동 생성 -->
            <property name="hibernate.hbm2ddl.auto" value="create"/>
            <!-- 이름 매핑 전략 설정 - 자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 매핑
             ex) lastModifiedDate -> last_modified_date -->
            <property name="hibernate.ejb.naming_strategy" value="org.hibernate.cfg.ImprovedNamingStrategy"/>
        </properties>
    </persistence-unit>
</persistence>
```
    


## Reference

[https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/cfgdepds005.htm](https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/cfgdepds005.htm)     


