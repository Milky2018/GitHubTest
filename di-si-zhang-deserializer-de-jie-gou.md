---
description: Lei Zhengyu - 2018年1月10日
---

# 第五章：Deserializer的结构

导言部分粗略地介绍了 parse 系列方法的使用方式和效果，现在有了第二章的基础，阅读 Deserializer 源码应当更加容易。

## 4.1 ObjectDeserializer接口与内部注册反序列化方案

所有的反序列化器都要实现 ObjectDeserializer 接口：

{% code-tabs %}
{% code-tabs-item title="ObjectDeserializer.java" %}
```java
public interface ObjectDeserializer {
    <T> T deserialze(DefaultJSONParser parser, Type type, Object fieldName);
    
    int getFastMatchToken();
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

实现这个接口的诸多序列化器中有 Deserializer 为尾缀的，也有 Codec 为尾缀的。你应该猜到了，这些以 Codec 为尾缀的“解码器”既能够序列化又能够反序列化。如我们在第三章提到的 FloatCodec:

{% code-tabs %}
{% code-tabs-item title="FloatCodec.java" %}
```java
public class FloatCodec implements ObjectSerializer, ObjectDeserializer {
    ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

它同时实现了 ObjectSerializer 和 ObjectDeserializer 接口。不过这不意味着同一个类型同时满足指内部注册序列化方案和内部注册反序列化方案就能为效率或编码带来多大的好处。因为我们说过，它是一个 Singleton，而且只有一个共享字段 decimalFormat, 共用价值几乎没有。

如果读者需要更多的内部注册反序列化方案的信息，以下是 ParserConfig 中的 initDeserializers\(\) 方法。同 SerializeConfig, ParserConfig 也维护着一个单例，内部是一个 IdentityHashMap.

{% code-tabs %}
{% code-tabs-item title="ParserConfig.java" %}
```java
public class ParserConfig {
    ...
    private void initDeserializers() {
        deserializers.put(SimpleDateFormat.class, MiscCodec.instance);
        deserializers.put(java.sql.Timestamp.class, SqlDateDeserializer.instance_timestamp);
        deserializers.put(java.sql.Date.class, SqlDateDeserializer.instance);
        deserializers.put(java.sql.Time.class, TimeDeserializer.instance);
        deserializers.put(java.util.Date.class, DateCodec.instance);
        deserializers.put(Calendar.class, CalendarCodec.instance);
        deserializers.put(XMLGregorianCalendar.class, CalendarCodec.instance);

        deserializers.put(JSONObject.class, MapDeserializer.instance);
        deserializers.put(JSONArray.class, CollectionCodec.instance);

        deserializers.put(Map.class, MapDeserializer.instance);
        deserializers.put(HashMap.class, MapDeserializer.instance);
        deserializers.put(LinkedHashMap.class, MapDeserializer.instance);
        deserializers.put(TreeMap.class, MapDeserializer.instance);
        deserializers.put(ConcurrentMap.class, MapDeserializer.instance);
        deserializers.put(ConcurrentHashMap.class, MapDeserializer.instance);

        deserializers.put(Collection.class, CollectionCodec.instance);
        deserializers.put(List.class, CollectionCodec.instance);
        deserializers.put(ArrayList.class, CollectionCodec.instance);

        deserializers.put(Object.class, JavaObjectDeserializer.instance);
        deserializers.put(String.class, StringCodec.instance);
        deserializers.put(StringBuffer.class, StringCodec.instance);
        deserializers.put(StringBuilder.class, StringCodec.instance);
        deserializers.put(char.class, CharacterCodec.instance);
        deserializers.put(Character.class, CharacterCodec.instance);
        deserializers.put(byte.class, NumberDeserializer.instance);
        deserializers.put(Byte.class, NumberDeserializer.instance);
        deserializers.put(short.class, NumberDeserializer.instance);
        deserializers.put(Short.class, NumberDeserializer.instance);
        deserializers.put(int.class, IntegerCodec.instance);
        deserializers.put(Integer.class, IntegerCodec.instance);
        deserializers.put(long.class, LongCodec.instance);
        deserializers.put(Long.class, LongCodec.instance);
        deserializers.put(BigInteger.class, BigIntegerCodec.instance);
        deserializers.put(BigDecimal.class, BigDecimalCodec.instance);
        deserializers.put(float.class, FloatCodec.instance);
        deserializers.put(Float.class, FloatCodec.instance);
        deserializers.put(double.class, NumberDeserializer.instance);
        deserializers.put(Double.class, NumberDeserializer.instance);
        deserializers.put(boolean.class, BooleanCodec.instance);
        deserializers.put(Boolean.class, BooleanCodec.instance);
        deserializers.put(Class.class, MiscCodec.instance);
        deserializers.put(char[].class, new CharArrayCodec());

        deserializers.put(AtomicBoolean.class, BooleanCodec.instance);
        deserializers.put(AtomicInteger.class, IntegerCodec.instance);
        deserializers.put(AtomicLong.class, LongCodec.instance);
        deserializers.put(AtomicReference.class, ReferenceCodec.instance);

        deserializers.put(WeakReference.class, ReferenceCodec.instance);
        deserializers.put(SoftReference.class, ReferenceCodec.instance);

        deserializers.put(UUID.class, MiscCodec.instance);
        deserializers.put(TimeZone.class, MiscCodec.instance);
        deserializers.put(Locale.class, MiscCodec.instance);
        deserializers.put(Currency.class, MiscCodec.instance);

        deserializers.put(Inet4Address.class, MiscCodec.instance);
        deserializers.put(Inet6Address.class, MiscCodec.instance);
        deserializers.put(InetSocketAddress.class, MiscCodec.instance);
        deserializers.put(File.class, MiscCodec.instance);
        deserializers.put(URI.class, MiscCodec.instance);
        deserializers.put(URL.class, MiscCodec.instance);
        deserializers.put(Pattern.class, MiscCodec.instance);
        deserializers.put(Charset.class, MiscCodec.instance);
        deserializers.put(JSONPath.class, MiscCodec.instance);
        deserializers.put(Number.class, NumberDeserializer.instance);
        deserializers.put(AtomicIntegerArray.class, AtomicCodec.instance);
        deserializers.put(AtomicLongArray.class, AtomicCodec.instance);
        deserializers.put(StackTraceElement.class, StackTraceElementDeserializer.instance);

        deserializers.put(Serializable.class, JavaObjectDeserializer.instance);
        deserializers.put(Cloneable.class, JavaObjectDeserializer.instance);
        deserializers.put(Comparable.class, JavaObjectDeserializer.instance);
        deserializers.put(Closeable.class, JavaObjectDeserializer.instance);

        deserializers.put(JSONPObject.class, new JSONPDeserializer());
    }
    ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 4.2 ParserConfig类

在第一章我展示了一个简单的 parseObject\(\) 方法。同 toJSONString\(\)一样，parseObect\(\) 也被重载了很多次，以下是两个个内部最终被调用的完整 parseObject\(\) 方法：

{% code-tabs %}
{% code-tabs-item title="JSON.java" %}
```java
public abstract class JSON implements JSONStreamAware, JSONAware {    
    ...
    public static <T> T parseObject(String input, Type clazz, ParserConfig config, ParseProcess processor,
                                          int featureValues, Feature... features) {
        if (input == null) {
            return null;
        }

        if (features != null) {
            for (Feature feature : features) {
                featureValues |= feature.mask;
            }
        }

        DefaultJSONParser parser = new DefaultJSONParser(input, config, featureValues);

        if (processor != null) {
            if (processor instanceof ExtraTypeProvider) {
                parser.getExtraTypeProviders().add((ExtraTypeProvider) processor);
            }

            if (processor instanceof ExtraProcessor) {
                parser.getExtraProcessors().add((ExtraProcessor) processor);
            }

            if (processor instanceof FieldTypeResolver) {
                parser.setFieldTypeResolver((FieldTypeResolver) processor);
            }
        }

        T value = (T) parser.parseObject(clazz, null);

        parser.handleResovleTask(value);

        parser.close();

        return (T) value;
    }
    ...
    public static <T> T parseObject(char[] input, int length, Type clazz, Feature... features) {
        if (input == null || input.length == 0) {
            return null;
        }

        int featureValues = DEFAULT_PARSER_FEATURE;
        for (Feature feature : features) {
            featureValues = Feature.config(featureValues, feature, true);
        }

        DefaultJSONParser parser = new DefaultJSONParser(input, length, ParserConfig.getGlobalInstance(), featureValues);
        T value = (T) parser.parseObject(clazz);

        parser.handleResovleTask(value);

        parser.close();

        return (T) value;
    }
    ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

到目前为止，反序列化步骤和序列化步骤还是相似的：创建 ParserConfig 对象，包括初始化内部注册反序列化方案和 features 的配置；添加反序列化拦截器；根据具体类型，将反序列化器实例的查找委托给 parser 对象；解析对象内部引用。

而不指定 ParserConfig 对象时，就使用第四章中提到的 Singleton：

{% code-tabs %}
{% code-tabs-item title="JSON.java" %}
```java
public abstract class JSON implements JSONStreamAware, JSONAware {    
    ...    
    public static <T> T parseObject(InputStream is, //
                                    Charset charset, //
                                    Type type, //
                                    Feature... features) throws IOException {
        return (T) parseObject(is, charset, type, ParserConfig.global, features);
    }
    ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

parseObject\(\) 方法内部委托 DefaultJSONParser 对象进行解析，以下是一个典型的 parseObject\(\) 方法：

{% code-tabs %}
{% code-tabs-item title="DefaultJSONParser.java" %}
```java
public class DefaultJSONParser implements Closeable {
    ...
    public <T> T parseObject(Type type, Object fieldName) {
        int token = lexer.token();
        if (token == JSONToken.NULL) {
            lexer.nextToken();
            return null;
        }

        if (token == JSONToken.LITERAL_STRING) {
            if (type == byte[].class) {
                byte[] bytes = lexer.bytesValue();
                lexer.nextToken();
                return (T) bytes;
            }

            if (type == char[].class) {
                String strVal = lexer.stringVal();
                lexer.nextToken();
                return (T) strVal.toCharArray();
            }
        }

        ObjectDeserializer derializer = config.getDeserializer(type);

        try {
            if (derializer.getClass() == JavaBeanDeserializer.class) {
                return (T) ((JavaBeanDeserializer) derializer).deserialze(this, type, fieldName, 0);
            } else {
                return (T) derializer.deserialze(this, type, fieldName);
            }
        } catch (JSONException e) {
            throw e;
        } catch (Throwable e) {
            throw new JSONException(e.getMessage(), e);
        }
    }
    ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

其中第24行委托了 config 配置具体的序列化器。在第三章，我们曾经将 JavaBean 类型与其它类型的序列化分开讨论，因为 JavaBean 会让解析变得复杂（触及类型信息）。在这里，因为泛型，反序列化要更加复杂。ParserConfig.getDeserializer\(\) 方法显然要对泛型进行特殊处理：

{% code-tabs %}
{% code-tabs-item title="ParserConfig.java" %}
```java
public class ParserConfig {
    ...    
    public ObjectDeserializer getDeserializer(Type type) {
        ObjectDeserializer derializer = this.deserializers.get(type);
        if (derializer != null) {
            return derializer;
        }

        if (type instanceof Class<?>) {
            return getDeserializer((Class<?>) type, type);
        }

        if (type instanceof ParameterizedType) {
            Type rawType = ((ParameterizedType) type).getRawType();
            if (rawType instanceof Class<?>) {
                return getDeserializer((Class<?>) rawType, type);
            } else {
                return getDeserializer(rawType);
            }
        }

        if (type instanceof WildcardType) {
            WildcardType wildcardType = (WildcardType) type;
            Type[] upperBounds = wildcardType.getUpperBounds();
            if (upperBounds.length == 1) {
                Type upperBoundType = upperBounds[0];
                return getDeserializer(upperBoundType);
            }
        }

        return JavaObjectDeserializer.instance;
    }
    ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

简要说明这段代码的具体逻辑：首先从已经注册的方案中查找class的反序列化器；如果找不到且目标是引用类型，用 getDeserializer\(\(Class&lt;?&gt;\) type, type\) 方法递归查找；如果又找不到，而且目标是泛型，先获取泛型的原始类型，判断其是否为引用类型，再用不同方式递归查找；接下来是通配符和限定类型的判断；最后，如果都不满足，就返回默认的反序列化器 JavaObjectDeserializer。

重载的 getDeserializer\(\(Class&lt;?&gt;\) type, type\) 方法体很庞大，但有了序列化 Bean 的代码阅读经验，我们知道，再初次遇见 Bean 时，该方法一定会返回一个由名为 createJavaBeanDeserializer\(\) 的方法生产的 JavaBeanDeserializer. 下一章我们将具体分析这个过程。

