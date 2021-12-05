　　myeclipse 工程用 idea 导入的时候，会报以下错误。

```
java: illegal character: \65279
```

　　大概原因是因为在 window下面保存 `BOM` 格式了，用户16进制来看的话，前面有这个字节

```
-17 -69 -65
```

　　需要要去掉这几个字段，idea 编译就会正常了。给出实现代码

　　用到了 `commons-io`

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

```java
import org.apache.commons.io.FileUtils;

import java.io.File;
import java.util.Arrays;
import java.util.Collection;
import java.util.Iterator;


public class Test {

      public static void main(String[] args) throws Exception {
        Collection<File> files = FileUtils.listFiles(new File("you source path"), null, true);

        Iterator<File> iterator = files.iterator();

        for( ;iterator.hasNext(); ){
            File next = iterator.next();
            if( next.getName().indexOf(".java")== -1 ){
                continue;
            }

            byte[] bytes1 = FileUtils.readFileToByteArray(next);
            if( bytes1.length == 0 ){
                continue;
            }

            if( bytes1[0] == -17 && bytes1[1] == -69 && bytes1[2] == -65 ){

                byte[] bytes2 = Arrays.copyOfRange(bytes1, 3, bytes1.length);
                FileUtils.writeByteArrayToFile(next, bytes2);
            }
        }

    }
}
```
