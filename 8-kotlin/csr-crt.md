# 코틀린으로 CSR/CRT 만들어보자

양방향 암호화를 사용해서 public key, private key를 이용해보자!&#x20;

```
import sun.security.pkcs10.*
import sun.security.x509.*
import java.math.*
import java.security.*
import java.security.cert.*
import java.util.*


object SslCsrGenerator {

    fun generateKeyPair(): KeyPair {
        val kpg = KeyPairGenerator.getInstance("RSA")
        kpg.initialize(2048)
        return kpg.genKeyPair()
    }

    // transform to base64 with begin and end line
    fun publicKeyToPem(publicKey: PublicKey): String {
        val base64PubKey = Base64.getEncoder().encodeToString(publicKey.encoded)
        return "-----BEGIN RSA PUBLIC KEY-----\n" +
            base64PubKey.replace("(.{64})".toRegex(), "$1\n") +
            "\n-----END RSA PUBLIC KEY-----\n"
    }

    fun privateKeyToPem(privateKey: PrivateKey): String {
        val base64PubKey = Base64.getEncoder().encodeToString(privateKey.encoded)
        return "-----BEGIN RSA PRIVATE KEY-----\n" +
            base64PubKey.replace("(.{64})".toRegex(), "$1\n") +
            "\n-----END RSA PRIVATE KEY-----\n"
    }

    fun certificateToPem(certificateByte: ByteArray): String {
        val base64PubKey = Base64.getEncoder().encodeToString(certificateByte)
        return "-----BEGIN CERTIFICATE REQUEST-----\n" +
            base64PubKey.replace("(.{64})".toRegex(), "$1\n") +
            "\n-----END CERTIFICATE REQUEST-----\n"
    }

    fun createCertificate(keyPair: KeyPair, generatorAuthority: GeneratorAuthority, sslRequestAccount: SslRequestAccount): ByteArray {
        val commonName = generatorAuthority.commonName
        val organizationalUnit = generatorAuthority.organizationalUnitName
        val organization = generatorAuthority.organizationName
        val country = generatorAuthority.countryName
        val localityState = country

        // common, orgUnit, org, locality, state, country
        val distinguishedName = X500Name(commonName, organizationalUnit, organization, localityState, localityState, country )

        val sigAlgorithm = "MD5WithRSA"
        val pkcs10 = PKCS10(keyPair.public)
        val signature = Signature.getInstance(sigAlgorithm)
        signature.initSign(keyPair.private)
        pkcs10.encodeAndSign(distinguishedName, signature)
        return pkcs10.encoded
    }

    // CRT 발급 - 미사용
    fun createSelfSignedCertificate(keyPair: KeyPair, generatorAuthority: GeneratorAuthority): X509Certificate {
        // inspired by https://hecpv.wordpress.com/2017/03/18/how-to-generate-x-509-certificate-in-java-1-8/

        val commonName = generatorAuthority.commonName
        val organizationalUnit = generatorAuthority.organizationalUnitName
        val organization = generatorAuthority.organizationName
        val country = generatorAuthority.countryName

        val validDays = generatorAuthority.validDays

        val distinguishedName = X500Name(commonName, organizationalUnit, organization, country)

        val info = X509CertInfo()

        val since = Date() // Since Now
        val until = Date(since.time + validDays * 86400000L) // Until x days (86400000 milliseconds in one day)

        val sn = BigInteger(64, SecureRandom())

        info.set(X509CertInfo.VALIDITY, CertificateValidity(since, until))
        info.set(X509CertInfo.SERIAL_NUMBER, CertificateSerialNumber(sn))
        info.set(X509CertInfo.SUBJECT, distinguishedName)
        info.set(X509CertInfo.ISSUER, distinguishedName)
        info.set(X509CertInfo.KEY, CertificateX509Key(keyPair.public))
        info.set(X509CertInfo.VERSION, CertificateVersion(CertificateVersion.V3))

        var algo = AlgorithmId(AlgorithmId.md5WithRSAEncryption_oid)
        info.set(X509CertInfo.ALGORITHM_ID, CertificateAlgorithmId(algo))

        // Sign the cert to identify the algorithm that is used.
        var cert = X509CertImpl(info)
        cert.sign(keyPair.private, "SHA256withRSA")

        // Update the algorithm and sign again
        algo = cert.get(X509CertImpl.SIG_ALG) as AlgorithmId
        info.set(CertificateAlgorithmId.NAME + "." + CertificateAlgorithmId.ALGORITHM, algo)

        cert = X509CertImpl(info)
        cert.sign(keyPair.private, "SHA256withRSA")

        return cert
    }
}

```
