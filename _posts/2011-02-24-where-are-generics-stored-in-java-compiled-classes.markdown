--- 
wordpress_id: 614
layout: post
title: Where are generics stored in java compiled classes?
date: 2011-02-24 18:00:39 +00:00
wordpress_url: http://caprazzi.net/?p=614
---
I was having a go at learning some Java bytecode and started looking at how generics were handled. As expected, the compiler was emitting cast instructions when generic types where _used_, but nothing where the types where _declared_: generics in&nbsp;Java&nbsp;are implemented using a&nbsp;technique&nbsp;called "erasure". Straight from the docs:<blockquote>When a generic type is instantiated, the compiler translates those types by a technique called&nbsp;_type erasure_ — a process where the compiler removes all information related to type parameters and type arguments within a class or method. Type erasure enables Java applications that use generics to maintain binary compatibility with Java libraries and applications that were created before generics.[http://download.oracle.com/javase/tutorial/java/generics/erasure.html](http://download.oracle.com/javase/tutorial/java/generics/erasure.html)</blockquote>But sure not ALL information about the type parameters is lost. That would mean that once I compile my code, all other developer would use it "the old way", with casts and all, but clearly this is not the case. Let's write two simple classes:<pre>``public class GenericClass&lt;T&gt; {}``</pre><pre>``public class StandardClass {}``</pre>And decompile them:<pre>$ javap -c GenericClasspublic class learn.GenericClass extends java.lang.Object{public learn.GenericClass();  Code:   0:   aload_0   1:   invokespecial   #8; //Method java/lang/Object."&lt;init&gt;":()V   4:   return}</pre><pre>$ javap  -c StandardClassCompiled from "StandardClass.java"public class learn.StandardClass extends java.lang.Object{public learn.StandardClass();  Code:   0:   aload_0   1:   invokespecial   #8; //Method java/lang/Object."&lt;init&gt;":()V   4:   return}</pre>As it should, the bytecode looks exactly the same. No type parameters.Let's get more data:<pre>javap -verbose StandardClassCompiled from "StandardClass.java"public class learn.StandardClass extends java.lang.Object  SourceFile: "StandardClass.java"  minor version: 0  major version: 50  Constant pool:const #1 = class        #2;     //  learn/StandardClass// ... snipconst #15 = Asciz       StandardClass.java;// ... snip</pre><pre>$ javap -verbose GenericClassCompiled from "GenericClass.java"public class learn.GenericClass extends java.lang.Object  SourceFile: "GenericClass.java"  **Signature: length = 0x2   00 13**  minor version: 0  major version: 50  Constant pool:const #1 = class        #2;     //  learn/GenericClass// ... snip**const #19 = Asciz       &lt;T:Ljava/lang/Object;&gt;Ljava/lang/Object;;**</pre>// ... snipThe two outputs are different at last: the constant_pool section of a class file has an optional "signature" field. This optional can specify the full signature of the class, including type parameters. The compiler can then use this information to do the right thing when turns sources into binaries.This stuff is specified in the section "4.8.8 The Signature Attribute" of the [updated JVM Specification](http://java.sun.com/docs/books/jvms/second_edition/jvms-clarify.html). 