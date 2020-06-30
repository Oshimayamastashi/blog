---
layout:     post
title:      "Spring Security Reference"
subtitle:   ""
date:       2020-06-20
author:     "Java"
header-img: "img/home-bg.jpg"
catalog: true
tags:
  - Spring
  - Spring Security
---

# Spring Security Reference

## Getting Started
Spring Security是一個基於Spring AOP跟Servlet過濾器的安全框架。它可以對基於Spring的應用提供身份驗證、授權和保護常見攻擊

### 發布版號

Spring Security版本的格式为MAJOR.MINOR.PATCH

* MAJOR版本可能包含重大變更，通常是改進安全性配合現代安全性實踐。
ex:4.0->5.0 支援OAuth2.0登入
* MINOR版本包含功能增強，但視為被動更新
* PATCH版本的跟新除了修正的Bug外，向前/向後兼容

### 使用Maven
#### 使用Spring Boot的Maven
Spring Boot提供了一個spring-boot-starter-security的starter把Spring Security相關的依賴包在一起
``` xml=
<dependencies>
    <!-- ... other dependency elements ... -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```
由於Spring Boot提供了Maven BOM來管理依賴版本，如果要改Spring Security版本，要調整Maven的屬性：
``` xml=
<properties>
    <spring-security.version>5.3.3.RELEASE</spring-security.version>
</dependencies>
```
#### 没有Spring Boot的Maven

使用沒有Spring Boot的Spring Security時，建議使用Spring Security的BOM，確保所有的dependency使用一致的Spring Security版本

``` xml=
<dependencyManagement>
    <dependencies>
        <!-- ... other dependency elements ... -->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-bom</artifactId>
            <version>5.3.3.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

最小的Spring Security Maven依賴關係集通常如下:
``` xml=
<dependencies>
    <!-- ... other dependency elements ... -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
    </dependency>
</dependencies>
```

#### Maven Repositories

所有的GA版本（.RELEASE结尾的版本）都會部署到Maven Central，如果要使用Snapshot版本或Milestone版本,pom檔要加上Repository

``` xml=
<repositories>
    <!-- ... possibly other repository elements ... -->
    <repository>
        <id>spring-snapshot</id>
        <name>Spring Snapshot Repository</name>
        <url>https://repo.spring.io/snapshot</url>
    </repository>
    <repository>
        <id>spring-milestone</id>
        <name>Spring Milestone Repository</name>
        <url>https://repo.spring.io/milestone</url>
    </repository>
</repositories>
```

### Project Modules

Spring Security在3.0後把代碼庫細分為不同的jar，這些jar區分了不同的功能區域和第三方依賴項:
* spring-security-core.jar
使用Spring Security的應用都需要，包含核心身份驗證、存取控制、遠程支持、基本配置API。
* spring-security-web.jar
包含過濾器和相關的Web安全基礎結構代碼跟所有與Servlet API相關的內容。ex:網頁身分驗證和URL的訪問控制
* spring-security-config.jar
包含安全命名空間解析跟Java Configuration
* spring-security-test.jar
支援使用Spring Security進行測試。
* spring-security-oauth2-core.jar
包含OAuth2.0框架的核心
* spring-security-oauth2-client.jar
支援OAuth 2.0的Client端功能。ex OAuth2.0登錄
* spring-security-remoting.jar
提供與Spring Remoting的整合
* spring-security-ldap。jar
包含LDAP身分驗證跟配置

## 驗證

### 密碼储存
Spring Security的PasswordEncoder介面用於執行密碼的單向轉換，以便安全地儲存密碼，實作它的類須要實作"如何編碼"以及"如何判斷未編碼的字元序列和編碼後的字串是否匹配"

``` java=
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);

    boolean matches(CharSequence rawPassword, String encodedPassword);
}
```

#### 歷史
![](https://i.imgur.com/O1TGmA8.png)
#### NoOpPasswordEncoder
Spring Security 5.0之前的預設密碼編碼器，已經被註記為@Deprecated
``` java=
@Deprecated
public final class NoOpPasswordEncoder implements PasswordEncoder {

    public String encode(CharSequence rawPassword) {
        return rawPassword.toString();
    }

    public boolean matches(CharSequence rawPassword, String encodedPassword) {
        return rawPassword.toString().equals(encodedPassword);
    }
}
```
#### DelegatingPasswordEncoder
Spring Security 5.0後為了替換過時的編碼器，將其他編碼器設為預設編碼器可能會遇到幾個問題:
* 有許多使用舊密碼編碼的資料無法輕鬆遷移
* 密碼儲存的最佳做法可能會再次發生變化
* 作為一個框架，Spring Security不能經常發生改變

為了維持新密碼編碼器和舊密碼的相容性跟可變性，5.0使用DelegatingPasswordEncoder作為預設的編碼器

DelegatingPasswordEncoder
``` java=
public class DelegatingPasswordEncoder implements PasswordEncoder {
  private final String idForEncode;
  private final PasswordEncoder passwordEncoderForEncode;
  private final Map<String, PasswordEncoder> idToPasswordEncoder;
  private PasswordEncoder defaultPasswordEncoderForMatches = new UnmappedIdPasswordEncoder();
  /*Some Code*/
  public DelegatingPasswordEncoder(String idForEncode, Map<String, PasswordEncoder> idToPasswordEncoder) {
    /*Some Code*/
    // 編碼器Key
    this.idForEncode = idForEncode;
    // 編碼器
    this.passwordEncoderForEncode = idToPasswordEncoder.get(idForEncode);
    // match用的其他編碼器
    this.idToPasswordEncoder = new HashMap<>(idToPasswordEncoder);
  }
}
```
建立自定義的DelegatingPasswordEncoder
``` java=
@Bean
public static PasswordEncoder passwordEncoder() {
    String idForEncode = "bcrypt";
    Map encoders = new HashMap<>();
    encoders.put(idForEncode, new BCryptPasswordEncoder());
    encoders.put("noop", NoOpPasswordEncoder.getInstance());
    encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
    encoders.put("sha256", new StandardPasswordEncoder());

    return new DelegatingPasswordEncoder(idForEncode, encoders);
}
```
DelegatingPasswordEncoder編碼後的密碼格式為:
```
{id}encodedPassword
```
id代表使用PaswordEncoder的Key，encodedPassword是編碼後的密碼
ex 使用BCryptPasswordEncoder加密後結果為
```
{bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG 
```
使用這種密碼格式後如果更改預設編碼器，舊密碼編碼的資料不需要更動
從版本4.0升到5.0有兩種方法:
1. 幫資料庫密碼加上前墜
2. 將DelegatingPasswordEncoder預設編碼器設為之前用的Encoder
``` java=
NoOpPasswordEncoder noOpPasswordEncoder = NoOpPasswordEncoder.getInstance();
delegatingPasswordEncoder.setDefaultPasswordEncoderForMatches(noOpPasswordEncoder);
```



