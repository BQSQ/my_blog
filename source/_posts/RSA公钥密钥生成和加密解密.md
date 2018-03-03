---
title: RSA公钥密钥生成和加密解密
date: 2018-01-31 09:04:43
tags: [加密解密,安全,RSA]
categories: [安全,加密解密]
keywords: [RSA,安全,加密解密,公钥，密钥]
---

*说明：非对称加密解密：需要用公钥进行加密，然后使用私钥进行解密。*

<!-- more -->

CreatePairRSAKeyUtil工具类
``` java
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.security.InvalidKeyException;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.SecureRandom;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import org.apache.commons.lang.ArrayUtils;
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

public class CreatePairRSAKeyUtil {
    private static BASE64Decoder decoder = new BASE64Decoder();

    private static BASE64Encoder encoder = new BASE64Encoder();

    /**
     * 创建密匙组，得到公匙、私匙
     * 
     * @param seed
     *            传入的生成密钥、公钥的字符串
     * @throws IOException
     */
    public static String[] createPairKey(String seed) {
        String[] strs = new String[2];
        try {
            // 根据特定的算法一个密钥对生成器
            KeyPairGenerator keygen = KeyPairGenerator.getInstance("RSA");
            // 加密随机数生成器 (RNG)
            SecureRandom random = new SecureRandom();
            // 重新设置此随机对象的种子
            random.setSeed(seed.getBytes());
            // 使用给定的随机源(和默认的参数集合)初始化确定密钥大小的密钥对生成器
            keygen.initialize(1024, random);
            // 生成密钥组
            KeyPair keys = keygen.genKeyPair();
            // 得到公匙
            PublicKey pubkey = keys.getPublic();
            String pubKeyValue = bytesToHexStr(pubkey.getEncoded());
            System.out.println("生成的公钥为:" + pubKeyValue);

            // 得到私匙
            PrivateKey prikey = keys.getPrivate();
            String prikeyValue = bytesToHexStr(prikey.getEncoded());
            System.out.println("生成的私钥为:" + prikeyValue);
            strs[0] = pubKeyValue;
            strs[1] = prikeyValue;
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return strs;
    }

    public static final String bytesToHexStr(byte[] bytes) {
        return encoder.encodeBuffer(bytes);
    }

    public static final byte[] hexStrToBytes(String str) throws IOException {
        return decoder.decodeBuffer(str);
    }

    /**
     * 加密
     * 
     * @param publicKey
     * @param srcBytes
     * @return
     * @throws NoSuchAlgorithmException
     * @throws NoSuchPaddingException
     * @throws InvalidKeyException
     * @throws IllegalBlockSizeException
     * @throws BadPaddingException
     * @throws IOException
     * @throws InvalidKeySpecException
     */
    public static String encrypt(String publicKeyValue, byte[] srcBytes)
            throws NoSuchAlgorithmException, NoSuchPaddingException,
            InvalidKeyException, IllegalBlockSizeException,
            BadPaddingException, IOException, InvalidKeySpecException {
        X509EncodedKeySpec bobPubKeySpec = new X509EncodedKeySpec(
                hexStrToBytes(publicKeyValue));
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        PublicKey publicKey = keyFactory.generatePublic(bobPubKeySpec);

        byte[] resultBytes = null;
        if (publicKey != null) {
            Cipher cipher = Cipher.getInstance("RSA");
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);
            // 加密时超过117字节就报错。为此采用分段加密的方法来加密
            StringBuilder subString = new StringBuilder();
            for (int i = 0; i < srcBytes.length; i += 117) {
                byte[] doFinal = cipher.doFinal(ArrayUtils.subarray(srcBytes,
                        i, i + 117));
                subString.append(new String(doFinal));
                resultBytes = ArrayUtils.addAll(resultBytes, doFinal);
            }
        }
        return bytesToHexStr(resultBytes);
    }

    /**
     * 解密
     * 
     * @param privateKey
     * @param srcBytes
     * @return
     * @throws NoSuchAlgorithmException
     * @throws NoSuchPaddingException
     * @throws InvalidKeyException
     * @throws IllegalBlockSizeException
     * @throws BadPaddingException
     * @throws IOException
     * @throws InvalidKeySpecException
     */
    public static String decrypt(String privateKeyValue, String srcBytes)
            throws Exception {
        PKCS8EncodedKeySpec priPKCS = new PKCS8EncodedKeySpec(
                hexStrToBytes(privateKeyValue));
        KeyFactory keyfact = KeyFactory.getInstance("RSA");
        PrivateKey privateKey = keyfact.generatePrivate(priPKCS);

        byte[] src;
        try {
            src = CreatePairRSAKeyUtil.hexStrToBytes(srcBytes);
            if (privateKey != null) {
                Cipher cipher = Cipher.getInstance("RSA");
                cipher.init(Cipher.DECRYPT_MODE, privateKey);
                // 解密时超过128字节就报错。为此采用分段解密的办法来解密
                StringBuffer strBuilder = new StringBuffer();
                for (int i = 0; i < src.length; i += 128) {
                    byte[] doFinal = cipher.doFinal(ArrayUtils.subarray(src, i,
                            i + 128));
                    strBuilder.append(new String(doFinal, "utf-8"));
                }
                srcBytes = strBuilder.toString();
            }
        } catch (Exception e) {
            e.printStackTrace();
            throw e;
        }
        return srcBytes;
    }

    public static void main(String[] args) throws Exception {
        String msg = "";
        File file = new File("D:/明文.txt");
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader(file));
            String tempString = null;
            while ((tempString = reader.readLine()) != null) {
                msg = msg + tempString;
            }
            reader.close();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reader.close();
        }
```

