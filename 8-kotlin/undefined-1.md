# 양방향 암호화를 이용해보자

```
import org.jasypt.contrib.org.apache.commons.codec_1_3.binary.*
import org.slf4j.*
import javax.crypto.*
import javax.crypto.spec.*

/**
 *  - aes128/cbc/PKCS5Padding
 *
 */
object DoEncrypt {

    private const val TRANSFORMATION = "AES/CBC/PKCS5Padding"
    private const val ALGORITHM = "AES"
    private const val MAX_LENGTH = 16

    private val secureKey: SecretKey
    private val ivSpec: IvParameterSpec

    private val log = LoggerFactory.getLogger(javaClass)

    init {
        val password = "password"
        val ivPassword = "ivPassword"

        val secretKey = if (password.length > MAX_LENGTH) password.substring(
            0,
            MAX_LENGTH
        ) else password
        val ivKey = if (ivPassword.length > MAX_LENGTH) ivPassword.substring(
            0,
            MAX_LENGTH
        ) else ivPassword

        val keyData = String.format("%${MAX_LENGTH}s", secretKey).replace(' ', '0').toByteArray()
        val iv = String.format("%${MAX_LENGTH}s", ivKey).replace(' ', '0').toByteArray()

        secureKey = SecretKeySpec(
            keyData,
            ALGORITHM
        )
        ivSpec = IvParameterSpec(iv)
    }

    private fun encryptAes(str: String): String {
        val c = Cipher.getInstance(TRANSFORMATION)
            .apply {
                init(
                    Cipher.ENCRYPT_MODE,
                    secureKey,
                    ivSpec
                )
            }
        val encrypted = c.doFinal(str.toByteArray(Charsets.UTF_8))

        return String(Base64.encodeBase64(encrypted))
    }

    private fun decryptAes(str: String): String {
        return try {
            val c = Cipher.getInstance(TRANSFORMATION)
                .apply {
                    init(
                        Cipher.DECRYPT_MODE,
                        secureKey,
                        ivSpec
                    )
                }
            val byteStr = Base64.decodeBase64(str.toByteArray())

            String(c.doFinal(byteStr), Charsets.UTF_8)
        } catch (e: Exception) {
            log.warn("decrypt failed for input $str", e)
            return str
        }
    }

    /**
     * 복호화
     */
    fun decrypt(encryptedMessage: String): String {
        return decryptAes(encryptedMessage)
    }

    /**
     * 암호화
     */
    fun encrypt(message: String): String {
        return encryptAes(message)
    }
}
```
