# 37|实战二（下）：重构ID生成器项目中各函数的异常处理代码

![](<../.gitbook/assets/image (1).png>)

平时进行软件设计开发的时候，我们除了要保证正常情况下的逻辑运行正确之外，还需要编写大量额外的代码，来处理有可能出现的异常情况，以保证代码在任何情况下，都在我们的掌控之内，不会出现非预期的运行结果。程序的 bug 往往都出现在一些边界条件和异常情况下，所以说，异常处理得好坏直接影响了代码的健壮性。全面、合理地处理各种异常能有效减少代码 bug，也是保证代码质量的一个重要手段。

在上一节课中，我们讲解了几种异常情况的处理方式，比如返回错误码、NULL 值、空对象、异常对象。针对最常用的异常对象，我们还重点讲解了两种异常类型的应用场景，以及针对函数抛出的异常的三种处理方式：直接吞掉、原封不动地抛出和包裹成新的异常抛出。

除此之外，在上一节课的开头，我们还针对 ID 生成器的代码，提出了 4 个有关异常处理的问题。今天，我们就用一节课的时间，结合上一节课讲到的理论知识，来逐一解答一下这几个问题。

## 重构 generate() 函数

首先，我们来看，对于 generate() 函数，如果本机名获取失败，函数返回什么？这样的返回值是否合理？

```java
public String generate() {
    String substrOfHostName = getLastFieldOfHostName();
    long currentTimeMillis = System.currentTimeMillis();
    String randomString = generateRandomAlphameric(8);
    String id = String.format("%s-%d-%s",
            substrOfHostName, currentTimeMillis, randomString);
    return id;
  }
```

ID 由三部分构成：本机名、时间戳和随机数。时间戳和随机数的生成函数不会出错，唯独主机名有可能获取失败。在目前的代码实现中，如果主机名获取失败，substrOfHostName 为 NULL，那 generate() 函数会返回类似“null-16723733647-83Ab3uK6”这样的数据。如果主机名获取失败，substrOfHostName 为空字符串，那 generate() 函数会返回类似“-16723733647-83Ab3uK6”这样的数据。

在异常情况下，返回上面两种特殊的 ID 数据格式，这样的做法是否合理呢？这个其实很难讲，我们要看具体的业务是怎么设计的。不过，我更倾向于明确地将异常告知调用者。所以，这里最好是抛出受检异常，而非特殊值。

按照这个设计思路，我们对 generate() 函数进行重构。重构之后的代码如下所示：

```java
  public String generate() throws IdGenerationFailureException {
    String substrOfHostName = getLastFieldOfHostName();
    if (substrOfHostName == null || substrOfHostName.isEmpty()) {
      throw new IdGenerationFailureException("host name is empty.");
    }
    long currentTimeMillis = System.currentTimeMillis();
    String randomString = generateRandomAlphameric(8);
    String id = String.format("%s-%d-%s",
            substrOfHostName, currentTimeMillis, randomString);
    return id;
  }
```

## 重构 getLastFieldOfHostName() 函数

对于 getLastFieldOfHostName() 函数，是否应该将 UnknownHostException 异常在函数内部吞掉（try-catch 并打印日志），还是应该将异常继续往上抛出？如果往上抛出的话，是直接把 UnknownHostException 异常原封不动地抛出，还是封装成新的异常抛出？

```java
  private String getLastFieldOfHostName() {
    String substrOfHostName = null;
    try {
      String hostName = InetAddress.getLocalHost().getHostName();
      substrOfHostName = getLastSubstrSplittedByDot(hostName);
    } catch (UnknownHostException e) {
      logger.warn("Failed to get the host name.", e);
    }
    return substrOfHostName;
 }
```

现在的处理方式是当主机名获取失败的时候，getLastFieldOfHostName() 函数返回 NULL 值。我们前面讲过，是返回 NULL 值还是异常对象，要看获取不到数据是正常行为，还是异常行为。获取主机名失败会影响后续逻辑的处理，并不是我们期望的，所以，它是一种异常行为。这里最好是抛出异常，而非返回 NULL 值。

至于是直接将 UnknownHostException 抛出，还是重新封装成新的异常抛出，要看函数跟异常是否有业务相关性。getLastFieldOfHostName() 函数用来获取主机名的最后一个字段，UnknownHostException 异常表示主机名获取失败，两者算是业务相关，所以可以直接将 UnknownHostException 抛出，不需要重新包裹成新的异常。

按照上面的设计思路，我们对 getLastFieldOfHostName() 函数进行重构。重构后的代码如下所示：

```java
 private String getLastFieldOfHostName() throws UnknownHostException{
    String substrOfHostName = null;
    String hostName = InetAddress.getLocalHost().getHostName();
    substrOfHostName = getLastSubstrSplittedByDot(hostName);
    return substrOfHostName;
 }
```

getLastFieldOfHostName() 函数修改之后，generate() 函数也要做相应的修改。我们需要在 generate() 函数中，捕获 getLastFieldOfHostName() 抛出的 UnknownHostException 异常。当我们捕获到这个异常之后，应该怎么处理呢？

