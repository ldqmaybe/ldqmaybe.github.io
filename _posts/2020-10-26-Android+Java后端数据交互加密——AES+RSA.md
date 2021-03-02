---
layout:     post
title:      Android+Java后端数据交互加密——AES+RSA
subtitle:   
date:       2020-10-26
author:     dingqiang.l
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - Android
    - Java
---

早前公司新开发了个项目，在数据加密部分讨论后选择了AES+RSA方式（对称+非对称）。参考了网上部分比较优秀的例子后，最终使用了一下实现方式，有需要的朋友可以在文章的最后获取完整demo。话不多说，直接开始。

下面会先简单介绍加解密思路，然后再分别介绍Java后端和Android端的实现，最后再简单搭建个环境实现两端数据加解密。

## 加密解密流程
### 一、加密：
        1、客户端和服务端双方约定好公私钥对，即公钥PUBLIC_KEY，私钥PRIVATE_KEY
        2、客户端使用私钥PRIVATE_KEY对数据进行签名
        3、客户端生成一个随机密钥KEY，并且使用公钥PUBLIC_KEY对其加密得到加密后的密钥ENCRYPT_KEY
        4、客户端使用加密后的密钥ENCRYPT_KEY对数据（包括签名信息）进行AES加密得到加密数据
        5、将加密后的数据以及加密后的密钥发送给服务端
        6、服务端加密同理

### 二、解密：
        1、服务端用私钥PRIVATE_KEY对加密后的密钥进行RSA解密，得到解密后的密钥DECRPTY_KEY
        2、服务端使用解密后的密钥DECRPTY_KEY对加密数据进行解密
        3、服务端利用SHA256withRSA算法，用DECRPTY_KEY对解密后的签名信息验签
        4、客户端解密同理

### 三、具体实现：
下面用到的Base64编码解密和16进制的转换都是使用了apache的一个工具库，但是需要注意的是，Android端不能直接使用，否则运行时程序会直接崩掉，具体原因和解决方法网上有很多，这里展开。我是直接下载了整个jar包，只提取用到的类。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201020161912344.png#pic_center =250x250)
#### 1、生成公私钥对及签名验签：
SHA256withRSA.java
```java
public class SHA256withRSA {

    public static final String KEY_ALGORITHM = "RSA";
    public static final String SIGNATURE_ALGORITHM = "SHA256withRSA";

    /**
     * 还原公钥
     *
     * @param publicKey 公钥串
     * @return PublicKey
     */
    public static PublicKey restorePublicKey(String publicKey) throws EncryptException {
        byte[] prikeyByte;
        PublicKey pubTypeKey = null;
        try {
            prikeyByte = Base64.decodeBase64(publicKey.getBytes(StandardCharsets.UTF_8));
            X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(prikeyByte);
            KeyFactory factory = KeyFactory.getInstance(KEY_ALGORITHM);

            pubTypeKey = factory.generatePublic(x509EncodedKeySpec);
            return pubTypeKey;
        } catch (Exception e) {
            e.printStackTrace();
            throw new EncryptException("-10000", e);
        }
    }

    /**
     * 还原私钥
     *
     * @param privateKey 私钥串
     * @return PrivateKey
     */
    public static PrivateKey restorePrivateKey(String privateKey) throws EncryptException {
        byte[] prikeyByte;
        PrivateKey priTypeKey;
        try {
            prikeyByte = Base64.decodeBase64(privateKey.getBytes(StandardCharsets.UTF_8));
            PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(prikeyByte);
            KeyFactory factory = KeyFactory.getInstance(KEY_ALGORITHM);
            priTypeKey = factory.generatePrivate(pkcs8EncodedKeySpec);
            return priTypeKey;
        } catch (Exception e) {
            e.printStackTrace();
            throw new EncryptException("-10001", e);
        }
    }


    /**
     * 签名
     *
     * @param privateKey 私钥
     * @param plainText  明文
     * @return 签名后的签名串
     */
    public static String sign(String privateKey, String plainText) throws EncryptException {
        try {
            PrivateKey restorePrivateKey = restorePrivateKey(privateKey);
            Signature sign = Signature.getInstance(SIGNATURE_ALGORITHM);
            sign.initSign(restorePrivateKey);
            sign.update(plainText.getBytes(StandardCharsets.UTF_8));
            byte[] signByte = sign.sign();
            return Base64.encodeBase64String(signByte);
        } catch (Exception e) {
            e.printStackTrace();
            throw new EncryptException("-10003", e);
        }
    }

    /**
     * 验签
     *
     * @param publicKey  公钥
     * @param plainText  明文
     * @param signedText 签名
     */
    public static boolean verifySign(String publicKey, String plainText, String signedText) throws EncryptException {
        try {
            PublicKey publicTypeKey = restorePublicKey(publicKey);
            Signature verifySign = Signature.getInstance(SIGNATURE_ALGORITHM);
            verifySign.initVerify(publicTypeKey);
            verifySign.update(plainText.getBytes(StandardCharsets.UTF_8));
            return verifySign.verify(Base64.decodeBase64(signedText.getBytes(StandardCharsets.UTF_8)));
        } catch (Exception e) {
            e.printStackTrace();
            throw new EncryptException("-10002", e);
        }
    }


    /**
     * 生成密钥对
     *
     * @return
     */
    public static Map<String, String> generateKeyBytes() {
        try {
            KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(KEY_ALGORITHM);
            keyPairGenerator.initialize(2048);
            KeyPair keyPair = keyPairGenerator.generateKeyPair();
            RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
            RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
            Map<String, String> keyMap = new HashMap<>();
            keyMap.put("PUBLIC_KEY", Base64.encodeBase64String(publicKey.getEncoded()));
            keyMap.put("PRIVATE_KEY", Base64.encodeBase64String(privateKey.getEncoded()));
            return keyMap;
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            System.out.println("签名验证失败");
        }
        return null;
    }
    public static void main(String[] args) throws EncryptException {
        Map<String, String> map = generateKeyBytes();
        String public_key = map.get("PUBLIC_KEY");
        String private_key = map.get("PRIVATE_KEY");
        System.out.println(public_key);
        System.out.println(private_key);
	}
}
```
执行main方法获取公私钥对，如下

