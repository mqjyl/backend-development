# 编码和字符集

## :pen\_fountain: 字节

&#x20;**字节**是计算存储容量的一种计量单位。我们知道计算机只能识别1和0组成的二进制位。一个数就是1位（bit），为了方便计算，我们规定8位就是一个**字节**。

## :pen\_fountain: 字符

**字符**和字节不太一样，任何一个文字或符号都是一个字符，但所占字节不一定，不同的编码导致一个**字符**所占的内存不同。

例如：标点符号`+`是一个**字符**，汉字`我们`是两个**字符**，在`GBK`编码中一个汉字占2个字节，在`UTF-8`编码中一个汉字占3个字节。

## :pen\_fountain: 编码规范

计算机只能识别0和1的二进制数，为了显示字符，国际组织就制定了编码规范，希望使用不同的二进制数来表示代表不同的字符，这样电脑就可以根据二进制数来显示其对应的字符。

### :paintbrush: 字库表

一套编码规范不一定包含世界上所有的字符，每套编码规范都有自己的使用场景。而**字库表**就存储了编码规范中能显示的所有字符，计算机就是根据二进制数从**字库表**中找到字符然后显示给用户滴，相当于一个存储字符的数据库。

例如：几乎所有汉字都保存在`GBK`编码规范的**字库表**中。所以可以显示汉字，但法语，俄语并不在其**字库表**中，所以`GBK`不能显示法语，俄语等不包含在其中的字符。

### :paintbrush: 编码字符集（字符集）

&#x20;在一个字库表中，每一个字符都有一个对应的二进制地址，而**编码字符集**就是这些地址的集合。

例如：在ASCII码的**编码字符集**中，字母`A`的序号（地址）是65，65的二进制就是`01000001`。我们可以说**编码字符集**就是用来存储这些二进制数的。而这个二进制数就是**编码字符集**中的一个元素，同时它也是字库表中字母`A`的地址。我们根据这个地址就可以显示出字母`A`。

**结论：字符集和字库表一一对应，相互转换，这是电脑识别字符的关键。**

### :paintbrush: **字符编码（编码方式）**

知道字库表和编码字符集后，我们就可以直接使用二进制地址来得到字符了。

但直接使用字符对应的二进制地址来显示文字是十分浪费的，Unicode 编码规范中包括了几百万个字符，想要包括几百万个不同的字符，起码需要3个字节的容量，为了方便将来扩展，Unicode还保留了更多未使用的空间，最多可以存储4个字节的容量。

因此为了区分每个字符，哪怕是`00000000 00000000 00000000 00001111`这种其实只占了1个字节的字符，我们也要为他分配4个字节的空间，这就导致一个可以用1G保存的文件，现在需要4G才能保存，这是极其浪费的做法。

于是程序员制定了一套算法来节省空间，而每种不同的算法都被称作一种**编码方式**（下文中为了便于理解都将使用**编码方式**来称呼**字符编码**）。一套编码规范可以有多种不同的**编码方式**，不同的编码方式有不同的适应场景。

例如：`UTF-8`就是一种**编码方式**，Unicode是一种编码规范。此外，Unicode还有UTF-16，UTF-32这两种**编码方式**。不同的**编码方式**节约的空间不同。

**总结：一个较短的二进制数，通过一种编码方式，转换成编码字符集中正常的地址，然后在字库表中找到一个对应的字符，最终显示给用户。**

## :pen\_fountain: **常见的编码规范**

### :paintbrush: **** ASCII码

ASCII码，是最早产生的编码规范，一共包含`00000000`\~`01111111`共128个字符，可以表示阿拉伯数字和大小写英文字母，以及一些简单的符号。可以看出ASCII码只需要1个字节的存储空间，最高位为0。后被称为（American Standard Code for Information Interchange，美国信息交换标准代码）。它没有特定的编码方式，直接使用地址对应的二进制数来表示，非要说那就叫他ASCII 编码方式。

### :paintbrush: `GBK`

`GBK`全称《汉字内码扩展规范》，支持国际标准`ISO/IEC10646-1`和国家标准`GB13000-1`中的全部中日韩汉字。**`GBK`字符集中所有字符占2个字节，不论中文英文都是2个字节。** 没有特殊的编码方式，习惯称呼`GBK` 编码。一般在国内，汉字较多时使用。

### :paintbrush: ISO-8859-1

ISO-8859-1收录的字符除ASCII收录的字符外，还包括西欧语言、希腊语、泰语、阿拉伯语、希伯来语对应的文字符号。因为ISO-8859-1编码范围使用了单字节内的所有空间，在支持ISO-8859-1的系统中传输和存储其他任何编码的字节流都不会被抛弃。换言之，把其他任何编码的字节流当作ISO-8859-1编码看待都没有问题。这是个很重要的特性，MySQL数据库默认编码是`Latin1`就是利用了这个特性。ASCII编码是一个7位的容器，ISO-8859-1编码是一个8位的容器。由此可见，**ISO-8859-1只占1个字节**，且MySQL数据库默认编码就是ISO-8859-1，有时，tomcat服务器默认也是使用ISO-8859-1编码，然而**ISO-8859-1是不支持中文的**，有时这就是在浏览器上显示乱码的原因。