按照之前的分析，ID 生成失败的时候，我们需要明确地告知调用者。所以，我们不能在 generate() 函数中，将 UnknownHostException 这个异常吞掉。那我们应该原封不动地抛出，还是封装成新的异常抛出呢？

我们选择后者。在 generate() 函数中，我们需要捕获 UnknownHostException 异常，并重新包裹成新的异常 IdGenerationFailureException 往上抛出。之所以这么做，有下面三个原因。

* 调用者在使用 generate() 函数的时候，只需要知道它生成的是随机唯一 ID，并不关心 ID 是如何生成的。也就说是，这是**依赖抽象而非实现编程**。如果 generate() 函数直接抛出 UnknownHostException 异常，实际上是暴露了实现细节。
* 从代码封装的角度来讲，我们不希望将 UnknownHostException 这个比较底层的异常，暴露给更上层的代码，也就是调用 generate() 函数的代码。而且，调用者拿到这个异常的时候，并不能理解这个异常到底代表了什么，也不知道该如何处理。
* UnknownHostException 异常跟 generate() 函数，在业务概念上没有相关性。

按照上面的设计思路，我们对 generate() 的函数再次进行重构。重构后的代码如下所示：

```java
public String generate() throws IdGenerationFailureException {
    String substrOfHostName = null;
    try {
      substrOfHostName = getLastFieldOfHostName();
    } catch (UnknownHostException e) {
      throw new IdGenerationFailureException("host name is empty.");
    }
    long currentTimeMillis = System.currentTimeMillis();
    String randomString = generateRandomAlphameric(8);
    String id = String.format("%s-%d-%s",
            substrOfHostName, currentTimeMillis, randomString);
    return id;
  }
```

## 重构 getLastSubstrSplittedByDot() 函数

对于 getLastSubstrSplittedByDot(String hostName) 函数，如果 hostName 为 NULL 或者空字符串，这个函数应该返回什么？

```java
  @VisibleForTesting
  protected String getLastSubstrSplittedByDot(String hostName) {
    String[] tokens = hostName.split("\\.");
    String substrOfHostName = tokens[tokens.length - 1];
    return substrOfHostName;
  }
```

理论上讲，参数传递的正确性应该有程序员来保证，我们无需做 NULL 值或者空字符串的判断和特殊处理。调用者本不应该把 NULL 值或者空字符串传递给 getLastSubstrSplittedByDot() 函数。如果传递了，那就是 code bug，需要修复。但是，话说回来，谁也保证不了程序员就一定不会传递 NULL 值或者空字符串。那我们到底该不该做 NULL 值或空字符串的判断呢？

如果**函数是 private 类私有的，只在类内部被调用，完全在你自己的掌控之下，自己保证在调用这个 private 函数的时候，不要传递 NULL 值或空字符串就可以了**。所以，我们可以不在 private 函数中做 NULL 值或空字符串的判断。**如果函数是 public 的，**你无法掌控会被谁调用以及如何调用（有可能某个同事一时疏忽，传递进了 NULL 值，这种情况也是存在的），为了尽可能提高代码的健壮性，**我们最好是在 public 函数中做 NULL 值或空字符串的判断。**

那你可能会说，getLastSubstrSplittedByDot() 是 protected 的，既不是 private 函数，也不是 public 函数，那要不要做 NULL 值或空字符串的判断呢？

之所以将它设置为 protected，是为了方便写单元测试。不过，单元测试可能要测试一些 corner case，比如输入是 NULL 值或者空字符串的情况。所以，这里我们最好也加上 NULL 值或空字符串的判断逻辑。虽然加上有些冗余，但多加些检验总归不会错的。

按照这个设计思路，我们对 getLastSubstrSplittedByDot() 函数进行重构。重构之后的代码如下所示：

```java
@VisibleForTesting
  protected String getLastSubstrSplittedByDot(String hostName) {
    if (hostName == null || hostName.isEmpty()) {
      throw IllegalArgumentException("..."); //运行时异常
    }
    String[] tokens = hostName.split("\\.");
    String substrOfHostName = tokens[tokens.length - 1];
    return substrOfHostName;
  }
```

按照上面讲的，我们在使用这个函数的时候，自己也要保证不传递 NULL 值或者空字符串进去。所以，getLastFieldOfHostName() 函数的代码也要作相应的修改。修改之后的代码如下所示：

```java
 private String getLastFieldOfHostName() throws UnknownHostException{
    String substrOfHostName = null;
    String hostName = InetAddress.getLocalHost().getHostName();
    if (hostName == null || hostName.isEmpty()) { // 此处做判断
      throw new UnknownHostException("...");
    }
    substrOfHostName = getLastSubstrSplittedByDot(hostName);
    return substrOfHostName;
 }
```

## 重构 generateRandomAlphameric() 函数

对于 generateRandomAlphameric(int length) 函数，如果 length < 0 或 length = 0，这个函数应该返回什么？

