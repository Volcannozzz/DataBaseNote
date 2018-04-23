# WeakHashMap 指南

### 1. 概览

在这篇文章中，我们将会讨论 ***java.util***包中的 ***WeakHashMap***类。

为了能够理解 ***WeakHashMap***这种数据结构，我们将会使用它来实现一个简单的缓存功能。然而，请谨记，该缓存功能仅仅是帮助理解该类的数据结构；自己去实现一个缓存往往并不是很值得的。

简单说， ***WeakHashMap***是基于 ***hashtable***的 ，对***Map***接口的实现，其 ***key***是 ***WeakReference***类型。

在日常使用中，当 ***WeakHashMap***中一个entry的key不再被使用的时候，该entry会被从 ***WeakHashMap***中移除，意即没有指向那个key的单独的引用。当GC回收一个key的时候，该key所在的entry也会被很有效率的从map中移除。所以这个类的行为与其他 ***Map***的实现还是存在一些不同的。



### 2. 强引用，软引用和弱引用

为了理解 ***WeakHashMap***是如何工作的，需要了解一下 ***WeakReference***这个类，这个类是 ***WeakHashMap***中key的基本结构。在 ***JAVA***中，有三个主要类型的引用，在下面的小节中会详细叙述。

#### 2.1 强引用

强引用是我们日常编程中最常用的一种引用类型：

```java
Integer prime = 1;
```

***prime***持有一个值为1的对象的强引用。任何对象，只要有一个强引用指向它，那么就不满足GC条件。

#### 2.2 软引用

简单来说，一个对象，如果有一个软引用指向它，直到JVM需要内存的时候才会被GC。

下面是Java中创建一个软引用的代码片段：

```java
Integer prime = 1;
SoftReference<Integer> soft = new SoftReference<Integer>(prime);
prime = null;
```

prime是指向对象的强引用。

接下来，将prime包装进入一个软引用。然后将强引用赋值为null，prime对象现在是满足GC条件的了，但是只有当JVM需要内存的时候才会被回收。

#### 2.3 弱引用

只有被弱引用所持有的对象是对GC友好的（意即很容易被收集）；在这种情况下，对于该对象的GC，发生时机不会延迟到JVM需要内存时。

下面是弱引用的一个例子：

```java
Integer prime = 1;
WeakReference<Integer> weak = new WeakReference<>(prime);
prime = null;
```

当prime被赋值为null时，prime对象会在下个GC周期被回收，因为没有强引用指向它了。弱引用被用作 ***WeakHashMap***中作为key的基本类型。

### 3. 使用 ***WeakHashMap*** 实现一个有效率的内存缓存功能

在缓存中，我们需要将一个较大的图片对象作为value，图片名字作为key。接下来，我们为这个问题选择一个合适的map实现。

简单的使用 ***HashMap***不是一个很好的选择。因为value对象可能会占据很大的内存空间。更重要的是，他们从来不会被GC回收，尽管这些对象再也不会被使用了。

理想情况下，我们想要这样一种 ***Map***的实现：允许GC自动删除回收不再使用的对象。当应用中的一个大图片对象的key不再被使用的时候，对应的entry会被从内存中删除。

幸运的是， ***WeakHashMap***恰好具备这些特性。下面看一下：

```java
WeakHashMap<UniqueImageName, BigImage> map = new WeakHashMap<>();
BigImage bigImage = new BigImage("image_id");
UniqueImageName imageName = new UniqueImageName("name_of_big_image");
 
map.put(imageName, bigImage);
assertTrue(map.containsKey(imageName));
 
imageName = null;
System.gc();
 
await().atMost(10, TimeUnit.SECONDS).until(map::isEmpty);
```

我们创建一个 ***WeakHashMap***对象用来存储 ***BigImage***对象。将 ***BigImage***作为值，一个 ***imageName***对象作为key。 ***imageName***以 ***WeakReference* **的类型存在。

接下来，设置*imageName* 为null，这样就没有任何引用指向 *bigImage* 对象了。 *WeakReference* 的默认行为是会在下一个GC中将entry回收，因此在下一个GC过程中， 这个entry会被删除。

调用 *System.gc()* 强制JVM触发GC，在GC之后， *WeakHashMap* 会变为空：

```java
WeakHashMap<UniqueImageName, BigImage> map = new WeakHashMap<>();
BigImage bigImageFirst = new BigImage("foo");
UniqueImageName imageNameFirst = new UniqueImageName("name_of_big_image");
 
BigImage bigImageSecond = new BigImage("foo_2");
UniqueImageName imageNameSecond = new UniqueImageName("name_of_big_image_2");
 
map.put(imageNameFirst, bigImageFirst);
map.put(imageNameSecond, bigImageSecond);
  
assertTrue(map.containsKey(imageNameFirst));
assertTrue(map.containsKey(imageNameSecond));
 
imageNameFirst = null;
System.gc();
 
await().atMost(10, TimeUnit.SECONDS)
  .until(() -> map.size() == 1);
await().atMost(10, TimeUnit.SECONDS)
  .until(() -> map.containsKey(imageNameSecond));
```

只有 *imageNameFirst* 引用被设置为null。*imageNameSecond* 并没有改变，在GC之后， map将只会持有一个entry--------*imageNameSecond*。

### 4. 总结

在这篇文章中，我们学习了JAVA重点引用类型，并完全理解了WeakHashMap 是如何工作的。我们利用***WeakHashMap ***创建了一个简单的缓存并且测试了他是否能够预期的进行工作。

****