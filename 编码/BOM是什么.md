## BOM是什么
> BOM: Byte Order Mark,即字节序标志

在UCS 编码中有一个叫做"ZERO WIDTH NO-BREAK SPACE"的字符，它的编码是FEFF。而FFFE在UCS中是不存在的字符，所以不应该出现在实际传输中。UCS规范建议我们在传输字节流前，先传输字符"ZERO WIDTH NO-BREAK SPACE"。这样如果接收者收到FEFF，就表明这个字节流是Big-Endian的；如果收到FFFE，就表明这个字节流是Little- Endian的。因此字符"ZERO WIDTH NO-BREAK SPACE"又被称作BOM。

BOM在Unicode编码中的应用：
* UTF-16用BOM标识字节序，通常Unicode文件的开通有0xFFFE表示小端字节序，0xFEFF表示大端字节序。

* UTF-8由于编码特性，通常只占有一个字节，**不需要BOM来表明字节顺序，但可以用BOM来表明编码方式。**字符"ZERO WIDTH NO-BREAK SPACE"的UTF-8编码是EF BB BF。所以如果接收者收到以EF BB BF开头的字节流，就知道这是UTF-8编码了。

记事本用的是ANSI编码，就是这种系统默认编码，所以，一个中文文本，用ANSI编码保存，在中文版里编码是GBK模式保存的时候，到繁体中文版里，用BIG5读取，就全乱套了。 

记事本也不甘心这样，所以它要支持Unicode，但是有一个问题，一段二进制编码，如何确定它是GBK还是BIG5还是UTF-16/UTF-8？

记事本的做法是在TXT文件的最前面保存一个标签，如果记事本打开一个TXT，发现这个标签，就说明是unicode。标签叫BOM。
* 如果是0xFF 0xFE，是UTF16LE。
* 如果是0xFE 0xFF则UTF16BE。
* 如果是0xEF 0xBB 0xBF，则是UTF-8。
* 如果没有这三个东西，那么就是ANSI，使用操作系统的默认语言编码来解释。

Windows相对对BOM处理比较好，是因为Windows把Unicode识别代码集成进了API里，主要是CreateFile()。打开文本文件时它会自动识别并剔除BOM。BOM不受欢迎主要是在UNIX环境下，因为很多UNIX程序不鸟BOM。主要问题出在UNIX那个所有脚本语言通行的首行#!标示，这东西依赖于shell解析，而很多shell出于兼容的考虑不检测BOM，所以加进BOM时shell会把它解释为某个普通字符输入导致破坏#!标示，这就麻烦了。其实很多现代脚本语言，比如Python，其解释器本身都是能处理BOM的，但是shell卡在这里，没办法，只能躺着也中枪。