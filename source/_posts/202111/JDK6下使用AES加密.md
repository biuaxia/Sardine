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
import org.apache.commons.codec.binary.Base64;
import org.bouncycastle.jce.provider.BouncyCastleProvider;

import javax.crypto.Cipher;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.Key;
import java.security.Security;
import java.security.spec.AlgorithmParameterSpec;
import java.util.Date;

public class AES {

    public AES() {
    }

    static {
        if (Security.getProvider("BC") == null) {
            Security.addProvider(new BouncyCastleProvider());
        }

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
                    return new String(original, "utf-8");
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

    private static Key getKey(String str) throws Exception {
        DESKeySpec dks = new DESKeySpec(str.getBytes("UTF-8"));
        SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES", "BC");
        return keyFactory.generateSecret(dks);
    }

    private static AlgorithmParameterSpec getIv(String iv) throws Exception {
        return new IvParameterSpec(iv.getBytes("UTF-8"));
    }

    public static String encrypt(String key, String iv, String str) throws Exception {
        Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding", "BC");
        cipher.init(1, getKey(key), getIv(iv));
        byte[] data = cipher.doFinal(str.getBytes("UTF-8"));
        return Base64encode(data);
    }

    public static String decrypt(String key, String iv, String str) throws Exception {
        Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding", "BC");
        cipher.init(2, getKey(key), getIv(iv));
        byte[] sss = cipher.doFinal(Base64decode(str));
        return new String(sss, "UTF-8");
    }

    public static String aesDecrypt(String key, String iv, String str) throws Exception {
        str = str.replace("_", "+");
        byte[] encrypted1 = Base64decode(str);
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding", "BC");
        cipher.init(2, new SecretKeySpec(key.getBytes("UTF-8"), "AES"), getIv(iv));
        byte[] original = cipher.doFinal(encrypted1);
        return new String(original, "UTF-8");
    }

    public static String aesEncrypt(String key, String iv, String str) throws Exception {
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding", "BC");
        cipher.init(1, new SecretKeySpec(key.getBytes("UTF-8"), "AES"), getIv(iv));
        byte[] encrypted = cipher.doFinal(str.getBytes("UTF-8"));
        String s = Base64encode(encrypted);
        s = s.replace("+", "_");
        return s;
    }

    private static String Base64encode(byte[] data) throws Exception {
        return new String(Base64.encodeBase64(data), "UTF-8");
    }

    private static byte[] Base64decode(String str) throws Exception {
        return Base64.decodeBase64(str.getBytes("UTF-8"));
    }

    public static String getTime() {
        return String.valueOf(new Date().getTime());
    }

    public static void main(String[] args) throws Exception {
        // String enString = "9HEhlcxA4eZk0Q6c42z6ZA==";
        // String deString = enString.replace("InC9ReO0SE", "+");
        // String sdeString = Decrypt(deString, "MTRjNzOANWJjYWEz");
        // System.out.println("解密后的字串是：" + sdeString);
        // for (int i = 0; i < 20; i++) {
        Date now = new Date(1637117714679L);
        System.out.println(now.getTime());
        System.out.println(aesEncrypt("3tcac4konflddsrh", "xapsk6al0opqq4eo", String.format("%s、%s", "313015", now.getTime())));
        System.out.println(aesDecrypt("3tcac4konflddsrh", "xapsk6al0opqq4eo", "yKTNKqAcPD6cFTso4Qqbs_W7aA3RNKBu3viOx7ESvBc="));
        System.out.println("------------------------------ 分割 ------------------------------ ");
        // }

        // String plain = decrypt("3tcac4konflddsrh", "xapsk6al0opqq4eo", "i3pGlVlwnTpSsfeBssi1BbWvCy+d7NL0E1R7/lZkmVc=");
        // String[] split = plain.split("、");
        // System.out.println(plain);
        // System.out.println(split[0]);
        // System.out.println(split[1]);
        // Date expireTime = new Date(Long.parseLong(split[1]));
        // System.out.println(expireTime);
        //
        // System.out.println(DateUtil.parse("2021-11-17 12:00:21", DateUtil.YEAR_MONTH_DAY_HOUR_MINUTE_SECOND_PATTERN).getTime());
        // System.out.println(DateUtil.parse("2021-11-17 12:00:31", DateUtil.YEAR_MONTH_DAY_HOUR_MINUTE_SECOND_PATTERN).getTime());
        //
        // System.out.println(System.currentTimeMillis() - Long.parseLong(split[1]) <= 10000);
    }
    
}
```

使用实例参考 main 方法，运行结果为：

![Snipaste20211116175726.png](https://b3logfile.com/file/2021/11/Snipaste_2021-11-16_17-57-26-62aab1a4.png)
