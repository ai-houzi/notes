## Spring security 5的一些坑

* 报错There is no PasswordEncoder mapped for the id "null"

  spring security 5 对 PasswordEncoder 做了相关的重构，原先默认配置的 PlainTextPasswordEncoder（明文密码）被移除了，想要做到明文存储密码，只能使用一个过期的类来过渡

  ```java
  //加入
  //已过期
  @Bean
  PasswordEncoder passwordEncoder(){
      return NoOpPasswordEncoder.getInstance();
  }
  
  ```

  spring security 提供了多种类来进行密码编码，并作为了相关配置的默认配置，只不过没有暴露为全局的 Bean。使用明文存储的风险在文章一开始就已经强调过，NoOpPasswordEncoder 只能存在于 demo 中。

  ```java
  //
  @Bean
  PasswordEncoder passwordEncoder(){
      return new BCryptPasswordEncoder();
  }
  
  //加密方式与对应的类
  bcrypt - BCryptPasswordEncoder (Also used for encoding)
  ldap - LdapShaPasswordEncoder
  MD4 - Md4PasswordEncoder
  MD5 - new MessageDigestPasswordEncoder("MD5")
  noop - NoOpPasswordEncoder
  pbkdf2 - Pbkdf2PasswordEncoder
  scrypt - SCryptPasswordEncoder
  SHA-1 - new MessageDigestPasswordEncoder("SHA-1")
  SHA-256 - new MessageDigestPasswordEncoder("SHA-256")
  sha256 - StandardPasswordEncoder
  ```

  参考：https://blog.csdn.net/dream_an/article/details/79381459

  https://www.cnkirito.moe/spring-security-6/

