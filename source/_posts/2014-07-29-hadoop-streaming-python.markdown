---
layout: post
title: "Hadoop Streaming编程(Python)"
date: 2014-07-29 17:17:59 +0800
comments: true
categories: Hadoop
keywords: Hadoop Streaming编程(Python)
description: Hadoop Streaming编程(Python)
---
Hadoop Streaming是Hadoop提供的一个工具，可以允许开发者以非Java的编程语言开发Mapper和Reducer，而且在开发过程中，我们只需要专注于数据的输入和输出处理，其它的事情都交给这个工具来做。本例是用Python实现一个简单的Wordcount例子，环境为：

- Hadoop:CDH4.3.2
- Python:2.6.6
- OS:Centos 6.5
- Hadoop Streaming:hadoop-2.0.0-cdh4.3.2/share/hadoop/tools/lib/hadoop-streaming-2.0.0-cdh4.3.2.jar

<!--more-->
### 代码实现 ###
Python实现wordcount的Mapper和Reducer，比较简单直接看代码，*mapper.py*：

	#!/usr/bin/python
	
	import sys
	for line in sys.stdin:
	    line = line.strip()
	    words = line.split()
	    for word in words:
	        print '%s\t%s' % (word,1)

*reducer.py*:

	#!/usr/bin/python
	
	from operator import itemgetter
	import sys
	
	current_word = None
	current_count = 0
	word = None
	
	# input comes from STDIN
	for line in sys.stdin:
	    # remove leading and trailing whitespace
	    line = line.strip()
	
	    # parse the input we got from mapper.py
	    word, count = line.split('\t', 1)
	
	    # convert count (currently a string) to int
	    try:
	        count = int(count)
	    except ValueError:
	        # count was not a number, so silently
	        # ignore/discard this line
	        continue
	
	    # this IF-switch only works because Hadoop sorts map output
	    # by key (here: word) before it is passed to the reducer
	    if current_word == word:
	        current_count += count
	    else:
	        if current_word:
	            # write result to STDOUT
	            print '%s\t%s' % (current_word, current_count)
	        current_count = count
	        current_word = word
	
	# do not forget to output the last word if needed!
	if current_word == word:
	    print '%s\t%s' % (current_word, current_count)

这里需要注意第一行是：*#!/usr/bin/python*，而不是其它例子中的*#!/usr/bin/env python*

### 数据准备 ###
下载下面链接的数据，拷贝到HDFS中

- [The Outline of Science, Vol. 1 (of 4) by J. Arthur Thomson](http://www.gutenberg.org/ebooks/20417)
- [The Notebooks of Leonardo Da Vinci](http://www.gutenberg.org/ebooks/5000)
- [Ulysses by James Joyce](http://www.gutenberg.org/ebooks/4300)

### 任务提交 ###
这块试了很多次，很多命令参数MR1和MR2不同的，比如必须这么写才可以：*-mapper "python mapper.py"*，下面是完整的执行脚本，经过测试没有问题：

    hadoop jar /home/hadoop/hadoop-2.0.0-cdh4.3.2/share/hadoop/tools/lib/hadoop-streaming-2.0.0-cdh4.3.2.jar -input /test/data/* -output /test/output  -file ./*.py -mapper "python mapper.py" -reducer  "python reducer.py" -numReduceTasks 2

完整代码参考这里项目：https://github.com/findhy/python-example/tree/master/hadoop/wordcount

### 参考 ###
http://www.michael-noll.com/tutorials/writing-an-hadoop-mapreduce-program-in-python/