```java
  @VisibleForTesting
  protected String generateRandomAlphameric(int length) {
    char[] randomChars = new char[length];
    int count = 0;
    Random random = new Random();
    while (count < length) {
      int maxAscii = 'z';
      int randomAscii = random.nextInt(maxAscii);
      boolean isDigit= randomAscii >= '0' && randomAscii <= '9';
      boolean isUppercase= randomAscii >= 'A' && randomAscii <= 'Z';
      boolean isLowercase= randomAscii >= 'a' && randomAscii <= 'z';
      if (isDigit|| isUppercase || isLowercase) {
        randomChars[count] = (char) (randomAscii);
        ++count;
      }
    }
    return new String(randomChars);
  }
}
```

我们先来看 length < 0 的情况。生成一个长度为负值的随机字符串是不符合常规逻辑的，是一种异常行为。所以，当传入的参数 length < 0 的时候，我们抛出 IllegalArgumentException 异常。

我们再来看 length = 0 的情况。length = 0 是否是异常行为呢？这就看你自己怎么定义了。我们既可以把它定义为一种异常行为，抛出 IllegalArgumentException 异常，也可以把它定义为一种正常行为，让函数在入参 length = 0 的情况下，直接返回空字符串。**不管选择哪种处理方式，最关键的一点是，要在函数注释中，明确告知 length = 0 的情况下，会返回什么样的数据。**

## 重构之后的 RandomIdGenerator 代码

对 RandomIdGenerator 类中各个函数异常情况处理代码的重构，到此就结束了。为了方便查看，我把重构之后的代码，重新整理之后贴在这里了。你可以对比着看一下，跟你的重构思路是否一致。

```java
public class RandomIdGenerator implements IdGenerator {
  private static final Logger logger = LoggerFactory.getLogger(RandomIdGenerator.class);

  @Override
  public String generate() throws IdGenerationFailureException {
    String substrOfHostName = null;
    try {
      substrOfHostName = getLastFieldOfHostName();
    } catch (UnknownHostException e) {
      throw new IdGenerationFailureException("...", e);
    }
    long currentTimeMillis = System.currentTimeMillis();
    String randomString = generateRandomAlphameric(8);
    String id = String.format("%s-%d-%s",
            substrOfHostName, currentTimeMillis, randomString);
    return id;
  }

  private String getLastFieldOfHostName() throws UnknownHostException{
    String substrOfHostName = null;
    String hostName = InetAddress.getLocalHost().getHostName();
    if (hostName == null || hostName.isEmpty()) {
      throw new UnknownHostException("...");
    }
    substrOfHostName = getLastSubstrSplittedByDot(hostName);
    return substrOfHostName;
  }

  @VisibleForTesting
  protected String getLastSubstrSplittedByDot(String hostName) {
    if (hostName == null || hostName.isEmpty()) {
      throw new IllegalArgumentException("...");
    }

    String[] tokens = hostName.split("\\.");
    String substrOfHostName = tokens[tokens.length - 1];
    return substrOfHostName;
  }

  @VisibleForTesting
  protected String generateRandomAlphameric(int length) {
    if (length <= 0) {
      throw new IllegalArgumentException("...");
    }

    char[] randomChars = new char[length];
    int count = 0;
    Random random = new Random();
    while (count < length) {
      int maxAscii = 'z';
      int randomAscii = random.nextInt(maxAscii);
      boolean isDigit= randomAscii >= '0' && randomAscii <= '9';
      boolean isUppercase= randomAscii >= 'A' && randomAscii <= 'Z';
      boolean isLowercase= randomAscii >= 'a' && randomAscii <= 'z';
      if (isDigit|| isUppercase || isLowercase) {
        randomChars[count] = (char) (randomAscii);
        ++count;
      }
    }
    return new String(randomChars);
  }
}
```

## 重点回顾

好了，今天的内容到此就讲完了。我们一块来总结回顾一下，你需要重点掌握的内容。

今天的内容比较偏实战，是对上节课学到的理论知识的一个应用。从今天的实战中，你学到了哪些更高层的软件设计和开发思想呢？我这里抛砖引玉，总结了下面 3 点。

* 再简单的代码，看上去再完美的代码，只要我们下功夫去推敲，总有可以优化的空间，就看你愿不愿把事情做到极致。
*   如果你内功不够深厚，理论知识不够扎实，那你就很难参透开源项目的代码到底优秀在哪里。就像如果我们没有之前的理论学习，没有今天我给你一点一点重构、讲解、分析，只是给你最后重构好的 Rando

    mIdGenerator 的代码，你真的

    能学到它的设计精髓吗？
*   对比第 34 节课

    最初小王的 IdGenerator 代码和最终的 RandomIdGenerator 代码，它们一个是“能用”，一个是“好用”，天壤之别。作为一名程序员，起码对代码要有追求啊，不然跟咸鱼有啥区别！

## 课堂讨论

我们花了 4 节课的时间，对一个非常简单的、不到 40 行的 ID 生成器代码，做了多次迭代重构。除了刚刚我在“重点回顾”中讲到的那几点之外，从这个迭代重构的过程中，你还学到哪些更有价值的东西？
