常见的安全算法，包括MD5、SHA、DES、AES、RSA等。



#### 1. 数字摘要算法

---

>数字摘要也称为消息摘要，它是一个唯一对应一个消息或文本的固定长度的值，它由一个单向Hash函数对消息进行计算而产生。如果消息在传递的途中改变了，接收者通过对收到消息采用相同的Hash重新计算，新产生的摘要与原摘要进行比较，就可知道消息是否被篡改了，因此消息摘要能够验证消息的完整性。消息摘要==采用单向Hash函数将需要计算的内容”摘要”成固定长度的串==，这个串亦称为数字指纹。这个串有固定的长度，且不同的明文摘要成密文，其结果总是不同的(相对的)，而同样的明文其摘要必定一致。这样这串摘要便可成为验证明文是否是”真身”的”指纹”了。

##### 1.1 MD5

`Message Digest Algorithm 5(信息摘要算法5)`，是数字摘要算法一种实现，用于**确保信息传输完整性和一致性**，摘要长度为128位。 MD5由MD4、 MD3、 MD2改进而来，主要增强算法复杂度和不可逆性，该算法因其普遍、稳定、快速的特点，在产业界得到了极为广泛的使用，目前主流的编程语言普遍都已有MD5算法实现。

>可以使用spring框架中的DigestUtils工具类进行MD5加密。
>
>也可以自定义实现工具类，代码如下：

```java
public class MD5Util {

  private static final Logger LOGGER = LoggerFactory.getLogger(MD5Util.class);

  private static final char[] HEX_DIGITS = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};

  private MD5Util() {
  }

  public static String toHexString(byte[] b) { // String to byte
    StringBuilder sb = new StringBuilder(b.length * 2);
    for (int i = 0; i < b.length; i++) {
      sb.append(HEX_DIGITS[(b[i] & 0xf0) >>> 4]);
      sb.append(HEX_DIGITS[b[i] & 0x0f]);
    }
    return sb.toString();
  }

  public static String getMd5(String s) {
    try {
      // Create MD5Util Hash
      MessageDigest digest = MessageDigest
        .getInstance("MD5");
      digest.update(s.getBytes());
      byte[] messageDigest = digest.digest();

      return toHexString(messageDigest);
    } catch (NoSuchAlgorithmException e) {
      LOGGER.error(">>>>>>>>>>MD5加密异常", e);
    }
    return "";
  }
}
```

##### 1.2 SHA

`Secure Hash Algorithm(安全散列算法`)。 1993年，安全散列算法(SHA)由美国国家标准和技术协会（NIST)提出，并作为联邦信息处理标准(FIPS PUB 180)公布， 1995年又发布了一个修订版FIPS PUB 180-1，通常称之为SHA-1。 SHA-1是基于MD4算法的，现在已成为公认的最安全的散列算法之一，并被广泛使用。SHA-1算法生成的摘要信息的长度为160位，由于生成的摘要信息更长，运算的过程更加复杂，在相同的硬件上， SHA-1的运行速度比MD5更慢，但是也更为安全。

