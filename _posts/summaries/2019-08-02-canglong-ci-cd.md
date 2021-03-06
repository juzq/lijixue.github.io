---
layout: post
title: 苍龙三国服务器的持续集成、交付、部署之路
date: 2019-08-02 8:00:01 +0800
categories: 总结归纳
tag: DevOps
---

* content
{:toc}

# 前言 {#preface}

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/1.jpg)

&emsp;&emsp;如今，持续集成已成为软件开发的标准化流程，也是敏捷开发的重要组成一环。所谓持续集成，是指软件开发团队成员定期（每天甚至更短）将自己的开发成果合并到产品项目中，通过自动化构建（编译、发布、自动化测试等）验证本次开发成果，尽可能快地发现错误与漏洞（可能是本次开发内容本身的问题，也可能是开发内容与产品现有内容之间的兼容性问题）。

&emsp;&emsp;而持续交付，则是指定期将产品新版本交付给质量团队或用户评审与测试，以尽早发现产品缺陷，为产品进入生产环境之前进行风险评估与故障排除。在游戏开发流程中，持续交付则表现为，将服务器/客户端最新代码（资源）更新到测试服，以供策划/QA团队测试。

&emsp;&emsp;最后持续部署，是指上一步交付的代码通过评审之后，能够自动部署到生产环境。他的目标是，代码在任何时刻都是可部署的，可以进入生产阶段。在游戏开发流程中，持续部署则表现为，将服务器/客户端代码（资源）更新到正式服。

&emsp;&emsp;开发团队通过持续集成、交付、部署，可以大大减少功能集成问题，提高团队的开发效率，使团队能开发出高质量的软件产品。

&emsp;&emsp;既然持续集成如此重要，那为什么"苍龙"这款产品一直没有做持续集成等相关流程呢？这就要从项目历史说起。

# 项目历史 {#history}

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/13.png)

&emsp;&emsp;苍龙项目组立项于2014年初，苍龙在研发初期，把专注力都放在产品本身，为了产品的快速上线，高度重(chóng)用了公司已有项目的开发成果。但在部署运维方面，仍然是用的非常原始的手段：手动编译、手动打包、手动更新...。好消息是，苍龙因为研发周期短，在其他同类游戏出现之前占领了“写实三国卡牌”这个新兴市场，在日本、韩国、东南亚都取得了不错的成绩。但是，因为包括持续集成等在内的流程等方面的不足，也给项目组带来了很多的问题与困扰。

# 问题 {#problems}
1.  游戏产品数值变动频繁，但策划没有自主更新测试服配置表的途径。过分依赖服务器程序员帮助其更新配置表，增加了更新配置表的时间消耗，也严重影响的服务器程序员的工作效率。
2.  手动编译、打包、更新对于服务器程序员来讲，不但费时费力，而且操作过程容易出错。（苍龙历史上曾出现多次因为更新时代码未编译而导致的临时维护事故）
3.  没有自动化测试流程（单元测试），仅依靠功能验收时的黑盒测试与功能更新时的冒烟测试，并不能完全保证代码质量。
4.  更新正式服仍依靠手动将更新内容上传到指定位置，然后运维进行更新操作，并不能保证更新内容在正式服与测试服完全一致（后来虽然增加了md5校验，但上传与更新过程仍然费时费力）
5.  更新正式服需要运维人员参与，开发人员需要在每次维护时与运维人员对接维护内容，会出现因运维人员的疏忽而漏更新某些服务器。

# 雏形：配置表发布工具 {#base-tool}

&emsp;&emsp;问题如此之多，需要一步一步来解决。最基础也是最亟待解决的问题就是问题1（策划自主更新测试服配置表），因为这严重影响了策划与服务器程序员的工作效率。时间还是2016年，当时我刚来到苍龙项目组，接到的第一个任务就是在一周之内开发出一套能交付给策划使用更新测试服配置表的工具。在梳理完服务器更新流程后，大致理了一下实现步骤：远程调用服务器脚本关闭服务器 -> 上传策划需要发布的配置表 -> 远程启动服务器。因为时间要求比较紧，所以只做了一个控制台版，效果如下图：

