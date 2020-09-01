---
weight: 1
title: "Java工具类整理"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java工具类整理"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

Java工具类整理，收集和整理的Java相关工具类，大部分来源网络，不过我都测试过了，哈哈


## Byte[]与hex互相转换，用在某些数据传输场景下

```java
import java.util.Arrays;

/**
 * Byte[]与hex的相互转换
 * @explain
 * @author Marydon
 * @creationTime 2018年6月11日下午2:29:11
 * @version 1.0
 * @since
 * @email marydon20170307@163.com
 */
public class ByteUtils {

    // 16进制字符
    private static final char[] HEX_CHAR = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f' };
}

/**
 * 方法一：将byte类型数组转化成16进制字符串
 * @explain 字符串拼接
 * @param bytes
 * @return
 */
public static String toHexString(byte[] bytes) {
    StringBuilder sb = new StringBuilder();
    int num;
    for (byte b : bytes) {
        num = b < 0 ? 256 + b : b;
        sb.append(HEX_CHAR[num / 16]).append(HEX_CHAR[num % 16]);
    }
    return sb.toString();
}

/**
 * 方法二： byte[] to hex string
 * @explain 使用数组
 * @param bytes
 * @return
 */
public static String toHexString2(byte[] bytes) {
    // 一个byte为8位，可用两个十六进制位表示
    char[] buf = new char[bytes.length * 2];
    int a = 0;
    int index = 0;
    // 使用除与取余进行转换
    for (byte b : bytes) {
        if (b < 0)
            a = 256 + b;
        else
            a = b;

        // 偶数位用商表示
        buf[index++] = HEX_CHAR[a / 16];
        // 奇数位用余数表示
        buf[index++] = HEX_CHAR[a % 16];
    }
    // char[]-->String
    return new String(buf);
}

/**
 * 方法三： byte[]-->hexString
 * @explain 使用位运算
 * @param bytes
 * @return
 */
public static String toHexString3(byte[] bytes) {
    char[] buf = new char[bytes.length * 2];
    int index = 0;
    // 利用位运算进行转换，可以看作方法二的变型
    for (byte b : bytes) {
        buf[index++] = HEX_CHAR[b >>> 4 & 0xf];
        buf[index++] = HEX_CHAR[b & 0xf];
    }

    return new String(buf);
}

/**
 * 方法四：byte[]-->hexString
 * @param bytes
 * @return
 */
public static String toHexString4(byte[] bytes) {
    StringBuilder sb = new StringBuilder(bytes.length * 2);
    // 使用String的format方法进行转换
    for (byte b : bytes) {
        sb.append(String.format("%02x", new Integer(b & 0xff)));
    }

    return sb.toString();
}  

/**
 * 将16进制字符串转换为byte[]
 * @explain 16进制字符串不区分大小写，返回的数组相同
 * @param hexString
 *            16进制字符串
 * @return byte[]
 */
public static byte[] fromHexString(String hexString) {
    if (null == hexString || "".equals(hexString.trim())) {
        return new byte[0];
    }

    byte[] bytes = new byte[hexString.length() / 2];
    // 16进制字符串
    String hex;
    for (int i = 0; i < hexString.length() / 2; i++) {
        // 每次截取2位
        hex = hexString.substring(i * 2, i * 2 + 2);
        // 16进制-->十进制
        bytes[i] = (byte) Integer.parseInt(hex, 16);
    }

    return bytes;
}
```

原本程序

```java
import java.util.Arrays;
import java.lang.Throwable;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

import org.apache.axis.encoding.Base64;

public class aesEncryption {
    private static final String CYPHER_MODE = "AES/CBC/NoPadding";

    public static byte[] encrypt(byte[] key, byte[] initVector, byte[] value) {
        try {
            IvParameterSpec iv = new IvParameterSpec(initVector);
            SecretKeySpec skeySpec = new SecretKeySpec(key, "AES");

            Cipher cipher = Cipher.getInstance(CYPHER_MODE);
            cipher.init(Cipher.ENCRYPT_MODE, skeySpec, iv);
            int blockSize = cipher.getBlockSize();
            byte[] plaintext = padding(value, blockSize);
            return cipher.doFinal(plaintext);
        } catch (Exception ex) {
            ex.printStackTrace();
        }

        return null;
    }

    public static byte[] decrypt(byte[] key, byte[] initVector, byte[] encrypted) {
        try {
            IvParameterSpec iv = new IvParameterSpec(initVector);
            SecretKeySpec skeySpec = new SecretKeySpec(key, "AES");

            Cipher cipher = Cipher.getInstance(CYPHER_MODE);
            cipher.init(Cipher.DECRYPT_MODE, skeySpec, iv);

            return unpadding(cipher.doFinal(encrypted));
        } catch (Exception ex) {
            ex.printStackTrace();
        }

        return null;
    }

    private static byte[] padding(byte[] value, int blockSize) {
        int plaintextLength = value.length;
        if (plaintextLength % blockSize != 0) {
            plaintextLength = plaintextLength + (blockSize - (plaintextLength % blockSize));
        }
        byte[] plaintext = new byte[plaintextLength];
        System.arraycopy(value, 0, plaintext, 0, value.length);
        return plaintext;
    }

    private static byte[] unpadding(byte[] bytes) {
        int i = bytes.length - 1;
        while (i >= 0 && bytes[i] == 0)
        {
            --i;
        }

        return Arrays.copyOf(bytes, i + 1);
    }

    public static void main(String[] args) {
        try {
            byte[] key = "keyskeyskeyskeys".getBytes();
            byte[] iv = "keyskeyskeyskeys".getBytes();
            byte[] content = "123".getBytes("utf-8");
            byte[] cyphertext = encrypt(key, iv, content);
            String b64 = Base64.encode(cyphertext);
            System.out.println(b64);
            byte[] de_b64 = decrypt(key, iv, Base64.decode(b64));
            String plaintext = new String(de_b64);
            System.out.println(plaintext);
        } catch (Throwable t) {
            t.printStackTrace();
        }
    }
}
```

