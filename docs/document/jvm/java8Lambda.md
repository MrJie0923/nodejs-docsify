### Lambda表达式

### http://blog.didispace.com/books/java8-tutorial/ch8.html

---

> java8中提供了接口默认实现方法、Lambda表达式、方法引用和重复注解、新版本的API：流控制(stream)、函数式编程、map扩展和新的时间日期API等

##### 集合排序

* Java8之前实现

  ```java
  List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");
  
  Collections.sort(names, new Comparator<String>() {
      @Override
      public int compare(String a, String b) {
          return b.compareTo(a);
      }
  });
  ```

  

* Java8 Lamabda实现

  ```java
  Collections.sort(names, (String a, String b) -> b.compareTo(a));
  ```

  

##### 函数式接口

Lambda表达式如何匹配Java的类型系统？每一个lambda都能够通过一个特定的接口，与一个给定的类型进行匹配。一个所谓的函数式接口必须要有且仅有一个抽象方法声明。每个与之对应的lambda表达式必须要与抽象方法的声明相匹配。由于默认方法不是抽象的，因此你可以在你的函数式接口里任意添加默认方法。

任意只包含一个抽象方法的接口，我们都可以用来做成lambda表达式。为了让你定义的接口满足要求，你应当在接口前加上@FunctionalInterface 标注。编译器会注意到这个标注，如果你的接口中定义了第二个抽象方法的话，编译器会抛出异常。

举例：

```java
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}

Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
Integer converted = converter.convert("123");
System.out.println(converted);    // 123
```

##### 方法和构造函数引用

上面的代码实例可以通过静态方法引用，使之更加简洁：

```java
Converter<String, Integer> converter = Integer::valueOf;
Integer converted = converter.convert("123");
System.out.println(converted);   // 123
```

##### 内置函数式接口

JDK 1.8 API中包含了很多内置的函数式接口。有些是在以前版本的Java中大家耳熟能详的，例如Comparator接口，或者Runnable接口。对这些现成的接口进行实现，可以通过@FunctionalInterface 标注来启用Lambda功能支持。