![2](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/2.png)

&emsp;&emsp;虽然比较简陋，界面也不太友好，但已经能基本解决策划更新配置表的问题。后来也增加了代码打包，发布代码等功能。

&emsp;&emsp;后记：可能是思维定式的限制，只按照任务要求做了工具解决更新更新问题，并没有想到使用jenkins来引入自动化流程，而是继续着手优化、迭代该发布工具，致使苍龙完整的持续集成、交付、部署工作的延后，这也是比较失误的地方，应当引以为戒。

# 优化：图形界面版本 {#gui-tool}

&emsp;&emsp;虽然已经有了测试服发布工具，但是控制台窗口界面不友好，操作不便，连我自己都比较嫌弃它，更何况是其他人。于是便有了制作图形界面版本的想法，在调研了策划、程序等多方需求后，我利用工作空档开始了图形界面版开发。因为桌面软件重在精简，上百m的jre环境让我放弃了继续使用java作为开发语言的想法，而是转向了python。python图形界面开发框架有wxpython，ssh库有paramiko，windows环境打包有pyinstaller，完全符合我的预期，因此图形界面版就此诞生。

![3](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/3.png)

&emsp;&emsp;该版本通过界面选择jdk路径，实现了代码自动编译、打包、上传、更新，基本实现了持续集成、交付的工作流程，同时解决了问题2（更新测试服过程繁琐、易出错的问题）。

> 代码已上传至GitLab，有兴趣可以前往 [查看](https://git.ppgame.com/lijixue/sgcard-server-publish-tool)

# 持续集成、交付与部署 {#ci-cd}

&emsp;&emsp;有了上述持续集成的流程之后，我又在思考持续部署应该怎么做，但是发现好像进入了死胡同。因为使用自定义工具方式的持续集成与交付，代码并没有提交，并不能保证更新的测试服的代码是完整和最新的，所以运维在更新正式服时，也不能直接使用测试服的代码，因此，已有的自定义工具更新无法完成持续部署等后续流程。

## Jenkins {#jenkins}

### Jenkins介绍 {#jenkins-desc}

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/4.png)

&emsp;&emsp;[Jenkins](https://jenkins.io/zh/)是一款开源 CI&CD （持续构建与部署）软件，用于自动化各种任务，包括构建、测试和部署软件。可以使用Maven来构建Java应用，用npm来构建Node.js与React应用，用PyInstaller来构建python应用等等。安装和使用Jenkins也非常简单，[官网](https://jenkins.io/zh/doc/book/installing/)详细介绍了在不同平台的多种安装方法，我选择了最方便的使用war包来运行，因为服务器自带有JDK环境。

### 使用Jenkins {#use-jenkins}

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/11.png)

&emsp;&emsp;jenkins的使用非常简单，只要按照提示一步步进行即可。这里选择最常用的“构建一个自由风格的软件项目”，如果后续步骤比较复杂，可以考虑使用流水线。

### 现有的构建方式 or Maven改造？ {#script-or-maven}

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/12.png)

&emsp;&emsp;这一步比较关键，Jenkins让我们选择使用何种方式来构建项目。这里我进行了一番斟酌，按照目前苍龙传统的构建方式，是使用javac来编译，然后使用zip来打包应该选择shell方式。但目前这种方式没有持续集成流程中“自动测试”这样一环，而自动测试对于持续集成来讲又非常重要。因此为了加入自动测试以及今后能更方便的构建苍龙服务器代码，最后决定，首先进行Maven的集成。

## Maven集成 {#maven}

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/5.png)

### Maven介绍 {#maven-desc}

&emsp;&emsp;Maven是一款专门对Java应用进行依赖管理的工具，它采用了统一的标准（Pom文件）来构建Java应用，使得Java开发者对于项目中使用的依赖组件能够一目了然，很便捷地处理依赖冲突的问题，也能很高效地完成编译、打包等操作。除此之外，Maven还能在打包之前完成自动化单元测试，非常有利于开发人员的自我测试，在功能开发阶段就找出并解决一些Bug。但非常不幸的是，苍龙服务器代码因为一些历史原因，并没有使用Maven。没有条件就创造条件，下面就开始了Maven集成工作。