### :paintbrush: Unicode

从以上几种编码规范可以看出，各种编码规范互不兼容，且只能表示自己需要的字符，于是，国际标准化组织（ISO）决定制定一套**全世界通用的编码规范，这就是Unicode。**

Unicode包含了全世界所有的字符。Unicode最多可以保存4个字节容量的字符。也就是说，要区分每个字符，每个字符的地址需要4个字节。这是十分浪费存储空间的，于是，程序员就设计了几种字符编码方式，比如：`UTF-8`，`UTF-16`，`UTF-32`。

最广为程序员使用的就是`UTF-8`，`UTF-8`是一种变长字符编码，**注意：`UTF-8`不是编码规范，而是编码方式**。

**`UTF-8`的编码规则表：**

|   Unicode 十六进制码点范围  |              UTF-8 二进制              |
| :-----------------: | :---------------------------------: |
| 0000 0000-0000 007F |               0xxxxxxx              |
| 0000 0080-0000 07FF |          110xxxxx 10xxxxxx          |
| 0000 0800-0000 FFFF |      1110xxxx 10xxxxxx 10xxxxxx     |
| 0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |

如上表所示，对于只需要1个字节的字符，`UTF-8`采用ASCII码的编码方式，最高位补0来表示。

例如：`01000001`我们就是用`01000001`来表示，对于一个字节的字符，其实就是直接使用地址表示。

而对于n个字节的字符（n>1），即大于一个字节的字符，采用第一个字节前n位补1。第n+1位填0，后面字节的前两位一律设为10。剩下的没有提及的二进制位，全部为这个符号的Unicode码。

例如：汉字`严`的Unicode码是`4E25`转换成二进制就是`00000000 00000000 01001110 00100101`有效位共15位，根据上表可知使用`UTF-8`字符编码后占3个字节，因此前3位是1，第4位（n+1位）是0，后面两个字节中每个字节的前两位都是10,即`1110 xxxx 10 xxxxxx 10xxxxxx`。填充进去后就变成了`1110 0100 10 111000 10 100101`共计24位占3个字节。

由此可见，**英文在`UTF-8`字符编码后只占1个字节，中文占了3个字节。**

## :pen\_fountain: **编码和解码**

### :paintbrush: **** 解码

一串二进制数，使用一种编码方式，转换成字符，这个过程我们称之为**解码**。就像解开密码一样，程序员可以选用任意的编码方式进行**解码**，但往往只有一种编码方式可以解开密码显示出正确的文字，而**使用错误的编码方式，产生其他不合理的字符，这就是我们通常说的——乱码！**

### :paintbrush: 编码

一串已经解码后的字符，我们也可以选用任意类型的编码方式重新转换成一串二进制数，这个过程就是**编码**，我们也可以称之为加密过程，无论使用哪一种编码方式进行**编码**，最终都是产生计算机可识别的二进制数，**但如果编码规范的字库表不包含目标字符，则无法在字符集中找到对应的二进制数。这将导致不可逆的乱码！例如：像ISO-8859-1的字库表中不包含中文，因此哪怕将中文字符使用ISO-8859-1进行**编码，再使用ISO-8859-1进行解码，也无法显示出正确的中文字符。

因此，乱码就是**编码**和**解码**使用的编码方式不一致，或者**编码**时其字库表中不包含相应字符所导致的结果。

以java语言为例，我们先将一串中文字符串使用UTF-8 编码方式进行**编码**变成字节数组，然后将字节数组打印出来：

```java
String chinese="汉";
//使用UTF-8编码方式进行编码。
byte[] bs = chinese.getBytes("UTF-8");
for (byte b : bs) {
	System.out.print(b+" ");
}
```

结果：

```java
-26 -79 -119
```

可以看出，1个汉字变成了3个字节，证明了1个汉字在`UTF-8`编码方式下占3个字节。继续将字节数组用`UTF-8` 编码方式进行**解码：**

```java
//使用UTF-8编码方式进行解码。
String utf8 = new String(bs,"UTF-8");
System.out.println(utf8);
```

结果：

```java
汉
```

解码后正确的显示了中文字符`汉`字。但如果我们使用`GBK`进行**解码：**

```java
//使用GBK编码方式进行解码。
String gbk = new String(bs, "GBK");
System.out.println(gbk);
123
```

结果：

```
姹?
```

说明使用错误的解码方式就会乱码。

**如果将汉字使用ISO-8859-1进行编码 ，然后解码：**

```java
String chinese = "编码方式";
//使用ISO-8859-1编码方式进行编码。
byte[] bs = chinese.getBytes("ISO-8859-1");
for (byte b : bs) {
	System.out.println(b + " ");
}
//使用ISO-8859-1编码方式进行解码。
String iso = new String(bs, "ISO-8859-1");
System.out.println("\n"+iso);
```

结果：

```java
63 63 63 63 
????
```

可以看出，**无论是哪个汉字，使用ISO-8859-1编码后，都变成了63**。