```java
public class SHAUtil {

  /**
     * 定义加密方式
     */
  private static final String KEY_SHA = "SHA";
  private static final String KEY_SHA1 = "SHA-1";
  /**
     * 全局数组
     */
  private static final String[] hexDigits = {"0", "1", "2", "3", "4", "5",
                                             "6", "7", "8", "9", "a", "b", "c", "d", "e", "f"};

  /**
     * 构造函数
     */
  private SHAUtil() {

  }

  /**
     * SHA 加密
     *
     * @param data 需要加密的字节数组
     * @return 加密之后的字节数组
     * @throws Exception
     */
  public static byte[] encryptSHA(byte[] data) throws NoSuchAlgorithmException {
    // 创建具有指定算法名称的信息摘要
    MessageDigest sha = MessageDigest.getInstance(KEY_SHA1);
    // 使用指定的字节数组对摘要进行最后更新
    sha.update(data);
    // 完成摘要计算并返回
    return sha.digest();
  }

  /**
     * SHA 加密
     *
     * @param data 需要加密的字符串
     * @return 加密之后的字符串
     * @throws Exception
     */
  public static String encryptSHA(String data) throws NoSuchAlgorithmException {
    // 验证传入的字符串
    if (Strings.isNullOrEmpty(data)) {
      return "";
    }
    // 创建具有指定算法名称的信息摘要
    MessageDigest sha = MessageDigest.getInstance(KEY_SHA1);
    // 使用指定的字节数组对摘要进行最后更新
    sha.update(data.getBytes());
    // 完成摘要计算
    byte[] bytes = sha.digest();
    // 将得到的字节数组变成字符串返回
    return byteArrayToHexString(bytes);
  }

  /**
     * 将一个字节转化成十六进制形式的字符串
     *
     * @param b 字节数组
     * @return 字符串
     */
  private static String byteToHexString(byte b) {
    int ret = b;
    if (ret < 0) {
      ret += 256;
    }
    int m = ret / 16;
    int n = ret % 16;
    return hexDigits[m] + hexDigits[n];
  }

  /**
     * 转换字节数组为十六进制字符串
     *
     * @param bytes 字节数组
     * @return 十六进制字符串
     */
  private static String byteArrayToHexString(byte[] bytes) {
    StringBuffer sb = new StringBuffer();
    for (int i = 0; i < bytes.length; i++) {
      sb.append(byteToHexString(bytes[i]));
    }
    return sb.toString();
  }
}
```



#### 2. 对称加密

---

在对称加密算法中，数据发送方将明文(原始数据)和加密密钥一起经过特殊加密算法处理后，生成复杂的加密密文进行发送，数据接收方收到密文后，若想读取原文，则需要使用加密使用的密钥及相同算法的逆算法对加密的密文进行解密，才能使其恢复成可读明文。在对称加密算法中，使用的密钥只有一个，发送和接收双方都使用这个密钥对数据进行加密和解密，这就要求加密和解密方事先都必须知道加密的密钥。

##### 2.1 DES 算法

DES算法属于对称加密算法，明文按64位进行分组，密钥长64位，但事实上只有56位参与DES 运算(第8、 16、 24、 32、 40、 48、 56、 64位是校验位，使得每个密钥都有奇数个1),分组后的明文和56位的密钥按位替代或交换的方法形成密文。由于计算机运算能力的增强，原版DES密码的密钥长度变得容易被暴力破解，因此演变出了3DES算法。 3DES是DES向AES过渡的加密算法，它使用3条56位的密钥对数据进行三次加密，是DES的一个更安全的变形。

##### 2.2 AES

AES的全称是Advanced Encryption Standard，即高级加密标准。AES算法作为新一代的数据加密标准汇聚了强安全性、高性能、高效率、易用和灵活等优 点，设计有三个密钥长度:128，192，256位，比DES算法的加密强度更高，更为安全。

```java
public class AESUtil {

  static byte[] key = "w@#$4@#$s^&3*&^4".getBytes();
  final static String algorithm = "AES";

  public static String encrypt(String data) {

    byte[] dataToSend = data.getBytes();
    Cipher c = null;
    try {
      c = Cipher.getInstance(algorithm);
    } catch (NoSuchAlgorithmException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (NoSuchPaddingException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    SecretKeySpec k = new SecretKeySpec(key, algorithm);
    try {
      c.init(Cipher.ENCRYPT_MODE, k);
    } catch (InvalidKeyException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    byte[] encryptedData = "".getBytes();
    try {
      encryptedData = c.doFinal(dataToSend);
    } catch (IllegalBlockSizeException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (BadPaddingException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    byte[] encryptedByteValue = Base64.getEncoder().encode(encryptedData);
    return new String(encryptedByteValue);
  }

  public static String decrypt(String data) {

    byte[] encryptedData = Base64.getDecoder().decode(data);
    Cipher c = null;
    try {
      c = Cipher.getInstance(algorithm);
    } catch (NoSuchAlgorithmException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (NoSuchPaddingException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    SecretKeySpec k =
      new SecretKeySpec(key, algorithm);
    try {
      c.init(Cipher.DECRYPT_MODE, k);
    } catch (InvalidKeyException e1) {
      // TODO Auto-generated catch block
      e1.printStackTrace();
    }
    byte[] decrypted = null;
    try {
      decrypted = c.doFinal(encryptedData);
    } catch (IllegalBlockSizeException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (BadPaddingException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    return new String(decrypted);
  }

  public static void main(String[] args) {
    String password = encrypt("12233440988:1239874389888:dd333");
    System.out.println(password);
    System.out.println(decrypt(password));
  }
}
```



