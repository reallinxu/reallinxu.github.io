# Spring-boot 小知识点 #

## 注解 ##


## Bean ##
+ MappingJackson2HttpMessageConverter
  Spring Boot底层通过HttpMessageConverters依靠Jackson库将Java实体类输出为JSON格式。当有多个转换器可用时，根据消息对象类型和需要的内容类型选择最适合的转换器使用。  

  ![avatar](/imgs/MessageConverter.png)

+ 