```python
from Crypto.Cipher import AES
import base64


class AESEncrypt:
    def __init__(self, key, iv):
        self.key = key
        self.iv = iv
        self.mode = AES.MODE_CBC

    def encrypt(self, text):
        cryptor = AES.new(self.key, self.mode, self.key)
        length = AES.block_size
        text_pad = self.padding(length, text)
        ciphertext = cryptor.encrypt(text_pad)
        cryptedStr = str(base64.b64encode(ciphertext), encoding='utf-8')
        return cryptedStr

    def padding(self, length, text):
        count = len(text.encode('utf-8'))
        if count % length != 0:
            add = length - (count % length)
        else:
            add = 0
        text1 = text + ('\0' * add)
        return text1

    def decrypt(self, text):
        base_text = base64.b64decode(text)
        cryptor = AES.new(self.key, self.mode, self.key)
        plain_text = cryptor.decrypt(base_text)
        ne = plain_text.decode('utf-8').rstrip('\0')
        return ne


if __name__ == '__main__':
    aes_encrypt = AESEncrypt(key='keyskeyskeyskeys', iv="keyskeyskeyskeys")  # 初始化key和IV
    text = '123'
    sign_data = aes_encrypt.encrypt(text)
    print(sign_data)
    data = aes_encrypt.decrypt(sign_data)
    print(data)
```


## 通过反射操作对象私有方法和私有变量

```java
public class ReflectionUtils {
 
    /**
     * 获取私有成员变量的值
     * @param instance
     * @param filedName
     * @return
     */
    public static Object getPrivateField(Object instance, String filedName) throws NoSuchFieldException, IllegalAccessException {
        Field field = instance.getClass().getDeclaredField(filedName);
        field.setAccessible(true);
        return field.get(instance);
    }
 
    /**
     * 设置私有成员的值
     * @param instance
     * @param fieldName
     * @param value
     * @throws NoSuchFieldException
     * @throws IllegalAccessException
     */
    public static void setPrivateField(Object instance, String fieldName, Object value) throws NoSuchFieldException, IllegalAccessException {
        Field field = instance.getClass().getDeclaredField(fieldName);
        field.setAccessible(true);
        field.set(instance, value);
    }
 
    /**
     * 访问私有方法
     * @param instance
     * @param methodName
     * @param classes
     * @param objects
     * @return
     * @throws NoSuchMethodException
     * @throws InvocationTargetException
     * @throws IllegalAccessException
     */
    public static Object invokePrivateMethod(Object instance, String methodName, Class[] classes, String objects) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Method method = instance.getClass().getDeclaredMethod(methodName, classes);
        method.setAccessible(true);
        return method.invoke(instance, objects);
    }
}
```

## restTemplate 响应泛型

```java
public static <T, A> T exchange(String url, HttpMethod method, ParameterizedTypeReference<T> responseBodyType, A requestBody) {
        RestTemplate restTemplate = new RestTemplate();
        // 请求头
        HttpHeaders headers = new HttpHeaders();
        MimeType mimeType = MimeTypeUtils.parseMimeType("application/json");
        MediaType mediaType = new MediaType(mimeType.getType(), mimeType.getSubtype(), StandardCharsets.UTF_8);
        // 请求体
        headers.setContentType(mediaType);
        // 发送请求
        HttpEntity<A> entity = new HttpEntity<>(requestBody, headers);
        ResponseEntity<T> resultEntity = restTemplate.exchange(url, method, entity, responseBodyType);
        return resultEntity.getBody();
    }

// 调用

ParameterizedTypeReference<ResponseBean<BasePushData>> responseBodyType = new ParameterizedTypeReference<ResponseBean<BasePushData>>() {};
//        String res = restTemplate.postForObject("http://localhost:8920/store", basePushData, String.class);
        ResponseBean<BasePushData> ret = exchange("http://localhost:8920/store", HttpMethod.POST, responseBodyType, basePushData);
```