```java
公钥：MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxURxPBkE0A2X3aum9T2ovkLkjk0fIlCBgLyBO8dOj8jLvb2/x+fPtEsrQC6XHF5d1gQRTg7BeWKo/j+ufO61Lis230evVLrWPRsCJInf9rWro9/0/x8OxHWN2/zHp8bq4l6kX1K8zJ6nz/7dM1oVVYNSt3O6buY5DVKO4hJiagVlQlLhrfp4H6e2LwRxE+gBqMZy4zH3gSekgcuX4URtrikswaRKAfE20T7iarOBNgAF2Wl+nQ0P0JONX1OtCFeadxXNW7jrRsplcr9NjaJo3Q0UPhTHLWCC8GljsZEoSmEdyV/fSzdVsQYpe8QrvVnKxdPya9JZSPJkXqAeKUKvKwIDAQAB
私钥：MIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQDFRHE8GQTQDZfdq6b1Pai+QuSOTR8iUIGAvIE7x06PyMu9vb/H58+0SytALpccXl3WBBFODsF5Yqj+P6587rUuKzbfR69UutY9GwIkid/2tauj3/T/Hw7EdY3b/MenxuriXqRfUrzMnqfP/t0zWhVVg1K3c7pu5jkNUo7iEmJqBWVCUuGt+ngfp7YvBHET6AGoxnLjMfeBJ6SBy5fhRG2uKSzBpEoB8TbRPuJqs4E2AAXZaX6dDQ/Qk41fU60IV5p3Fc1buOtGymVyv02NomjdDRQ+FMctYILwaWOxkShKYR3JX99LN1WxBil7xCu9WcrF0/Jr0llI8mReoB4pQq8rAgMBAAECggEAdnx5lyNf5KeFhDvJ+Jukc0MyjNZ90NqSLoULCqDX6z0sQzdpreTquNw9ijtxwDReIGIpEr2CMCq2XqBZaejnImgKeWpRQY9Hh0RlsRSvVTwhcDjgqyw4boA9SNk4AupTwswd5rOHe1AAc8odiu6cydQrJs14OYxG0F26PMfWHN86yLGUHYXnrpesUePehoCyJoG8bzzsaiV+reZiH8ChjdbafWx4twkKbslYyIEf8eEJUNzGg1yp8NbMbbAaBXRh+Tyak+XuyIIdW2Rt49HKGEmJu3aCpJK5Ba47HRsdSMa8+6S2qHW9CHAnFNwKAhyUXKzlDnyLxSmGUqEI5bIUwQKBgQDkWxSNAMNeaduXcfFmt2abzh8E74TqDsD68au2jornycvXhpkUuVevMhGOtBwOc5d70LjZ02XmCDAOMOCo5SzYWeT4rnYw3iblCfxmP6lI7Dge737UO7+EYHetNS7qkO5WGfsKfcqx6K0glXSFE+qye0gcmyk4AsD0o4cVlRCHmQKBgQDdJehttSLhSItabf3jgfGxxFxwarSZRq+MEUZ1TPvZZ9xa86tO27H/wPhdt2lgD+WDi1EZiTJT3O9CfQ8ZiVU3WMwUzA61oeOI9TknMUGWFMJRwLtecCZ1umCT5F8n7PsvGYteVCwdxbZz6cAq0rNj+4+LPPHmwipj3h3FZNuXYwKBgQCxJVbT63ujiksnOOUj4bJfu46krYpWaAucoE2s2Pc4yHqxP2ERipZS+mxUX7REIbep/Ujo8e1ifYeJ+rDNVLttOo89u1lEn5FcrFp4l3ojb9w5Y2DoE1GGx68PVuqGXNgHQzBT+zF6wh1L4aT6d3Dh9HEEf/mB0eEN5q2sOG8SQQKBgQCAodQYPAwVzgSAjQnok2TqabT7DpYNsbfaWRIKmMTFKExb+u/h5pgakzvkBxMb9SMi6J47pDnJ3fCtU+C8kc0nbFcIocjMjWWz/C9KRLRJf7mno9tYixNT1xzl6SgQKR/RvaH7NCqVBrOhqI1GW1hNB73u13w9JSNTA5d9gbTY0QKBgQC1/cKZLr0PV9KcEUwErwQyDrXyp4oaZ3+Mb5bIX5QtgR910RRFH9aRZLHpp/YWLVP8DV6IdEzldTsNbF4AegVkUz7JxyX8hCdXmxhbhyi6s4QY1+YobrVavpWaZvi8PS+NzFGf1Pfi0AUnrP3cYS3nJFf6qB8qmFI8tv7ldkz0KA==
```
#### 2、RSA随机密钥对称加解密
注意这里使用的模式和填充分别是ECB和PKCS1Padding
```java
    /**
     * 加密
     *
     * @param data   加密之前的数据
     * @param pubKey 公钥
     * @return 加密之后的数据
     */
    public static String encrypt(String data, String pubKey) throws EncryptException {
        try {
            // base64编码的公钥
            byte[] decoded = Base64.decodeBase64(pubKey);
            KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);
            RSAPublicKey publicKey = (RSAPublicKey) keyFactory.generatePublic(new X509EncodedKeySpec(decoded));

            Cipher cipher = Cipher.getInstance(PADDING);
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);

            int splitLength = publicKey.getModulus().bitLength() / 8 - 11;
            byte[][] arrays = RsaUtil.splitBytes(data.getBytes(), splitLength);
            StringBuilder stringBuffer = new StringBuilder();
            for (byte[] array : arrays) {
                stringBuffer.append(RsaUtil.bytesToHexString(cipher.doFinal(array)));
            }
            return stringBuffer.toString();
        }catch (Exception e) {
            System.out.println("RSA 加密失败");
            e.printStackTrace();
            throw new EncryptException("-10004",e);
        }
    }
    /**
     * 解密
     *
     * @param data   解密之前的数据
     * @param priKey 秘钥
     * @return 解密之后的数据
     */
    public static String decrypt(String data, String priKey) throws EncryptException {
        try {
            KeySpec keySpec = new PKCS8EncodedKeySpec(Base64.decodeBase64(priKey));
            KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);
            RSAPrivateKey privateKey = (RSAPrivateKey) keyFactory.generatePrivate(keySpec);

            Cipher cipher = Cipher.getInstance(PADDING);
            cipher.init(Cipher.DECRYPT_MODE, privateKey);

            int splitLength = privateKey.getModulus().bitLength() / 8;
            byte[] contentBytes = hexStringToBytes(data);
            byte[][] arrays = RsaUtil.splitBytes(contentBytes, splitLength);
            StringBuilder stringBuffer = new StringBuilder();
            for (byte[] array : arrays) {
                stringBuffer.append(new String(cipher.doFinal(array)));
            }
            return stringBuffer.toString();
        } catch (Exception e) {
            System.out.println("RSA 解密失败");
            e.printStackTrace();
            throw new EncryptException("-10004",e);
        }
    }
```
#### 3、AES对数据非对称加解密
注意这里的模式和填充是AES/CBC/PKCS5Padding
```java
   /**
     * aes解密
     *
     * @param content 要解密的密文
     * @return 解密后的明文
     * @throws EncryptException 自定义异常
     */
    public static String decrypt(String content,String key) throws EncryptException {
        try {
            String ivStr = content.substring(0, 32);
            String message = content.substring(32);

            SecretKeySpec sKeySpec = new SecretKeySpec(Hex.decodeHex(key.toCharArray()), "AES");
            Cipher cipher = Cipher.getInstance(AES_PADDING);
            IvParameterSpec iv = new IvParameterSpec(Hex.decodeHex(ivStr.toCharArray()));
            cipher.init(Cipher.DECRYPT_MODE, sKeySpec, iv);

            byte[] original = cipher.doFinal(Base64.decodeBase64(message));
            return new String(original, ENCODING_CODE);
        } catch (Exception e) {
            e.printStackTrace();
            throw new EncryptException("-1006", e);
        }
    }

    /**
     * aes加密
     *
     * @param content 要加密的密文
     * @return 加密后的密文
     * @throws EncryptException 自定义异常
     */
    public static String encrypt(String content,String key) throws EncryptException {
        try {
            Cipher cipher = Cipher.getInstance(AES_PADDING);
            SecretKeySpec sKeySpec = new SecretKeySpec(Hex.decodeHex(key.toCharArray()), "AES");
            IvParameterSpec iv = getSecretIv(16);
            cipher.init(Cipher.ENCRYPT_MODE, sKeySpec, iv);
            byte[] encrypted = cipher.doFinal(content.getBytes(ENCODING_CODE));
            return new String(Hex.encodeHex(iv.getIV())).concat(Base64.encodeBase64String(encrypted));
        } catch (Exception e) {
            throw new EncryptException("-1007", e);
        }
    }
```
到此，基本工作已经完成，先简单验证下。
```java
    public static void main(String[] args) throws EncryptException {
   
        JsonObject busDataJson = new JsonObject();
        busDataJson.addProperty("plainText", plainText);
        //私钥PRIVATE_KEY对数据进行签名
        String signInfo = SHA256withRSA.sign(Encrypt.PRIVATE_KEY, plainText);
        busDataJson.addProperty("signInfo", signInfo);
        System.out.println("签名信息：" + signInfo);
		//生成随机密钥
        String aesKey = AesUtil.getAESSecureKey();
        System.out.println("密钥明文：" + aesKey);
		//公钥PUBLIC_KEY密钥RSA加密
        String encryptedAesKey = RsaUtil.encrypt(aesKey, Encrypt.PUBLIC_KEY);
        System.out.println("密钥密文：" + encryptedAesKey);
		//密钥对数据进行AES
        String encrypt = AesUtil.encrypt(busDataJson.toString(), aesKey);
        System.out.println("加密数据：" + encrypt);
        JsonObject root = new JsonObject();
        root.addProperty("encryptKey", encryptedAesKey);
        root.addProperty("busData", encrypt);
        System.out.println("======================以下是解密=======================");
        String encryptKey = root.get("encryptKey").getAsString();
        //私钥PRIVATE_KEY对加密后的密钥进行RSA解密
        String decryptKey = RsaUtil.decrypt(encryptKey, Encrypt.PRIVATE_KEY);
        System.out.println("解密后的密钥：" + decryptKey);
        String busData = root.get("busData").getAsString();
        //解密后的密钥DECRPTY_KEY对加密数据进行解密
        String decryptBusData = AesUtil.decrypt(busData, decryptKey);
        JsonObject respDataJsonObject = JsonParser.parseString(decryptBusData).getAsJsonObject();
        System.out.println("解密后的数据：" + respDataJsonObject.toString());

        String decryptPlaitText = respDataJsonObject.get("plainText").getAsString();
        String decryptSignInfo = respDataJsonObject.get("signInfo").getAsString();
        //验签
        boolean verifySign = SHA256withRSA.verifySign(Encrypt.PUBLIC_KEY, decryptPlaitText, decryptSignInfo);
        System.out.println("是否验签通过：" + verifySign);

    }
```
最终加解密结果如下，并没有太大的问题。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201020171435310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)