### Maven集成步骤 {#maven-step}
1. 整理各服务器项目代码（游戏服、战斗服、世界服、日志服）所使用的jar包，区分出公有jar包与私有jar包。
2. 将公有jar包配置为直接从阿里云中央仓库下载，私有jar包上传到私有仓库，再从私有仓库下载。
3. 整理与解决依赖冲突问题

### 遇到的问题与解决 {#problem-resolve}
1.  大部分私有jar包来源未知，并且没有源码，也在开源平台中无法找到（应该是祖传jar包），无法通过编译源码方式deploy到私有仓库。解决方法：在Nexus Repository Manager后台登陆后直接上传jar包。
2.  某些jar包根据名字和内容判断，应该是公有jar包，但是却没有标注版本号（坑啊！），因此无法从阿里云中央仓库中下载。解决方法：作为私有jar包处理。
3.  原项目中jar包冲突却一直没被发现，例如activemq-all-5.10.2.jar与log4j-slf4j-impl-2.1.jar冲突，启动时会提示slf4j重复绑定。解决方法：将all包拆分为单个组件，并使用exclusion标签排除冲突的jar。
4.  原项目代码采用JDK1.6编译，但JDK1.6无法兼容最新版的Maven3.6.x。解决方法：使用最后能支持JDK1.6的Maven3.2.5版本。
5.  **（最棘手）**项目路径问题。普通Java项目源码路径为ROOT/src，但maven项目的源码路径为ROOT/src/main/java。如果要更改源码路径，那么以前在SVN上建立的分支将无法识别新的源码路径（已经过测试验证）。这个问题引起了我的好奇，并且发现在Git上修改源项目码路径不会有该问题，但是在SVN上就会出现。经过一番比较深入的研究后发现，在Git中如果将某个文件A移动了路径B，git会记录该版本进行了rename A->B，在其他分支上对A的修改仍然可以合并到B。而SVN则不一样，如果文件A移动到了路径B，SVN会记录该版本将A删除，新建了B，在其他分支上对A的修改就无法合并到B。因此，受苍龙使用SVN的限制，就无法修改源码路径，那么如何使Maven能够识别原有的源码路径ROOT/src呢？好在Maven提供了对源码和资源指定的支持：

```
<sourceDirectory>src</sourceDirectory>
<testSourceDirectory>test/java</testSourceDirectory>
<resources>
    <resource>
        <directory>resources</directory>
    </resource>
</resources>
<testResources>
    <testResource>
        <directory>test/resources</directory>
    </testResource>
</testResources>
```

&emsp;&emsp;至此，Maven集成完毕。

## 测试集成 {#test}

&emsp;&emsp;Maven已经有了，那么可以开始使用Jenkins进行持续集成了吗？不！不要忘了Maven还有一项重要使命：自动完成单元测试。苍龙服务器代码中虽然之前有引入JUnit包，但是需要手动运行，非常不方便（以前也从来没有手动运行过）。测试集成也比较简单，主要分为两类：

### 单元测试 {#unit-test}

*   静态公共方法(public static ...)：直接在测试类中调用即可。

*   私有方法(private ...)：这类方法无法在测试类中直接调用，这时候Java的反射调用就能排上用场了：

    ```java
    Method method = xxx.getClass().getDeclaredMethod("xxx", xxx.class,xxx.class);
    method.setAccessible(true);
    method.invoke(xxx, xxx, xxx);
    ```

*   Spring注入对象的成员方法（service等）：从Spring上下文中获取对象再调用即可。

### Mock测试 {#mock-test}

&emsp;&emsp;有时候编写测试代码时会出现这些情况，某个类可能过于复杂，可能因为依赖过多，甚至可能构造方法因为业务需要被设置成了私有（private）访问，导致我们无法直接new出这个对象，这时候怎么测试呢？这时候Mock测试就派上用场了。我选用的是目前使用最广泛的mockito库，步骤如下：

