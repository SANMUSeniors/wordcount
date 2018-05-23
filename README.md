# wordcount
Hadoop: Intellij结合Maven本地运行和调试MapReduce程序 (无需搭载Hadoop和HDFS环境)
2018年05月20日 10:33:18
阅读数：11 编辑
楼主花费了1天终于按照教程自己第一次成功运行了这个案例。
Hadoop: Intellij结合Maven本地运行和调试MapReduce程序 (无需搭载Hadoop和HDFS环境)
 2016-05-24 
 12717 
 Hadoop 
 maven 
 39
Hadoop: Run and Debug Hadoop MapReduce Applications with Intellij IDEA and Maven locally, without Hadoop and HDFS environment.

前言
Hadoop可以运行在三种模式下：

单机模式
伪分布模式
完全分布式模式
相信初学者入门Hadoop的第一堂课就是伪分布模式Hadoop系统的安装，相信一定是血泪史各种翻教程各种重装。而实际上，基于Hadoop的MapReduce程序在单机上运行，并不一定需要安装伪分布模式Hadoop系统，甚至，并不一定需要安装Hadoop。

运行和调试MapReduce程序只需要有相应的Hadoop依赖包就行，可以完全当成一个普通的JAVA程序。本文就将介绍这种简单方便的方法。

简介
正如上文所说，在单机模式下，可以将MapReduce程序当成一个普通的JAVA程序；对比伪分布模式，其主要不足就在于没有Hadoop的整个管理控制系统，如JobTracker面板，而只是用来运行和调试程序；而其优点就在于开发调试方便，编写的程序通常不需要修改即可在真实的分布式Hadoop集群下运行。

Maven
Maven是一个项目管理工具，我们这里主要用到的是它的依赖管理系统。通常我们在开发Hadoop MapReduce程序时，首先要下载对应版本的镜像，然后加载镜像中的JAR依赖包，开始编写代码。这个步骤说起来容易但经常会碰到错综复杂的依赖关系，而利用Maven就能轻松解决这个问题。只需要在Maven配置文件中指定Hadoop依赖包名字和版本号，Maven就能自动搞定这些依赖，你只需要专心写代码就好了。

Intellij IDEA
为什么不用Eclipse呢？这里没有贬低Eclipse的意思，只是我认为Eclipse用户体验太差，而且各种操作过于繁琐。Intellij用起来更顺手，内置了Maven的支持，而且看起来似乎更有前景，就使用Intellij了。

环境要求
JDK 1.7（1.8似乎也可以，但Hadoop官方推荐1.7）
Intellij
不需要安装任何模式的Hadoop。

WordCount
这里以Hadoop的官方示例程序WordCount为例，演示如何一步步编写程序直到运行。

新建项目
在Intellij中点击File->New->Project，在弹出的对话框中选择Maven，JDK选择1.7，点击Next。

Screenshot New Maven Project

接下来填写Maven的GroupId和ArtifactId，随便填，点击Next。

Screenshot Project Info

然后是Project name，这里填写WordCount，点击Finish。

Screenshot New Maven Project

这样就新建好了一个空的项目，别着急，还有一个地方可能需要修改。打开Intellij的Preference偏好设置，定位到Build, Execution, Deployment->Compiler->Java Compiler，将WordCount的Target bytecode version修改为1.7。

Screenshot Target Bytecode Version

配置依赖
新建项目后，在Intellij左上方会有项目文件结构，双击以编辑pom.xml，这就是Maven的配置了。

添加源
pom.xml初始内容如下

XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.polarxiong</groupId>
    <artifactId>hadoop</artifactId>
    <version>1.0-SNAPSHOT</version>

</project>
在project内尾部添加

XML
<repositories>
    <repository>
        <id>apache</id>
        <url>http://maven.apache.org</url>
    </repository>
</repositories>
添加apache源。

添加依赖
这里只需要用到基础依赖hadoop-core和hadoop-common；如果需要读写HDFS，则还需要依赖hadoop-hdfs和hadoop-client；如果需要读写HBase，则还需要依赖hbase-client。

在project内尾部添加

XML
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-core</artifactId>
        <version>1.2.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.2</version>
    </dependency>
</dependencies>
这里hadoop-core的version一般为1.2.1，hadoop-common的version可以依照你的实际需要来。

修改pom.xml完成后，Intellij右上角会提示Maven projects need to be Imported，点击Import Changes以更新依赖

附上完整的pom.xml

XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.polarxiong</groupId>
    <artifactId>hadoop</artifactId>
    <version>1.0-SNAPSHOT</version>

    <repositories>
        <repository>
            <id>apache</id>
            <url>http://maven.apache.org</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-core</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.2</version>
        </dependency>
    </dependencies>
</project>
WordCount
在src->main->java下新建一个WordCount类，添加内容

Java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

    public static class TokenizerMapper
            extends Mapper<Object, Text, Text, IntWritable> {

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context
        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer
            extends Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values,
                           Context context
        ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
此代码来自Hadoop官方教程，出处见参考。

配置输入文件
WordCount对输入文件字符进行计数，输出计数的结果。首先需要配置输入路径，这里在WordCount下（src同级目录）新建一个文件夹input，并添加一个或多个文本文件到input中，作为示例。

File List Input

这里还有一件事情，点击File->Project Structure，在弹出来的对话框中选择Modules项，点击Sources选项卡，将Language level调整为7。（如果你用到版本控制的话，可以在这里将input文件夹标记为Excluded。

Screenshot Modules Settings

配置运行参数
这里我们需要配置此程序运行时的Main class，以及WordCount需要的输入输出路径。

在Intellij菜单栏中选择Run->Edit Configurations，在弹出来的对话框中点击+，新建一个Application配置。配置Main class为WordCount（可以点击右边的...选择），Program arguments为input/ output/，即输入路径为刚才创建的input文件夹，输出为output。

Screenshot Edit Configurations

运行和调试
运行
上述配置完成后，点击菜单栏Run->Run 'WordCount'即开始运行此MapReduce程序，Intellij下方会显示Hadoop的运行输出。待程序运行完毕后，Intellij左上方会出现新的文件夹output，其中的part-r-00000就是运行的结果了！

Screenshot Run

由于Hadoop的设定，下次运行时务必删除output文件夹！

调试
断点调试也很容易，在需要设置的代码前单击加上断点，点击菜单栏Run->Debug 'WordCount'即开始调试，程序会在断点处停下。

Screenshot Debug

Windows下的权限问题
Windows下运行可能会出错，提示

ERROR security.UserGroupInformation: PriviledgedActionException as ...
这是因为当前用户没有权限来设置路径权限（Linux无此问题），一个解决方法是给hadoop打补丁，参考Failed to set permissions of path: tmp，因为这里使用的Maven，此方法不太适合。另一个方法是将当前用户设置为超级管理员（“计算机管理”，“本地用户和组”中设置），或以超级管理员登录运行此程序。

不过我觉得最好的解决方法是在Linux或macOS上跑hadoop。

上面是原来的楼主写的解决办法，我看有一个大神是这样解决的，完美解决了这个问题。

win用户看这里，关于文中提到的Failed to set permissions of path: tmp问题，有简单解决办法。
换hadoop-core的版本换成0.20.2版本。直接改pom文件中<version>1.2.1</version>这行改成<version>0.20.2</version>，相应的main方法里面的 Job job = Job.getInstance(conf, "word count");也要改成Job job = new Job(conf, "word count");就可以了。
另外一种复杂点的办法就是，自己下载1.2.1源码包，把其中的报错那行给注掉，然后打包，再使用。当然网上有人已经改好了，并且提供打包下载，不过改的是1.0.2版本，下载地址参考http://blog.csdn.net/shiqidide/article/details/7585048,要自己安装jar包到maven路径下，安装方法mvn install:install-file -Dfile=(注意这里改成自己的路径)hadoop-core-1.0.2-modified.jar -DgroupId=org.apache.hadoop -DartifactId=hadoop-core -Dversion=1.0.2 -Dpackaging=jar，然后再参考简单步骤更改版本，及main方法。

运行后可能出现这样的问题及解决办法

Windows下用Eclipse开发Hadoop程序遇到的问题及解决方法
1. 运行hadoop程序报错如下：

Exception in thread "main" java.io.IOException: Cannot run program "chmod": CreateProcess error=2

解决方法： 只需要把cygwin的bin目录加到windows的用户环境变量中就可以了，然后需要重启eclipse

小结
上面描述的步骤有些多，但逻辑上都是很清晰的，有过一次经验以后就之后就容易多了，主要就是在pom.xml的配置和运行参数的配置上。删除WordCount程序的运行不需要任何的Hadoop开发环境，并且依赖问题全部交给Maven解决了，怎么样？是不是非常简单？

NICE!