## Java后端
简单搭建个服务端，基于SpringBoot，非常简单，这里不展开。![在这里插入图片描述](https://img-blog.csdnimg.cn/20201020172759970.png#pic_center)
利用postman或者它老婆模拟客户端发数据，这里推荐一个[贼好用的Chrome插件](http://api.crap.cn/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201020173238409.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)
## Android端
Android上使用的加解密工具和Java后端是一致的，那就简单了，将之前后端的工具类和公私钥拷过来，按照加解密流程走一遍，so easy !

这里简单写个Android自身加解密、Android端加密，后端解密、后端加密，Android端解密的demo。
```java
    private void showDecrpytData(String response) throws EncryptException {
        JsonObject root = JsonParser.parseString(response).getAsJsonObject();

        String encryptKey = root.get("encryptKey").getAsString();
        String decryptKey = RsaUtil.decrypt(encryptKey, Encrypt.PRIVATE_KEY);
        System.out.println("解密后的密钥：" + decryptKey);

        String busData = root.get("busData").getAsString();
        String decryptBusData = AesUtil.decrypt(busData, decryptKey);

        JsonObject respDataJsonObject = JsonParser.parseString(decryptBusData).getAsJsonObject();
        System.out.println("解密后的数据：" + respDataJsonObject.toString());

        String decryptPlaitText = respDataJsonObject.get("plainText").getAsString();
        String decryptSignInfo = respDataJsonObject.get("signInfo").getAsString();
        etInput.setText(decryptPlaitText);
    }

    private String getEncrpytData() throws EncryptException {
        JsonObject busDataJson = new JsonObject();
        String plainText = etInput.getText().toString();
        String signInfo = SHA256withRSA.sign(Encrypt.PRIVATE_KEY, plainText);
        busDataJson.addProperty("plainText", plainText);
        busDataJson.addProperty("signInfo", signInfo);

        String aesKey = AesUtil.getAESSecureKey();
        String encryptedAesKey = RsaUtil.encrypt(aesKey, Encrypt.PUBLIC_KEY);
        String encrypt = AesUtil.encrypt(busDataJson.toString(), aesKey);

        JsonObject root = new JsonObject();
        root.addProperty("encryptKey", encryptedAesKey);
        root.addProperty("busData", encrypt);
        return root.toString();
    }
```
图片就不截了，有兴趣的可以到github上查看代码。

本文主要简单介绍了AES+RSA加解密流程和使用，具体算法并没有介绍，可参考[AES 加密算法的原理详解](https://blog.csdn.net/gulang03/article/details/81175854)；

后端demo：[请戳这里](https://github.com/ldqmaybe/hotch-server)
Android端demo：[请戳这里](https://github.com/ldqmaybe/aes_rsa)


**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)












