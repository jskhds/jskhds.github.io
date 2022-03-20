---
title: java读取文件的六种方式
date: 2021-12-03 15:58:57
tags: java读取文件
---

我们 java 老师很喜欢让我们自学，说程序在课堂上是学不会的，所以他要多布置作业督促我们。 这次作业是分析他之前项目的一个天体测量文件，所以我顺便记录一下在 java 里读取文件的几种方式

<!-- more -->

##### 1.Scanner + new File

```java
package readFile_learning;
import java.io.FileReader;
import java.io.IOException;
import java.util.Scanner;
//Scanner + new File 既可以按行读取 也可以按分隔符获取
public class methods_1 {
	public static void main(String[] args) throws IOException{
		// TODO Auto-generated method stub
//		String fileName = "D:\\javaTest\\N1601335746_1_gaia1.QMPF";
		String fileName = "D:\\javaTest\\test1.txt";
//		按行读取
		try(Scanner sc = new Scanner(new FileReader(fileName))){
			while(sc.hasNextLine()) {
				String line = sc.nextLine();
				System.out.println(line);
			}
		}
//		按分隔符读取
		try(Scanner sc = new Scanner(new FileReader(fileName))){
			sc.useDelimiter("\\!!!");
			while(sc.hasNext()) {
				String str = sc.next();
				System.out.println(str);
			}
		}
		catch (Exception e) {
	        System.out.println("文件读取错误!");
	    }		
	}
}

```

##### 2.Stream流

```java
package readFile_learning;

import java.io.IOException;
import java.nio.file.Files;

import java.nio.file.Paths;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class methods_2 {
//按行读取 推荐
	public static void main(String[] args) throws IOException{
		// TODO Auto-generated method stub
		String fileName = "D:\\javaTest\\N1601335746_1_gaia1.QMPF";
//		读取文件内容到Stream流中，按行读取,不用一次性把文件加载到内存中 每一行都存在Stream流中
		@SuppressWarnings("resource")
		Stream<String> lines = Files.lines(Paths.get(fileName));
//		随机按行处理 就去掉Ordered
//		lines.forEachOrdered(ele ->{;
//		System.out.println(ele);
//		});
//		按文件顺序处理
//		lines.forEachOrdered(System.out::println); 不知道为什么跑出不来 代码是对的
//		其它要求
//		并行流 利用cpu加快行处理速度
		lines.parallel().forEachOrdered(ele ->{;
		System.out.println(ele);
		});
//		转换为 List对象 不适合文件大的情况
		List<String> collect = lines.collect(Collectors.toList());
		
	}

}

```

##### 3.List String 

```java
package readFile_learning;

 
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;

public class methods_3 {

	public static void main(String[] args) throws IOException {
		// TODO Auto-generated method stub
//直接转换为List String 一次性把文件加到lines里 适合文件不是很大 需要对文件按行处理时
		String fileName = "D:\\javaTest\\N1601335746_1_gaia1.QMPF";
		List<String> lines = Files.readAllLines(Paths.get(fileName));
		lines.forEach(System.out::println);
	}

}

```

4. ##### Files.readString(Paths.get(fileName))

```java
package readFile_learning;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;

public class methods_4 {

	public static void main(String[] args) throws IOException {
		// TODO Auto-generated method stub
//		java 11 支持的方式 读取文件不超过2G 不是按行读取 一次性读取出来
		String fileName = "D:\\javaTest\\N1601335746_1_gaia1.QMPF";
		String s = Files.readString(Paths.get(fileName));
		System.out.println(s);
	}

}


```

##### 5.非JDK11 一次性读取文件的办法

```java
package readFile_learning;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;

public class methods_5 {

	public static void main(String[] args) throws IOException {
		// TODO Auto-generated method stub
		String fileName = "D:\\javaTest\\N1601335746_1_gaia1.QMPF";
//		不是JDK11 但是要一次性读取全部文件
		byte[] bytes = Files.readAllBytes(Paths.get(fileName));
		String content = new String(bytes,StandardCharsets.UTF_8);
		System.out.println(content);
	}

}

```

##### 6.管道流

```java
package readFile_learning;

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class methods_6 {
//管道流 适合读取文件内存比较大的
	public static void main(String[] args) throws FileNotFoundException, IOException {
		// TODO Auto-generated method stub
		String fileName = "D:\\javaTest\\N1601335746_1_gaia1.QMPF";
		try(BufferedReader br = new BufferedReader(new FileReader(fileName))){
			String line;
			while((line = br.readLine())!=null) {
				System.out.println(line);
			}
		}
	}

}

```

之前我们还写 JavaMail 来着，一开始我看到都两眼一蒙不知道是什么，后来自己去搜索自学，发现还是挺简单的。感谢互联网
