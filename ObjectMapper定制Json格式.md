#利用ObjectMapper定制化全局的一次尝试
##背景
* 工程较大较复杂的情况下需要修改DateTime的Jsonformat格式，使其返回ISO8061的格式，而不是返回时间戳
* 利用JodaTime的默认转换格式就是ISO8061格式，但是由于部分数据存储的原因，一部分前端还是会收到时间戳格式的时间
* 目前部分的数据类中有加上@Jsonformat注解，单个的更改返还给前端的DateTime格式，还存在部分遗漏
* 关注到ObjectMapper这个类，首先这个类拥有默认的Json格式转换 **Jackson20ObjectMapperBuilder**，然后我们可以通过在其之后加上Customizer来实现添加定制化,也就是**Jackson2ObjectMapperBuilderCustomizer**


##实践

* 第一步，需要在拥有@Configuration注解的类下添加一个新的Bean

`Jackson2ObjectMapperBuilderCustomizer addCustomBigDecimalDeserialization()`

* 第二步，需要重写该类下的customize方法

```
 @Override
public void customize(Jackson2ObjectMapperBuilder jacksonObjectMapperBuilder) {
    jacksonObjectMapperBuilder.deserializerByType(BigDecimal.class, bigDecimalDeserializer);
            }   
```

* 第三步，定义一个@component，继承serializer和deserializer

```
@Component
public class BigDecimalDeserializer extends StdDeserializer<BigDecimal> {

    public BigDecimalDeserializer() {
        super(BigDecimal.class);
    }

    @Override
    public BigDecimal deserialize(JsonParser p, DeserializationContext ctxt) throws IOException {
        ...
    }

    ...

}
```


##测试

* 在Controller层写了新的测试接口，以时间戳的形式实例一个DateTime利用postman测试接口返回值
* 能够正常返回ISO8061格式的时间格式
* 但是由于没有推到线上测试，害怕跟后端微服务交换数据会出现问题，所以该方案最终没有实现
        
          

