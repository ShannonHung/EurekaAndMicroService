# EurekaAndMicroService

Hack.md : https://hackmd.io/@EasyEatWeb/Skg5h29fD

# 總結
首先我要先架一個Eureka Server，待會給MicroService去註冊用的
## 1. 架Eureka Server步驟
* step1 先設定POM.XML : 必須要有`spring-cloud-starter-config`/ `spring-cloud-netflix-eureka-server` / `spring-cloud-starter-parent`
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
            <version>2.4.0-b180725.0427</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-parent</artifactId>
                <version>Greenwich.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
* step2 再來去設定`application.properties`
	* ==你會發現`logging.level.com.netflix.eureka=ON`不能用，請改成`DEBUG`==
```xml
spring.application.name=eureka-server 
server.port=8761

#don't register itself as a client
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
logging.level.com.netflix.eureka=DEBUG
logging.level.com.netflix.discovery=DEBUG
```
* step3 在`@SpringBootApplication` 的地方再新增一個`@EnableEurekaServer`
```java 
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```
* step4 嘗試去 http:localhost/8761 去看看有沒有出現Eureka的畫面，如果有就可以進行下一個步驟 建立MicroService

## 2. 架設好Eureka Service之後呢，就可以開始製作MicroService
* step1 先建立基本設定 `POM.xml` : 
	* 需要有jpa才可以設定@Entity等標示 
	* 基本的`starter-web`&`starter-test` 
	* `spring-boot-starter-data-rest`才可以使用`CrueRepository`
	* 然後可以存放Entity資料的h2
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>
```
* step2: 建立Entity
* step3: 建立Entity的Repository
* step4: 嘗試去 http:localhost:8080/items 檢查看看有沒有出現JSON，如果有就可以去下一個步驟

## 3. 開始將microservice自動註冊到eureka service裡面，有幾個步驟，要非常小心!!尤其是POM.xml設置的地方
* step1: 設定 pom.xml 新增一些dependency==注意版本要跟Eureka server一樣，否則會壞掉==
	* 下面的`<version>2.1.0.RELEASE</version>`是參考1-step1的版本`spring-cloud-netflix-eureka-server`一定要一樣!
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
```
* step2: 去`application.properties`設定client的相關設定
	* `spring.application.name`: 這是註冊在Eureka Service的名稱
	* 在client裡面的`server.port`跟server裡面的`server.port`不一樣，client的可以看到items的JSON，server是用來設定Eureka service的port
	* `eureka.client.serviceUrl.defaultZone`: 這是用來設定eureka查看的網址，8761要跟server的`server.port`一樣
```xml
#Eureka
spring.application.name=item-service
server.port=8762
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
eureka.client.service-url.default-zone=http://localhost:8761/eureka/
instance.preferIpAddress=true
```
* step3: 去MicroServiceApplication.java增加`@EnableEurekaClient`
```java 
@EnableEurekaClient
@SpringBootApplication
public class MicroservicesApplication {

    public static void main(String[] args) {
        SpringApplication.run(MicroservicesApplication.class, args);
    }
}
```
* step4: 最後執行`microserviceApplication.main`去查看 http://localhost:8761 應該可以進入spring Eureka畫面並且看到 ITEM-SERVICE被註冊了。也可以進入http://localhost:8762 看到相關JSON
> http://localhost:8762
![](https://i.imgur.com/ZWdfFem.png)

> http://localhost:8761
![](https://i.imgur.com/eNDpThS.png)


 ==DONE== 機車幹花我兩個晚上找bug
 
