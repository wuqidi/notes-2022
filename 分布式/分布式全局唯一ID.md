#### UUID

Version1：根据时间+MAC地址，每秒可产生1630亿个。场景：反向解析主机MAC。MySql提供的UUID函数是V1。

Version2：根据用户组ID+时间+MAC地址。一般用不到。

Version3/5：根据散列名字空间标识符和名称。V3使用MD5；V5使用SHA1。
						主要适用于给定字符串生成相同UUID。V3不推荐，V5可用于密码。

Version4：伪随机生成。每秒10亿。首选

#### NanoID

NanoID与UUID-V4相似。

JS：不安全的 `Math.random()`NanoID 使用 `crypto module` 和 `Web Crypto API`

与 UUID 字母表中的 36 个字符不同，NanoID 只有 21 个字符。(这算个屁优势，默认21个)

NanoID **允许开发人员使用自定义字母表**（这是真有点用处的）

<!-- https://mvnrepository.com/artifact/com.aventrix.jnanoid/jnanoid -->
<dependency>
    <groupId>com.aventrix.jnanoid</groupId>
    <artifactId>jnanoid</artifactId>
    <version>2.0.0</version>
    <scope>runtime</scope>
</dependency>

```java
import java.security.SecureRandom;
import java.util.Random;
 
public final class NanoIdUtils {
    public static final SecureRandom DEFAULT_NUMBER_GENERATOR = new SecureRandom();
    //随机算法是伪随机，随机算法的起源数叫做种子数。相同种子数的随机对象，相同生成次数，生成的随机数则相同。
    //SecureRandom:以鼠标、键盘点击和CPU时钟等随机事件作为种子，种子不具备预测性。
    //Random：以系统当前时间的毫秒数作为种子，有规律。
    public static final char[] DEFAULT_ALPHABET = "_-0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();//64
    public static final int DEFAULT_SIZE = 21;//默认21个字符
 
    private NanoIdUtils() {
    }
 
    public static String randomNanoId() {
        return randomNanoId(DEFAULT_NUMBER_GENERATOR, DEFAULT_ALPHABET, 21);
    }
 
    public static String randomNanoId(Random random, char[] alphabet, int size) {
        if (random == null) {
            throw new IllegalArgumentException("random cannot be null.");
        } else if (alphabet == null) {
            throw new IllegalArgumentException("alphabet cannot be null.");
        } else if (alphabet.length != 0 && alphabet.length < 256) {
            if (size <= 0) {
                throw new IllegalArgumentException("size must be greater than zero.");
            } else {
                int mask = (2 << (int)Math.floor(Math.log((double)(alphabet.length - 1)) / Math.log(2.0D))) - 1;
                //Math.floor:向下取整，返回小于输入数值最接近的整数
                //Math.log:对数，以e为底。
                int step = (int)Math.ceil(1.6D * (double)mask * (double)size / (double)alphabet.length);
                //Math.ceil：对浮点数向上取整
                StringBuilder idBuilder = new StringBuilder();
                while(true) {
                    byte[] bytes = new byte[step];
                    random.nextBytes(bytes); 
                    for(int i = 0; i < step; ++i) {
                        int alphabetIndex = bytes[i] & mask;
                        if (alphabetIndex < alphabet.length) {
                            idBuilder.append(alphabet[alphabetIndex]);
                            if (idBuilder.length() == size) {
                                return idBuilder.toString();
                            }
                        }
                    }
                }
            }
        } else {
            throw new IllegalArgumentException("alphabet must contain between 1 and 255 symbols.");
        }
    }
}
```

