<properties
   pageTitle="Use Hadoop Pig in HDInsight | Azure"
   description="Learn how to use Pig with Hadoop on HDInsight through Remote Desktop."
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang=""
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="02/18/2015"
   ms.author="larryfr"/>

#Run Pig jobs using the Pig command (Remote Desktop)

[AZURE.INCLUDE [pig-selector](../includes/hdinsight-selector-use-pig.md)]

This document provides a walkthrough of using the Pig command to run Pig Latin statements interactively, or as a batch job, on a Linux-based Hadoop on HDInsight cluster. Pig Latin allows you to create MapReduce applications by describing data transformations, rather than map and reduce functions.

##<a id="prereq"></a>Prerequisites

To complete the steps in this article, you will need the following.

* A Windows-based HDInsight (Hadoop on HDInsight) cluster

* A Windows 7, 8 or 10 client

##<a id="connect"></a>Connect with remote desktop

Enable Remote Desktop for the HDInsight cluster, then connect to it by following the instructions at <a href="http://azure.microsoft.com/documentation/articles/hdinsight-administer-use-management-portal/#rdp" target="_blank">Connect to HDInsight clusters using RDP</a>.

##<a id="pig"></a>Use the Pig command

2. Once connected, start the **Hadoop Command Line** using the icon on the desktop.

2. Use the following to start the Pig command-line.

		%pig_home%\bin\pig

	You will be presented with a `grunt>` prompt. 

3. Enter the following statement.

		LOGS = LOAD 'wasb:///example/data/sample.log';

	This command loads the contents of the sample.log file into LOGS. You can view the contents of the file by using the following.

		DUMP LOGS;

4. Next, transform the data by applying a regular expression to extract only the logging level from each record using the following.

		LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;

	You can use **DUMP** to view the data after the transformation. In this case, `DUMP LEVELS;`.

5. Continue applying transformations by using the following statements. Use `DUMP` to view the result of the transformation after each step.

	<table>
	<tr>
	<th>Statement</th><th>What it does</th>
	</tr>
	<tr>
	<td>FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;</td><td>Removes rows that contain a null value for the log level and stores the results into FILTEREDLEVELS.</td>
	</tr>
	<tr>
	<td>GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;</td><td>Groups the rows by log level and stores the results into GROUPEDLEVELS.</td>
	</tr>
	<tr>
	<td>FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;</td><td>Creates a new set of data that contains each unique log level value and how many times it occurs. This is stored into FREQUENCIES</td>
	</tr>
	<tr>
	<td>RESULT = order FREQUENCIES by COUNT desc;</td><td>Orders the log levels by count (descending,) and stores into RESULT</td>
	</tr>
	</table>

6. You can also save the results of a transformation using the `STORE` statement. For example, the following saves the `RESULT` to the **/example/data/pigout** directory on the default storage container for your cluster.

		STORE RESULT into 'wasb:///example/data/pigout'

	> [AZURE.NOTE] The data is stored in the specified directory in files named **part-nnnnn**. If the directory already exists, you will receive an error.

7. To exit the grunt prompt, enter the following statement.

		QUIT;

###Pig Latin batch files

You can also use the Pig command to run Pig Latin contained in a file.

3. After exiting the grunt prompt, open **Notepad** and create a new file named **pigbatch.pig** in the **%PIG_HOME%** directory.

4. Type or paste the following lines into the **pigbatch.pig** file, then save when done.

		LOGS = LOAD 'wasb:///example/data/sample.log';
		LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
		FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
		GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
		FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
		RESULT = order FREQUENCIES by COUNT desc;
		DUMP RESULT;

5. Use the following to run the **pigbatch.pig** file using the pig command.

		pig %PIG_HOME%\pigbatch.pig

	Once the batch job completes, you should see the following output, which should be the same as when you used `DUMP RESULT;` in the previous steps.

		(TRACE,816)
		(DEBUG,434)
		(INFO,96)
		(WARN,11)
		(ERROR,6)
		(FATAL,2)

##<a id="summary"></a>Summary

As you can see, the Pig command allows you to interactively run MapReduce operations using Pig Latin, as well as run statements stored in a batch file.

##<a id="nextsteps"></a>Next steps

For general information on Pig in HDInsight.

* [Use Pig with Hadoop on HDInsight](hdinsight-use-pig.md)

For information on other ways you can work with Hadoop on HDInsight.

* [Use Hive with Hadoop on HDInsight](hdinsight-use-hive.md)

* [Use MapReduce with Hadoop on HDInsight](hdinsight-use-mapreduce.md)