此外，Java 8 API 还提供了很多新的函数式接口，来降低程序员的工作负担。有些新的接口已经在[Google Guava](https://code.google.com/p/guava-libraries/)库中很有名了。如果你对这些库很熟的话，你甚至闭上眼睛都能够想到，这些接口在类库的实现过程中起了多么大的作用。

* #### Predicates

  Predicate是一个布尔类型的函数，该函数只有一个输入参数。Predicate接口包含了多种默认方法，用于处理复杂的逻辑动词（and, or，negate）

  ```java
  Predicate<String> predicate = (s) -> s.length() > 0;
  
  predicate.test("foo");              // true
  predicate.negate().test("foo");     // false
  
  Predicate<Boolean> nonNull = Objects::nonNull;
  Predicate<Boolean> isNull = Objects::isNull;
  
  Predicate<String> isEmpty = String::isEmpty;
  Predicate<String> isNotEmpty = isEmpty.negate();
  ```

* #### Functions

  Function接口接收一个参数，并返回单一的结果。默认方法可以将多个函数串在一起（compse, andThen）

  ```java
  Function<String, Integer> toInteger = Integer::valueOf;
  Function<String, String> backToString = toInteger.andThen(String::valueOf);
  
  backToString.apply("123");     // "123"
  ```

* #### Suppliers

  Supplier接口产生一个给定类型的结果。与Function不同的是，Supplier没有输入参数。

  ```java
  Supplier<Person> personSupplier = Person::new;
  personSupplier.get();   // new Person
  ```

* #### Consumers

  Consumer代表了在一个输入参数上需要进行的操作。

  ```java
  Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
  greeter.accept(new Person("Luke", "Skywalker"));
  ```

* #### Comparators

  Comparator接口在早期的Java版本中非常著名。Java 8 为这个接口添加了不同的默认方法。

  ```java
  Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);
  
  Person p1 = new Person("John", "Doe");
  Person p2 = new Person("Alice", "Wonderland");
  
  comparator.compare(p1, p2);             // > 0
  comparator.reversed().compare(p1, p2);  // < 0
  ```

* #### Optionals

  Optional不是一个函数式接口，而是一个精巧的工具接口，用来防止NullPointerException产生。这个概念在下一节会显得很重要，所以我们在这里快速地浏览一下Optional的工作原理。

  Optional是一个简单的值容器，这个值可以是null，也可以是non-null。考虑到一个方法可能会返回一个non-null的值，也可能返回一个空值。为了不直接返回null，我们在Java 8中就返回一个Optional.

  ```java
  Optional<String> optional = Optional.of("bam");
  
  optional.isPresent();           // true
  optional.get();                 // "bam"
  optional.orElse("fallback");    // "bam"
  
  optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
  ```



#### Streams

java.util.Stream表示了某一种元素的序列，在这些元素上可以进行各种操作。Stream操作可以是中间操作，也可以是完结操作。完结操作会返回一个某种类型的值，而中间操作会返回流对象本身，并且你可以通过多次调用同一个流操作方法来将操作结果串起来（就像StringBuffer的append方法一样————译者注）。Stream是在一个源的基础上创建出来的，例如java.util.Collection中的list或者set（map不能作为Stream的源）。Stream操作往往可以通过顺序或者并行两种方式来执行。

我们先了解一下序列流。首先，我们通过string类型的list的形式创建示例数据：

```java
List<String> stringCollection = new ArrayList<>();
stringCollection.add("ddd2");
stringCollection.add("aaa2");
stringCollection.add("bbb1");
stringCollection.add("aaa1");
stringCollection.add("bbb3");
stringCollection.add("ccc");
stringCollection.add("bbb2");
stringCollection.add("ddd1");
```

Java 8中的Collections类的功能已经有所增强，你可以之直接通过调用Collections.stream()或者Collection.parallelStream()方法来创建一个流对象。下面的章节会解释这个最常用的操作。

* #### Filter

  接受一个predicate接口类型的变量，并将所有流对象中的元素进行过滤。该操作是一个中间操作，因此它允许我们在返回结果的基础上再进行其他的流操作（forEach）。ForEach接受一个function接口类型的变量，用来执行对每一个元素的操作。ForEach是一个中止操作。它不返回流，所以我们不能再调用其他的流操作。

  ```java
  stringCollection
      .stream()
      .filter((s) -> s.startsWith("a"))
      .forEach(System.out::println);
  
  // "aaa2", "aaa1"
  ```

* #### Sorted

  Sorted是一个中间操作，能够返回一个排过序的流对象的视图。流对象中的元素会默认按照自然顺序进行排序，除非你自己指定一个Comparator接口来改变排序规则。

  ```java
  stringCollection
      .stream()
      .sorted()
      .filter((s) -> s.startsWith("a"))
      .forEach(System.out::println);
  
  // "aaa1", "aaa2"
  ```

  一定要记住，sorted只是创建一个流对象排序的视图，而不会改变原来集合中元素的顺序。原来string集合中的元素顺序是没有改变的。

  ```java
  System.out.println(stringCollection);
  // ddd2, aaa2, bbb1, aaa1, bbb3, ccc, bbb2, ddd1
  ```

* #### Map

  map是一个对于流对象的中间操作，通过给定的方法，它能够把流对象中的每一个元素对应到另外一个对象上。下面的例子就演示了如何把每个string都转换成大写的string. 不但如此，你还可以把每一种对象映射成为其他类型。对于带泛型结果的流对象，具体的类型还要由传递给map的泛型方法来决定。

  ```java
  stringCollection
      .stream()
      .map(String::toUpperCase)
      .sorted((a, b) -> b.compareTo(a))
      .forEach(System.out::println);
  
  // "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"
  ```

* #### Match

  匹配操作有多种不同的类型，都是用来判断某一种规则是否与流对象相互吻合的。所有的匹配操作都是终结操作，只返回一个boolean类型的结果。

  ```java
  boolean anyStartsWithA =
      stringCollection
          .stream()
          .anyMatch((s) -> s.startsWith("a"));
  
  System.out.println(anyStartsWithA);      // true
  
  boolean allStartsWithA =
      stringCollection
          .stream()
          .allMatch((s) -> s.startsWith("a"));
  
  System.out.println(allStartsWithA);      // false
  
  boolean noneStartsWithZ =
      stringCollection
          .stream()
          .noneMatch((s) -> s.startsWith("z"));
  
  System.out.println(noneStartsWithZ);      // true
  ```

* #### Count

  Count是一个终结操作，它的作用是返回一个数值，用来标识当前流对象中包含的元素数量。

  ```java
  long startsWithB =
      stringCollection
          .stream()
          .filter((s) -> s.startsWith("b"))
          .count();
  
  System.out.println(startsWithB);    // 3
  ```

* #### Reduce

  该操作是一个终结操作，它能够通过某一个方法，对元素进行削减操作。该操作的结果会放在一个Optional变量里返回。

  ```java
  Optional<String> reduced =
      stringCollection
          .stream()
          .sorted()
          .reduce((s1, s2) -> s1 + "#" + s2);
  
  reduced.ifPresent(System.out::println);
  // "aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2"
  ```

* ### Parallel Streams

  像上面所说的，流操作可以是顺序的，也可以是并行的。顺序操作通过单线程执行，而并行操作则通过多线程执行。

  下面的例子就演示了如何使用并行流进行操作来提高运行效率，代码非常简单。

  首先我们创建一个大的list，里面的元素都是唯一的：

  ```java
  int max = 1000000;
  List<String> values = new ArrayList<>(max);
  for (int i = 0; i < max; i++) {
      UUID uuid = UUID.randomUUID();
      values.add(uuid.toString());
  }
  ```

  现在，我们测量一下对这个集合进行排序所使用的时间。

#### Map

正如前面已经提到的那样，map是不支持流操作的。而更新后的map现在则支持多种实用的新方法，来完成常规的任务。

```java
Map<Integer, String> map = new HashMap<>();

for (int i = 0; i < 10; i++) {
    map.putIfAbsent(i, "val" + i);
}

map.forEach((id, val) -> System.out.println(val));
```