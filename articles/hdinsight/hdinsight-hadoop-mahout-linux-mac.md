<properties
	pageTitle="Generate recommendations using Mahout and Hadoop | Microsoft Azure"
	description="Learn how to use the Apache Mahout machine learning library to generate movie recommendations with Linux-based HDInsight (Hadoop)."
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor=""/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/16/2015"
	ms.author="larryfr"/>

#Generate movie recommendations by using Apache Mahout with Linux-based Hadoop in HDInsight (preview)

[AZURE.INCLUDE [mahout-selector](../../includes/hdinsight-selector-mahout.md)]

Learn how to use the [Apache Mahout](http://mahout.apache.org) machine learning library with Azure HDInsight to generate movie recommendations.

Mahout is a [machine learning][ml] library for Apache Hadoop. Mahout contains algorithms for processing data, such as filtering, classification, and clustering. In this article, you will use a recommendation engine to generate movie recommendations that are based on movies your friends have seen.

> [AZURE.NOTE] The steps in this document require a Linux-based Hadoop on HDInsight cluster (preview). For information on using Mahout with a Windows-based cluster, see [Generate movie recommendations by using Apache Mahout with Windows-based Hadoop in HDInsight](hdinsight-mahout.md)

##Prerequisites

* A Linux-based Hadoop on HDInsight cluster. For information about creating one, see [Get started using Linux-based Hadoop in HDInsight][getstarted]

##<a name="recommendations"></a>Understanding recommendations

One of the functions that is provided by Mahout is a recommendation engine. This engine accepts data in the format of `userID`, `itemId`, and `prefValue` (the users preference for the item). Mahout can then perform co-occurance analysis to determine: _users who have a preference for an item also have a preference for these other items_. Mahout then determines users with like-item preferences, which can be used to make recommendations.

The following is an extremely simple example that uses movies:

* __Co-occurance__: Joe, Alice, and Bob all liked _Star Wars_, _The Empire Strikes Back_, and _Return of the Jedi_. Mahout determines that users who like any one of these movies also like the other two.

* __Co-occurance__: Bob and Alice also liked _The Phantom Menace_, _Attack of the Clones_, and _Revenge of the Sith_. Mahout determines that users who liked the previous three movies also like these three.

* __Similarity recommendation__: Because Joe liked the first three movies, Mahout looks at movies that others with similar preferences liked, but Joe has not watched (liked/rated). In this case, Mahout recommends _The Phantom Menace_, _Attack of the Clones_, and _Revenge of the Sith_.

##Load the data

Conveniently, [GroupLens Research][movielens] provides rating data for movies in a format that is compatible with Mahout. Use the following steps to download the data, then load it into the default storage for the cluster:

1. Use SSH to connect to the Linux-based HDInsight cluster. The address to use when connecting is `CLUSTERNAME-ssh.azurehdinsight.net` and the port is `22`.

	For more information on using SSH to connect to HDInsight, see the following documents:

    * **Linux, Unix or OS X clients**: See [Connect to a Linux-based HDInsight cluster from Linux, OS X or Unix](hdinsight-hadoop-linux-use-ssh-unix.md#connect-to-a-linux-based-hdinsight-cluster)

    * **Windows clients**: See [Connect to a Linux-based HDInsight cluster from Windows](hdinsight-hadoop-linux-use-ssh-windows.md#connect-to-a-linux-based-hdinsight-cluster)

2. Download the MovieLens 100k archive, which contains 100,000 ratings from 1000 users for 1700 movies.

        curl -O http://files.grouplens.org/datasets/movielens/ml-100k.zip

3. Extract the archive using the following command:

        unzip ml-100k.zip

    This will extract the contents to a new folder named **ml-100k**.

4. Use the following commands to copy the data to HDInsight storage:

        cd ml-100k
        hadoop fs -copyFromLocal ml-100k/u.data /example/data/u.data

    The data contained in this file has a structure of `userID`, `movieID`, `userRating`, and `timestamp`, which tells us how highly each user rated a movie. Here is an example of the data:


		196	242	3	881250949
		186	302	3	891717742
		22	377	1	878887116
		244	51	2	880606923
		166	346	1	886397596

##Run the job

Use the following command to run the recommendation job:

	mahout recommenditembased -s SIMILARITY_COOCCURRENCE --input /example/data/u.data --output /example/data/mahoutout  --tempDir /temp/mahouttemp

> [AZURE.NOTE] The job may take several minutes to complete, and may run multiple MapReduce jobs.

##View the output

1. Once the job completes, use the following command to view the generated output:

		hadoop fs -text /example/data/mahoutout/part-r-00000

	The output will appear as follows:

		1	[234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
		2	[282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
		3	[284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
		4	[690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]

	The first column is the `userID`. The values contained in '[' and ']' are `movieId`:`recommendationScore`.

2. Some of the other data contained in the **ml-100k** directory can be used to make the data more user friendly. First, download the data using the following command:

		hadoop fs -copyToLocal /example/data/mahoutout/part-r-00000 recommendations.txt

	This will copy the output data to a file named **recommendations.txt** in the current directory.

3. Use the following command to create a new Python script that will look up movie names for the data in the recommendations output:

		nano show_recommendations.py

	When the editor opens, use the following as the contents of the file:

		#!/usr/bin/env python

		import sys

		if len(sys.argv) != 5:
		        print "Arguments: userId userDataFilename movieFilename recommendationFilename"
		        sys.exit(1)

		userId, userDataFilename, movieFilename, recommendationFilename = sys.argv[1:]

		print "Reading Movies Descriptions"
		movieFile = open(movieFilename)
		movieById = {}
		for line in movieFile:
		        tokens = line.split("|")
		        movieById[tokens[0]] = tokens[1:]
		movieFile.close()

		print "Reading Rated Movies"
		userDataFile = open(userDataFilename)
		ratedMovieIds = []
		for line in userDataFile:
		        tokens = line.split("\t")
		        if tokens[0] == userId:
		                ratedMovieIds.append((tokens[1],tokens[2]))
		userDataFile.close()

		print "Reading Recommendations"
		recommendationFile = open(recommendationFilename)
		recommendations = []
		for line in recommendationFile:
		        tokens = line.split("\t")
		        if tokens[0] == userId:
		                movieIdAndScores = tokens[1].strip("[]\n").split(",")
		                recommendations = [ movieIdAndScore.split(":") for movieIdAndScore in movieIdAndScores ]
		                break
		recommendationFile.close()

		print "Rated Movies"
		print "------------------------"
		for movieId, rating in ratedMovieIds:
		        print "%s, rating=%s" % (movieById[movieId][0], rating)
		print "------------------------"

		print "Recommended Movies"
		print "------------------------"
		for movieId, score in recommendations:
		        print "%s, score=%s" % (movieById[movieId][0], score)
		print "------------------------"

	Press **Ctrl-X**, **Y**, and finally **Enter** to save the data.

3. Use the following command to make the file executable:

		chmod +x show_recommendations.py

4. Run the Python script:

		./show_recommendations.py 4 ml-100k/u.data ml-100k/u.item recommendations.txt

	This will look at the recommendations generated for user ID 4.

	* The **u.data** file is used to retrieve movies that the user has rated
	* The **u.item** file is used to retrieve the names of the movies
	* The **recommendations.txt** is used to retrieve the movie recommendations for this user

	The output from this command will be similar to the following:

		Reading Movies Descriptions
		Reading Rated Movies
		Reading Recommendations
		Rated Movies
		------------------------
		Mimic (1997), rating=3
		Ulee's Gold (1997), rating=5
		Incognito (1997), rating=5
		One Flew Over the Cuckoo's Nest (1975), rating=4
		Event Horizon (1997), rating=4
		Client, The (1994), rating=3
		Liar Liar (1997), rating=5
		Scream (1996), rating=4
		Star Wars (1977), rating=5
		Wedding Singer, The (1998), rating=5
		Starship Troopers (1997), rating=4
		Air Force One (1997), rating=5
		Conspiracy Theory (1997), rating=3
		Contact (1997), rating=5
		Indiana Jones and the Last Crusade (1989), rating=3
		Desperate Measures (1998), rating=5
		Seven (Se7en) (1995), rating=4
		Cop Land (1997), rating=5
		Lost Highway (1997), rating=5
		Assignment, The (1997), rating=5
		Blues Brothers 2000 (1998), rating=5
		Spawn (1997), rating=2
		Wonderland (1997), rating=5
		In & Out (1997), rating=5
		------------------------
		Recommended Movies
		------------------------
		Seven Years in Tibet (1997), score=5.0
		Indiana Jones and the Last Crusade (1989), score=5.0
		Jaws (1975), score=5.0
		Sense and Sensibility (1995), score=5.0
		Independence Day (ID4) (1996), score=5.0
		My Best Friend's Wedding (1997), score=5.0
		Jerry Maguire (1996), score=5.0
		Scream 2 (1997), score=5.0
		Time to Kill, A (1996), score=5.0
		Rock, The (1996), score=5.0
		------------------------

##Delete temporary data

Mahout jobs do not remove temporary data that is created while processing the job. The `--tempDir` parameter is specified in the example job to isolate the temporary files into a specific path for easy deletion. To remove the temp files, use the following command:

	hadoop fs -rm -f -r wasb:///temp/mahouttemp

> [AZURE.WARNING] If you do not remove the temporary files or the output file, you will receive a 'cannot overwrite file' error message if you run the job again with the same `--tempDir` path.

##Next steps

Now that you have learned how to use Mahout, discover other ways of working with data on HDInsight:

* [Hive with HDInsight](../hadoop-use-hive.md)
* [Pig with HDInsight](../hadoop-use-pig.md)
* [MapReduce with HDInsight](../hadoop-use-mapreduce.md)

[build]: http://mahout.apache.org/developers/buildingmahout.html
[movielens]: http://grouplens.org/datasets/movielens/
[100k]: http://files.grouplens.org/datasets/movielens/ml-100k.zip
[getstarted]: hdinsight-hadoop-linux-tutorial-get-started.md
[upload]: hdinsight-upload-data.md
[ml]: http://en.wikipedia.org/wiki/Machine_learning
[forest]: http://en.wikipedia.org/wiki/Random_forest
[management]: https://manage.windowsazure.com/
[enableremote]: ./media/hdinsight-mahout/enableremote.png
[connect]: ./media/hdinsight-mahout/connect.png
[hadoopcli]: ./media/hdinsight-mahout/hadoopcli.png
[tools]: https://github.com/Blackmist/hdinsight-tools
 