1.  模拟出该对象：`XXX mock = mock(XXX.class);`
2.  设置XXX.class的xx()方法返回值：`when(mock.xx()).thenReturn(xx);`

&emsp;&emsp;这样就可以在测试代码中，很方便的调用mock.xx()了，非常简单。

&emsp;&emsp;以后在每次Maven打包时，就会自动查找test路径下所有类中以@Test注解的所有public void xxx测试方法，如果测试结果与预期不符，就会终止打包，我们便可以在打包阶段就解决掉一些Bug，降低产品的风险。

## 流程图 {#jenkins-ci-cd}

&emsp;&emsp;类似于之前采用工具的持续集成方式，jenkins的集成流程也分为了程序版与策划版。

### 程序版流程图 {#programmer}

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/6.png)

### 策划版流程图 {#designer}

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/7.png)

### 优化 {#optimize}

&emsp;&emsp;可以看到，两条流水线的区别就是持续集成不同，程序是检出的代码，并且需要编译、测试、打包；而策划检出的是配置表。

&emsp;&emsp;两条流水线采用的是独立运行的方式，程序负责代码的持续集成，策划负责配置表的持续集成。使用了一段时间之后，我们发现，这种方式在开发版本（主线分支）没有问题，因为开发版本只有测试服没有正式服。而到了生产版本（线上分支），如果按照这种方式，程序、策划流水线独立运行，会导致在正式服维护期间，需要重启2次正式服。而正式服的重启又比较重度（需要备份线上数据），这就导致服务器更新时间翻倍（10分钟增加到20分钟）。为了解决上述问题，我对上述方案进行了改进，将两条流水线合并。

流程图如下：

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/8.png)

&emsp;&emsp;在该优化版本中，将之前的两条流水线相同的部分合并，不同的部分保留（SVN检出配置表、代码）。配置表和代码的产物生成之后，将两份产物合并作为新的产物，再生成版本号，用该版本产物来更新测试服与正式服。

# 服务器更新操作流程对比 {#compare}

* 原有更新流程（每一步均为手动操作）：

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/9.png)

* 现有更新流程（只有一步操作）：

![](https://gitee.com/juzii/res/raw/master/pic/2019/07/canglong_ci_develop/10.png)

# 细节对比 {#detail-compare}
以下分别为有/无持续集成、交付、部署下的对比结果：

*   正式服维护操作时间
    *   无：开发人员15分钟+运维人员15分钟。
    *   有：开发人员1秒（无需运维人员参与）。
*   正式服维护服务器更新时间
    *   无：约30分钟。
    *   有：约10分钟。
*   操作复杂度
    *   无：约20个步骤，非常复杂.
    *   有：只需要点一下鼠标，非常简单。
*   出错概率
    *   无：较高，手动执行的20个步骤，若有一个出错，则会出现更新错误（曾多次出现因代码未编译、未使用策划最新配置表、运维人员漏更新某个功能服等问题）。
    *   有：几乎为0，使用的是经过测试与验证的更新步骤与脚本。

# 总结 {#summary}

1.  持续集成与部署（CI/CD）应在项目初期就应该建立，能大大提高开发效率、提高产品质量，甚至能让产品走得更远。
2.  自定义工具式的持续集成在进行持续部署方面会较为困难，推荐还是使用Jenkins来实现持续集成、交付与部署。
3.  实践证明，自定义工具在测试性代码或配置表会比较有用，因为这种方式不用担心会把测试性代码或配置表更新到正式服。
4.  如今，DevOps的思想已在全球普及，它是重视软件开发（Dev）与运维技术（Ops）之间沟通合作一种文化，透过自动化“软件交付”和“架构变更”的流程，使得构建、测试、发布软件能够更加地快捷、频繁和可靠。



>   参考资料：[The Product Managers’ Guide to Continuous Delivery and DevOps](https://www.mindtheproduct.com/2016/02/what-the-hell-are-ci-cd-and-devops-a-cheatsheet-for-the-rest-of-us/)