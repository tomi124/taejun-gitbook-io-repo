# 파일 압축 해보자

파일 압축의 경우 Blocking 로직이 수반되기 때문에, 조심하자&#x20;

```
import java.io.*
import java.util.zip.*

/**
 * @author Taejun Park
 */
object ZipMakerUtils {
    private fun File.bufferedOutputStream(size: Int = 8192) = BufferedOutputStream(this.outputStream(), size)
    private fun File.zipOutputStream(size: Int = 8192) = ZipOutputStream(this.bufferedOutputStream(size))
    private fun File.bufferedInputStream(size: Int = 8192) = BufferedInputStream(this.inputStream(), size)
    private fun File.asZipEntry() = ZipEntry(this.name)

    // 지정된 파일 압축
    fun archiveZip(files: List<File>, destination: File) =
        destination.zipOutputStream().use {
            files.forEach { file ->
                it.putNextEntry(file.asZipEntry())
                file.bufferedInputStream().use { bis -> bis.copyTo(it) }
            }
        }
}

```
