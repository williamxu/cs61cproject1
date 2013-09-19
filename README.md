cs61cproject1
=============

CS61C Fall 2013 Project 1 - Co-occurrence

TA: Jeffrey Dong and Kevin Yeun

Part 1 due Sunday, September 22nd @ 23:59:59 PST
Announcements

Updates and clarifications will be posted here.

    9/15/13 - Project 1-1 posted.
    9/18/13 - Updated Makefile to run DoublePair.java.

Setup

Make sure you are using a machine from 330 Soda or 273 Soda when doing this project.

Copy the proj1 files into your local proj1 directory. We recommend you create a separate repository for this project. If you would like to have the files in a directory called proj1, you can do:

$ mkdir ~/proj1
$ cd ~/proj1
$ git pull ~cs61c/proj/01 master

Note: This is an individual project. Do not show any code to other students, even for debugging.
Overview

For this project, you're finally going to roll up your sleeves and use MapReduce to answer a big data problem. The question you're going to answer is this: given a word, what other words are statistically associated with it? If I say 'love', or 'death', or 'terrorism', what other words and concepts go with it?

A reasonable statistical definition is the co-occurrence, which measures how often two words appear together in documents. Given a target word, we can figure out which words in a body of text (called the corpus) are most closely related to it by ranking each unique word in the corpus by its co-occurrence rate with the target word. To calculate:

    Let Aw be the number of occurrences of w in the corpus.
    Let Cw be the number of occurrences of w in documents that also have the target word.

   Co-occurrence rate :=  if(Cw > 0)   Cw * (log(Cw))3  / Aw 
                                    else  0

Note that 1) we will use the natural logarithm when calculating co-occurrence and 2) we will not be calculating co-occurrence.

Here is an example that illustrates this definition. Let the target word be "Dave":

Doc_ID#1: Randy, Krste, Dave
Doc_ID#2: Randy, Dave, Randy
Doc_ID#3: Randy, Krste, Randy


Occurrences: ARandy = 5; AKrste = 2;
Co-occurrences: CRandy = 3; CKrste = 1;
Co-occurrence Rate:

    with Randy: CRandy * (log(CRandy))3 / ARandy = 3/5 * (log(3))3 = 0.7956
    with Krste: CKrste * (log(CKrste))3 / AKrste = 1/2 * (log(1))3 = 0

This does nothing to account for the distance between words however. A fairly straightforward generalization of the problem is to, instead of giving each co-occurence a value of 1, give it a value f(d), where d is the minimum distance from the word occurrence to the nearest instance of the target word measured in words. Suppose our target word is cat. The values of d for each word is given below:

Document:     The fat cat did not like the skinny cat.
Value of d:    2   1       1   2    3   2     1


If the target word does not exist in the document, the value of d should be Double.POSITIVE_INFINITY.

The function f() takes d as input and outputs another number. We will define f() to send infinity to 0 and positive numbers to numbers greater than or equal to one. The result of the generalization is as follows:

    Let Aw be the number of occurrences of w in the corpus.
    Let Sw be the the sum of f(dw) over all occurrences of w in the corpus.

   Co-occurrence rate :=  if(Sw > 0)   Sw * (log(Sw))3  / Aw 
                                    else  0

Your task is to produce an ordered list of words for the target word sorted by generalized co-occurrence rate, ordered with the biggest co-occurrence rates at the top. The data will be the same data we used in labs 2 and 3. This isn't the most sophisticated text-analysis algorithm out there, but it's enough to illustrate what you can do with MapReduce.
Part 1 Instructions

In the proj1 directory that you pulled, you will find skeleton code for two MapReduce jobs and a Makefile in a file called Proj1.java and also a file called DoublePair.java, which is a custom Writable class. That directory will also include some test data which will contain some of the results from our reference implementation. You should get the same output as we did, approximate to some floating point roundoff.

For part 1, you will need to :

    Implement the DoublePair class in DoublePair.java.
    Implement the MapReduce jobs in Proj1.java to calculate co-occurrence.

The DoublePair Class

The DoublePair class is a custom Writable class that holds two double variables. Implement all of the functions with the comment //YOUR CODE HERE. You are not required to use DoublePair in your implementation of Proj1.java, but you are required to implement it, and we will test your DoublePair for correctness. Note that if you choose not to use DoublePair, you still should be using some form of custom keys or values. If you would like to use DoublePair as a Hadoop key, you should implement the WritableComparable interface by defining the methods compareTo() and hashCode() (refer to the spec here).

