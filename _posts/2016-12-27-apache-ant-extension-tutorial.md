---
layout: post
title:  "Apache Ant 扩展教程"
date:   2016-12-27 15:23:04 +0800
tags:   dev--java
---

Apache Ant 致力于成为一款灵活方便的构建工具，尽管对 Java 支持更多，也可以通过一些第三方库来支持其它语言的构建，甚至一些常规维护任务。鉴于Apache Ant 使用 XML 作为配置语言，以描述性见长，而无法处理过于复杂的过程逻辑，因此便有了著名的 Ant-Contrib 扩展包（主页见[这里](http://ant-contrib.sourceforge.net/)）的用武之地。Ant-Contrib 的使用固然增加了 Apache Ant 的可编程性，但以笔者的观点看，违背了 Apache Ant 的设计初衷，同时 XML 本身即使具有了编程能力，传统编程语言的逻辑表现力绝非 XML 可比。事实上，通过其官方 API 扩展 Apache Ant 使其完成用户定制功能，从长远来看，具有更好的简洁性、健壮性、可维护性和稳定性，只不过相对亲切的 XML，阅读 API 的艰巨任务掩盖了扩展 Apache Ant 的优势罢了。

------------------

Apache Ant 构建文件由两部分元素组成，分别是 `Task`（任务） 和 `DataType`（数据类型）。通常而言，类型表示一个资源集合，如`Fileset`（文件集合）；任务用于执行某些操作。虽然任务和类型有很多不同点，但两者从 Java 类结构上看又有很多相似之处。例如：

```java
package packagePath;
import org.apache.tools.ant.Task;  // 任务都继承自这里
import org.apache.tools.ant.BuildException;

/*
 * 使用 Java Bean 规范定义 XML 属性，属性名从 getter/setter 名中推测得到。
 * 若要添加子元素，需要使用 addXXX(YYYY e) 方法。XXX 为子元素的 XML 元素名
 * （在 XML 中不分大小写，但在 Java 中的命名要符合 Java Bean 规范）；
 * YYYY 为其实际的 Java 类名。
 */
public class MyTask extends Task {
    
    private String myStringAttribute;
    private int myIntAttribute;
    private File myFileAttribute;
    
    private ArrayList<SelfDefinedSubElement> l;

    public MyTask() {
        l = new ArrayList<SelfDefinedSubElement>();
    }

    public String getMyStringAttribute() {
        return myStringAttribute;
    }

    // 其它两个 getters ...

    public void setMyStringAttribute(String myStringAttribute) {
        this.myStringAttribute = myStringAttribute;
    }

    // 其它两个 setters ...

    public void addSelfDefinedElement(SelfDefinedSubElement e) {
        l.add(e);
    }

    /*
     * 在这里开始执行任务。DataType 没有这个方法；但 DataType 有获取引用
     * 的方法，即在一个地方使用属性 id 标志数据类型然后在另一个地方用 refid
     * 获得其引用。详询 Apache Ant API
     */
    @Override
    public void execute() {
        if (myStringAttribute == null) {
            throw new BuildException("myStringAttribute not set");
        }

        // 其它输入检查 ...

        // 要完成的操作 ...
    }  
}
```

这是一个任务，一个 `Java` 文件。

```properties
# 在这里定义 MyTask 在 XML 里的元素名
nameUsedByMyTaskInBuildfile=packagePath.MyTask
selfDefinedElement=它的全限定类路径
```

这是任务声明，一个 `propertes` 文件。

```xml
<target name="XXX">
  <!-- some other tasks -->
  <nameUsedByMyTaskInBuildfile myStringAttribute="stringValue"
                               myFileAttribute="C:\Users"
                               myIntAttribute="5">
    <selfDefinedElement someAttributes="" />
  </nameUsedByMyTaskInBuildfile>
  <!-- some other tasks -->
</target>
```

这是该任务所对应的一个可能的 XML 示例。

```java
package anotherPackagePath;
import org.apache.tools.ant.types.DataType;  // 数据类型继承自这里
import org.apache.tools.ant.BuildException;

/*
 * 说明与任务说明相同
 */
public class MyType extends DataType {
    
    private String myStringAttribute;
    private int myIntAttribute;
    private File myFileAttribute;
    
    private ArrayList<AnotherSelfDefinedSubElement> l;

    public MyTask() {
        l = new ArrayList<AnotherSelfDefinedSubElement>();
    }

    public String getMyStringAttribute() {
        return myStringAttribute;
    }

    // 其它两个 getters ...

    public void setMyStringAttribute(String myStringAttribute) {
        this.myStringAttribute = myStringAttribute;
    }

    // 其它两个 setters ...

    public void addAnotherSelfDefinedElement(AnotherSelfDefinedSubElement e) {
        l.add(e);
    }
}
```

这是一个数据类型，一个 `Java` 文件。

```properties
nameUsedByMyTypeInBuildfile=anotherPackagePath.MyType
anotherSelfDefinedElement=它的全限定类路径
```

这是数据类型声明，一个 `propertes` 文件。

```xml
<nameUsedByMyTypeInBuildfile myStringAttribute="stringValue"
                             myFileAttribute="C:\Users"
                             myIntAttribute="5"
                             id="my.id">
  <anotherSelfDefinedElement someAttributes="" />
</nameUsedByMyTypeInBuildfile>
```

这是该数据类型所对应的一个可能的 XML 示例。



----------------

相关阅读：
: [Apache Ant API 的基本使用方法](http://wangbaoaiboy.blog.163.com/blog/static/521119102012123111216547/)
: [Apache Ant API](http://download.csdn.net/detail/Javazzk001/343069)（这是一个下载地址，Apache Ant 不提供官方的在线 API）