#### 3. 非对称加密

---

非对称加密算法又称为公开密钥加密算法，它需要两个密钥，一个称为==公开密钥(public key)==，即公钥，==另一个称为私有密钥(private key)==，即私钥。公钥与私钥需要配对使用，如果用公钥对数据进行加密，只有用对应的私钥才能进行解密，而如果使用私钥对数据进行加密，那么只有用对应的公钥才能进行解密。因为加密和解密使用的是两个不同的密钥，所以这种算法称为非对称加密算法。

非对称加密算法实现机密信息交换的基本过程是：甲方生成一对密钥并将其中的一把作为公钥向其它人公开，得到该公钥的乙方使用该密钥对机密信息进行加密后再发送给甲方，甲方再使用自己保存的另一把专用密钥，即私钥，对加密后的信息进行解密。

##### 3.1 RAS

RSA是目前最有影响力的非对称加密算法，它能够抵抗到目前为止已知的所有密码攻击，已被ISO推荐为公钥数据加密标准。 RSA算法基于一个十分简单的数论事实：将两个大素数相乘十分容易，但反过来想要对其乘积进行因式分解却极其困难，因此可以将乘积公开作为加密密钥。

```java
public class RSAUtil {

  /**
   * 字节数据转字符串专用集合
   */
  private static final char[] HEX_CHAR = {'0', '1', '2', '3', '4', '5', '6',
                                          '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};

  /**
   * 随机生成密钥对
   */
  public static void genKeyPair(String filePath) {
    // KeyPairGenerator类用于生成公钥和私钥对，基于RSA算法生成对象
    KeyPairGenerator keyPairGen = null;
    try {
      keyPairGen = KeyPairGenerator.getInstance("RSA");
    } catch (NoSuchAlgorithmException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    // 初始化密钥对生成器，密钥大小为96-1024位
    keyPairGen.initialize(1024, new SecureRandom());
    // 生成一个密钥对，保存在keyPair中
    KeyPair keyPair = keyPairGen.generateKeyPair();
    // 得到私钥
    RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
    // 得到公钥
    RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
    try {
      // 得到公钥字符串
      // 得到私钥字符串
      String privateKeyString = new String(Base64.getEncoder().encode(privateKey.getEncoded()));
      String publicKeyString = new String(Base64.getEncoder().encode(publicKey.getEncoded()));
      // 将密钥对写入到文件

      File file1 = new File(filePath + "publicKey.keystore");
      File file2 = new File(filePath + "privateKey.keystore");
      if (!file1.exists()) {
        file1.createNewFile();
      }
      if (!file2.exists()) {
        file2.createNewFile();
      }
      FileWriter pubfw = new FileWriter(filePath + "/publicKey.keystore");
      FileWriter prifw = new FileWriter(filePath + "/privateKey.keystore");
      BufferedWriter pubbw = new BufferedWriter(pubfw);
      BufferedWriter pribw = new BufferedWriter(prifw);
      pubbw.write(publicKeyString);
      pribw.write(privateKeyString);
      pubbw.flush();
      pubbw.close();
      pubfw.close();
      pribw.flush();
      pribw.close();
      prifw.close();
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  /**
   * 从文件中输入流中加载公钥
   *
   * @param
   * @throws Exception 加载公钥时产生的异常
   */
  public static String loadPublicKeyByFile(String path) throws Exception {
    try {
      BufferedReader br = new BufferedReader(new FileReader(path
                                                            + "/publicKey.keystore"));
      String readLine = null;
      StringBuilder sb = new StringBuilder();
      while ((readLine = br.readLine()) != null) {
        sb.append(readLine);
      }
      br.close();
      return sb.toString();
    } catch (IOException e) {
      throw new Exception("公钥数据流读取错误");
    } catch (NullPointerException e) {
      throw new Exception("公钥输入流为空");
    }
  }

  /**
   * 从字符串中加载公钥
   *
   * @param publicKeyStr 公钥数据字符串
   * @throws Exception 加载公钥时产生的异常
   */
  public static RSAPublicKey loadPublicKeyByStr(String publicKeyStr)
    throws Exception {
    try {
      byte[] buffer = Base64.getDecoder().decode(publicKeyStr);
      KeyFactory keyFactory = KeyFactory.getInstance("RSA");
      X509EncodedKeySpec keySpec = new X509EncodedKeySpec(buffer);
      return (RSAPublicKey) keyFactory.generatePublic(keySpec);
    } catch (NoSuchAlgorithmException e) {
      throw new Exception("无此算法");
    } catch (InvalidKeySpecException e) {
      throw new Exception("公钥非法");
    } catch (NullPointerException e) {
      throw new Exception("公钥数据为空");
    }
  }

  /**
   * 从文件中加载私钥
   *
   * @param
   * @return 是否成功
   * @throws Exception
   */
  public static String loadPrivateKeyByFile(String path) throws Exception {
    try {
      BufferedReader br = new BufferedReader(new FileReader(path
                                                            + "/privateKey.keystore"));
      String readLine = null;
      StringBuilder sb = new StringBuilder();
      while ((readLine = br.readLine()) != null) {
        sb.append(readLine);
      }
      br.close();
      return sb.toString();
    } catch (IOException e) {
      throw new Exception("私钥数据读取错误");
    } catch (NullPointerException e) {
      throw new Exception("私钥输入流为空");
    }
  }

  public static RSAPrivateKey loadPrivateKeyByStr(String privateKeyStr)
    throws Exception {
    try {
      byte[] buffer = Base64.getDecoder().decode(privateKeyStr);
      PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(buffer);
      KeyFactory keyFactory = KeyFactory.getInstance("RSA");
      return (RSAPrivateKey) keyFactory.generatePrivate(keySpec);
    } catch (NoSuchAlgorithmException e) {
      throw new Exception("无此算法");
    } catch (InvalidKeySpecException e) {
      throw new Exception("私钥非法");
    } catch (NullPointerException e) {
      throw new Exception("私钥数据为空");
    }
  }

  /**
   * 公钥加密过程
   *
   * @param publicKey     公钥
   * @param plainTextData 明文数据
   * @return
   * @throws Exception 加密过程中的异常信息
   */
  public static byte[] encrypt(RSAPublicKey publicKey, byte[] plainTextData)
    throws Exception {
    if (publicKey == null) {
      throw new Exception("加密公钥为空, 请设置");
    }
    Cipher cipher = null;
    try {
      // 使用默认RSA
      cipher = Cipher.getInstance("RSA");
      // cipher= Cipher.getInstance("RSA", new BouncyCastleProvider());
      cipher.init(Cipher.ENCRYPT_MODE, publicKey);
      byte[] output = cipher.doFinal(plainTextData);
      return output;
    } catch (NoSuchAlgorithmException e) {
      throw new Exception("无此加密算法");
    } catch (NoSuchPaddingException e) {
      e.printStackTrace();
      return null;
    } catch (InvalidKeyException e) {
      throw new Exception("加密公钥非法,请检查");
    } catch (IllegalBlockSizeException e) {
      throw new Exception("明文长度非法");
    } catch (BadPaddingException e) {
      throw new Exception("明文数据已损坏");
    }
  }

  /**
   * 私钥加密过程
   *
   * @param privateKey    私钥
   * @param plainTextData 明文数据
   * @return
   * @throws Exception 加密过程中的异常信息
   */
  public static byte[] encrypt(RSAPrivateKey privateKey, byte[] plainTextData)
    throws Exception {
    if (privateKey == null) {
      throw new Exception("加密私钥为空, 请设置");
    }
    Cipher cipher = null;
    try {
      // 使用默认RSA
      cipher = Cipher.getInstance("RSA");
      cipher.init(Cipher.ENCRYPT_MODE, privateKey);
      byte[] output = cipher.doFinal(plainTextData);
      return output;
    } catch (NoSuchAlgorithmException e) {
      throw new Exception("无此加密算法");
    } catch (NoSuchPaddingException e) {
      e.printStackTrace();
      return null;
    } catch (InvalidKeyException e) {
      throw new Exception("加密私钥非法,请检查");
    } catch (IllegalBlockSizeException e) {
      throw new Exception("明文长度非法");
    } catch (BadPaddingException e) {
      throw new Exception("明文数据已损坏");
    }
  }

  /**
   * 私钥解密过程
   *
   * @param privateKey 私钥
   * @param cipherData 密文数据
   * @return 明文
   * @throws Exception 解密过程中的异常信息
   */
  public static byte[] decrypt(RSAPrivateKey privateKey, byte[] cipherData)
    throws Exception {
    if (privateKey == null) {
      throw new Exception("解密私钥为空, 请设置");
    }
    Cipher cipher = null;
    try {
      // 使用默认RSA
      cipher = Cipher.getInstance("RSA");
      // cipher= Cipher.getInstance("RSA", new BouncyCastleProvider());
      cipher.init(Cipher.DECRYPT_MODE, privateKey);
      byte[] output = cipher.doFinal(cipherData);
      return output;
    } catch (NoSuchAlgorithmException e) {
      throw new Exception("无此解密算法");
    } catch (NoSuchPaddingException e) {
      e.printStackTrace();
      return null;
    } catch (InvalidKeyException e) {
      throw new Exception("解密私钥非法,请检查");
    } catch (IllegalBlockSizeException e) {
      throw new Exception("密文长度非法");
    } catch (BadPaddingException e) {
      throw new Exception("密文数据已损坏");
    }
  }

  /**
   * 公钥解密过程
   *
   * @param publicKey  公钥
   * @param cipherData 密文数据
   * @return 明文
   * @throws Exception 解密过程中的异常信息
   */
  public static byte[] decrypt(RSAPublicKey publicKey, byte[] cipherData)
    throws Exception {
    if (publicKey == null) {
      throw new Exception("解密公钥为空, 请设置");
    }
    Cipher cipher = null;
    try {
      // 使用默认RSA
      cipher = Cipher.getInstance("RSA");
      // cipher= Cipher.getInstance("RSA", new BouncyCastleProvider());
      cipher.init(Cipher.DECRYPT_MODE, publicKey);
      byte[] output = cipher.doFinal(cipherData);
      return output;
    } catch (NoSuchAlgorithmException e) {
      throw new Exception("无此解密算法");
    } catch (NoSuchPaddingException e) {
      e.printStackTrace();
      return null;
    } catch (InvalidKeyException e) {
      throw new Exception("解密公钥非法,请检查");
    } catch (IllegalBlockSizeException e) {
      throw new Exception("密文长度非法");
    } catch (BadPaddingException e) {
      throw new Exception("密文数据已损坏");
    }
  }

  /**
   * 字节数据转十六进制字符串
   *
   * @param data 输入数据
   * @return 十六进制内容
   */
  public static String byteArrayToString(byte[] data) {
    StringBuilder stringBuilder = new StringBuilder();
    for (int i = 0; i < data.length; i++) {
      // 取出字节的高四位 作为索引得到相应的十六进制标识符 注意无符号右移
      stringBuilder.append(HEX_CHAR[(data[i] & 0xf0) >>> 4]);
      // 取出字节的低四位 作为索引得到相应的十六进制标识符
      stringBuilder.append(HEX_CHAR[(data[i] & 0x0f)]);
      if (i < data.length - 1) {
        stringBuilder.append(' ');
      }
    }
    return stringBuilder.toString();
  }


  public static void main(String[] args) throws Exception {
    String filepath = "F:/temp/";
    File file = new File(filepath);
    if (!file.exists()) {
      file.mkdir();
    }
    genKeyPair(filepath);
    System.out.println("--------------公钥加密私钥解密过程-------------------");
    String plainText = "1223333323:8783737321232:dewejj28i33e92hhsxxxx";
    //公钥加密过程
    byte[] cipherData = encrypt(loadPublicKeyByStr(loadPublicKeyByFile(filepath)), plainText.getBytes());
    String cipher = new String(Base64.getEncoder().encode(cipherData));
    //私钥解密过程
    byte[] res = decrypt(loadPrivateKeyByStr(loadPrivateKeyByFile(filepath)), Base64.getDecoder().decode(cipher));
    String restr = new String(res);
    System.out.println("原文：" + plainText);
    System.out.println("加密密文：" + cipher);
    System.out.println("解密：" + restr);
    System.out.println();
  }
}
```