Feel free to write a main method in DoublePair and define tests there. Whether you have a main method or not won't affect any other parts of your project. For a review on custom Hadoop keys/values, please refer to lab 3. If you have written your own main method and would like to test, you can run DoublePair with the following command:

$ make doublepair

The MapReduce Jobs

Your solution should implement the generalized co-occurrence algorithm above. Make sure to convert every word you're reading to lowercase first. We will be using three different f(d) functions, the first of which ignores the d parameter and outputs either 0 or 1. Using the first function is the same as using the basic co-occurrence algorithm we described above. To get the value of f(d), you should call the f() method in the Func class. You can select which function is being used by setting the value of the funcNum parameter to either 0, 1, or 2 in conf.xml. While you may define additional f()'s, please do not modify any of the existing ones.

There is also a combiner called Combine1 that can be run between the first Map and first Reduce job. Although implementing this is not required for getting the correct output, implementing this will be worth roughly 10% of your grade. Your combiner must be non-trivial (ie. it must not be the identity function), and you must make sure that your output is the same regardless of whether your combiner is run or not.

You will need to modify some of the type signatures in Proj1.java. Please refer to what you did in lab 2 if you are having trouble. The framework expects you to output (from the second job) a DoubleWritable and Text pair. These should be the score for each word and the word itself, respectively. At the end of your MapReduce jobs, you should output only the first 100 or so results.

Here is a list of potentially helpful info:

    Hadoop guarantees that a reduce tasks receives its keys in sorted order. Note that your co-occurrence values also need to be in sorted order.

    If you do not override a particular map() and/or reduce() function, the mapper and/or reducer for that stage will be the identity function.

    You are free to (and should!) use Java's built-in data structures. In particular, it may be wise to use something other than just Lists.

    The String and Math classes in Java may come in handy.

Running Things Locally

You can launch a job via:

$ hadoop jar proj1.jar Proj1 -conf conf.xml <params> <input> <intermediateDir> <outDir>


You should edit conf.xml to refer to the target word and f for the analysis the three f's we'll be using are numbered 0, 1, and 2). The <input> parameter specifies which input file we are using. The reference solutions were generated from ~cs61c/data/billOfRights.txt.seq.

The params <intermediateDir> and <outDir> specify the intermediate and output directories. These can be whatever you like. The intermediate directory will hold the output from the first MapReduce job, which is used as input for the second job. This may be helpful in debugging.

The <params> field supports two parameters, -Dcombiner and -DrunJob2 Specifying -Dcombiner=true will turn on the combiner, while specifying -Dcombiner=false will turn it off. Specifiying -DrunJob2=false will cause only the first job to run, and will cause its output to be in a friendly (Text) format instead of as sequence files. This is intended for debugging. The default value for -Dcombiner is false and -DrunJob2 is true.

We'll also give you a program (Importer.java) that you can use to convert text files to the appropriate sequence file format. This will let you create your own test data, although it is not required.

Make sure you delete the intermediate and final outputs between runs as to not exceed your disk quotas.

You may submit your project by running:

$ submit proj1-1


Note: Your code MUST compile or you will NOT receive credit!!!

Please make sure to submit only Proj1.java and DoublePair.java. Before you submit, we strongly suggest you test your code and read the grading section below.

Also, make sure you are outputting exactly what we're asking and nothing else, or the autograder may mark your solution as wrong. If your formatting matches our reference solution, it should be fine.

Part 2 Instructions

Will be posted at a later date. As a general overview, we will be asking you to run your code on EC2 with certain target words and funcNums.
Grading

We're going to grade your code by running it in an automated test harness on sample data. We expect you to give [approximately] the right numeric answers, barring floating point roundoff. So don't modify the three f()'s given. We promise to avoid tests that are overly sensitive to roundoff.

Note again that your code MUST compile for you to receive credit. If you are having trouble implementing a way to find the minimum distance between a word and the target word, you may choose to implement the basic co-occurrence algorithm only for partial credit.

We will post a more detailed rubric in the next few days.

project 1: Co-occurence
