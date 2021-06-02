# NetPassport' Sign data with Java

The following is a code sample in JAVA for those who are using NetPassport
without JWT signature and want to sign the data theirself

## Prepare your data as a valid JSON

```java
class Main {
    public static void main(String[] args) throws Exception {

        String str = "{\"netPassportID\":\"662020766529949999\",\"pluginData\":[{\"phone\":\"10\",\"stockid\":\"1111\",\"date\":\"1/298\",\"kabSort\":\"11\",\"docNumber\":\"5\",\"accountKey\":444}]}";

        String hash = MD5.md5(str); // hash result
        String sig = SHA256RSA.sign(hash); // signature result
        System.out.println("Signature=\n" + sig);
    }
}
```

## Hash (MD5) the data

```java
import java.security.MessageDigest;

public class MD5 {
    public static String md5(String input) throws Exception {
        MessageDigest md = MessageDigest.getInstance("MD5");
        md.update(input.getBytes());

        byte byteData[] = md.digest();

        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < byteData.length; i++) {
            sb.append(Integer.toString((byteData[i] & 0xff) + 0x100, 16).substring(1));
        }

        return sb.toString();
    }
}
```

## Sign the data with your private key

```java
import java.security.KeyFactory;
import java.security.Signature;
import java.security.spec.PKCS8EncodedKeySpec;
import java.util.Base64;

public class SHA256RSA {

    public static String sign(String hash) throws Exception {
        // Not a real private key! Replace with your private key!
        String strPk = "-----BEGIN PRIVATE KEY-----\n"
            + "MIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQDoRxJ9XShHQWY8\n"
            + "uoZGjeYEB9NdEW99vh1WUE9+4UHFMTgJ5y9iEx+Qhzm5Czltj8W5VCgQ7kAICv+I\n"
            + "-----END PRIVATE KEY-----";

        String base64Signature = signSHA256RSA(hash, strPk);
        return base64Signature;
    }

    // Create base64 encoded signature using RSA-SHA256.
    private static String signSHA256RSA(String input, String strPk) throws Exception {
        // Remove markers and new line characters in private key
        String realPK = strPk.replaceAll("-----END PRIVATE KEY-----", "")
            .replaceAll("-----BEGIN PRIVATE KEY-----", "")
            .replaceAll("\n", "");

        byte[] b1 = Base64.getDecoder().decode(realPK);
        PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(b1);
        KeyFactory kf = KeyFactory.getInstance("RSA");

        Signature privateSignature = Signature.getInstance("SHA256withRSA");
        privateSignature.initSign(kf.generatePrivate(spec));
        privateSignature.update(input.getBytes("UTF-8"));
        byte[] s = privateSignature.sign();

        return Base64.getEncoder().encodeToString(s);
    }
}

```
