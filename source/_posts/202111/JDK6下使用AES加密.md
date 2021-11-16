title: JDK6下使用AES加密
date: 2021-11-16 17:53:00
toc: true
category:
- Java
tags:
- Java
- JDK
- jdk
- JDK1.6
- AES
- aes
- 加密
- 算法
- 使用
---

最近碰到老项目需要对明文进行加密，但是晚上找了一圈没有找到相关的算法。后来找到项目中已有的加密算法，特此记录。

![Snipaste20211116175447.png](https://b3logfile.com/file/2021/11/Snipaste_2021-11-16_17-54-47-2948345f.png)


<!-- more -->


代码如下：

```java
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import org.apache.commons.codec.binary.Base64;

public class AES {
    public AES() {
    }

    public static String Decrypt(String sSrc, String sKey) throws Exception {
        try {
            if (sKey == null) {
                System.out.print("16位skey不为空");
                return null;
            } else if (sKey.length() != 16) {
                System.out.print("skey长度不是16位");
                return null;
            } else {
                byte[] raw = sKey.getBytes("utf-8");
                SecretKeySpec skeySpec = new SecretKeySpec(raw, "AES");
                Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
                cipher.init(2, skeySpec);
                byte[] encrypted1 = (new Base64()).decode(sSrc);

                try {
                    byte[] original = cipher.doFinal(encrypted1);
                    String originalString = new String(original, "utf-8");
                    return originalString;
                } catch (Exception var8) {
                    System.out.println(var8.toString());
                    return null;
                }
            }
        } catch (Exception var9) {
            System.out.println(var9.toString());
            return null;
        }
    }

    public static void main(String[] args) throws Exception {
        String enString = "9HEhlcxA4eZk0Q6c42z6ZA==";
        String deString = enString.replace("InC9ReO0SE", "+");
        String sdeString = Decrypt(deString, "MTRjNzOANWJjYWEz");
        System.out.println("解密后的字串是：" + sdeString);
    }
}
```

使用实例参考 main 方法，运行结果为：

![Snipaste20211116175726.png](https://b3logfile.com/file/2021/11/Snipaste_2021-11-16_17-57-26-62aab1a4.png)
