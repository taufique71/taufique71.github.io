---
layout: post
title: First Step with Custom Analysis in Soot Framework
date: 2017-12-22
category: tech
permalink: first-step-with-custom-analysis-in-soot-framework
---

For some personal works, recently I had to learn <a href="https://sable.github.io/soot/">Soot framework</a> - a framework for analyzing and transforming Java and Android applications.
I could manage to use it for my purpose quite well but I faced some problems while learning how to use it.
There are many resources out there in the <a href="https://github.com/sable/soot/wiki">Soot wiki</a> but still I had to invest quite an amount of focus and time to grasp the primary concept of a custom Soot analysis.
I think the reason behind this is lack of resources that helps in taking baby steps with Soot.
So, I thought I should write a post on it for my future references as well as to help the Soot newbies out there who are struggling with same case.  
<br/>

First step is to decide in which pack the analysis is to be placed.
Soot has concepts of packs and phases in it's execution flow.
Details of these concepts can be read in <a href="http://www.bodden.de/2008/11/26/soot-packs/">this blog post</a> of Professor Eric Bodden.
Two packs `jtp` and `wjtp` are involved with custom analysis.
If analysis is intra-procedural one then `jtp` pack is to be used and if the analysis is inter-procedural one then `wjtp` pack is to be used.  
<br/>

Next step is to create a class implementing appropriate interface provided by Soot.
If target pack is `jtp` then `BodyTransformer` interface is to be implemented.
If target pack is `wjtp` then `SceneTransformer` interface is to be implemented.
In both cases method to be implemented is `internalTransform`.
Difference is that in case of `SceneTransformer`, `internalTransform` method takes two arguments while in case of `BodyTransformer`, `internalTransform` method takes three arguments - one extra argument for method body to be analyzed.
So, the implemented class would be like following -  
<br/>

```java
package x;

import soot.*;
import java.util.*;

public class XAnalyzer extends BodyTransformer{
    protected void internalTransform(Body b, String phaseName, Map options){
        // Run analysis
    }
}
```  
<br/>

Next thing to do is to add this new analysis to the appropriate pack of Soot.
To do it, a new main class is to be introduced.
From this main class, new analysis class is added to the appropriate pack and main method of Soot is called to delegate next stages of execution.
Code of this main class would be like following -  
<br/>

```java
package x;

import soot.*;

public class Main{
    public static void main(String args[]){
        // Add the new analysis to appripriate pack of Soot
        soot.PackManager.v()
            .getPack("jtp")
            .add(new Transform("jtp.x", new XAnalyzer()));

        // Delegate next stages of execution to main method of Soot.
        soot.Main.main(args);
    }
}
```  
<br/>

Now, the new main class can be run in following way to see that new analysis is also run in the process.
Usage is same as the usage of Soot as command line tool.  
<br/>

```java
java -cp ./lib/sootclasses-2.5.0.jar:./lib/jasminclasses-2.5.0.jar:./lib/polyglotclasses-1.3.5.jar:. constantanalyzer.Main -cp ./test:. -pp -process-dir ./test/
```  
<br/>

References:
1. <a href="http://www.bodden.de/2008/11/26/soot-packs/">Packs and Phases in Soot</a>
2. <a href="A Survivor's Guide to Java Program Analysis with Soot">A Survivor's Guide to Java Program Analysis with Soot</a>
3. <a href="https://mailman.cs.mcgill.ca/pipermail/soot-list/">Soot Mailing List Archive</a>
