缓存key序列话
=========== 
基于缓存key的序列化比较特殊，有些序列化器(包括序列化方法和反序列化方法)在进行反序列化时，无法指定反序列化的类型。
因此在序列化的时候就要在序列化字节流中加入类型(class)，这样在反序列化时无需外部指定类型，就可以正确的反序列化，常用的有jdk序列化器或者hession序列化器。
例如：spring boot redis的序列化器接口定义如下：  

```java
public interface RedisSerializer<T> {

	/**
	 * Serialize the given object to binary data.
	 * 
	 * @param t object to serialize
	 * @return the equivalent binary data
	 */
	byte[] serialize(T t) throws SerializationException;

	/**
	 * Deserialize an object from the given binary data.
	 * 
	 * @param bytes object binary representation
	 * @return the equivalent object instance
	 */
	T deserialize(byte[] bytes) throws SerializationException;
}

```
但如果key是二进制方式写入redis对调试和性能都没有好处，而且redis的key存储一般情况下都会使用简单的java基本类型对象，例如：String、Integer等，
几乎90%都是String，因此我想重新设置一个基于缓存key的序列化器。

重复造轮子，缓存key的序列化器
==========
设计的思路，在序列化时先在序列化流中(byte[])中写入类型，然后再根据类型调用对应的序列化器，序列化对象并追加到序列化流中。
反序列化的时候先读取类型，然后获取对应的序列化器，进行返序列化操作。为了简单和高效，这里
序列化字节流格式：类型+分隔符[:]+序列化字节数组，例如：string:xxxxxxxx。




