# Hadoop-Pig-ETL-AWS
AWS ElasticMapReduce with Apache Pig to create data for histogram of outdegrees of vertices from nodes in a network, .5TB of data. A histogram of this data tells us whether the network was generated randomly (counts will have an exponential distribution) or was man-made (counts will have Zipf distribution; points will be linear on log-log scale). I'm planning to go back on AWS and get the output and work up a datavis. I wrote some of this code but credit and thanks to the excellent Data Science At Scale Coursera specialization.

register s3n://directory.aws.amazon.com/myudfs.jar

-- initially: load the test file into Pig: this is 250KB

-- raw = LOAD 's3n://directory.aws.amazon.com/test-file' USING TextLoader as (line:chararray);

-- later: load a bigger file: this one is 2GB.

-- raw = LOAD 's3n://directory.aws.amazon.com/bigfile-chunk-000' USING TextLoader as (line:chararray);

-- load the entire big file, 0.5TB.

raw = LOAD 's3n://directory.aws.amazon.com/bigfile-chunk-*' USING TextLoader as (line:chararray);

-- parse each line into ntriples

ntriples = foreach raw generate FLATTEN(myudfs.RDFSplit3(line)) as (subject:chararray,predicate:chararray,object:chararray);

--group the n-triples by subject column

subjects = group ntriples by (subject) PARALLEL 50;

-- flatten the objects out (because group by produces a tuple of each subject

-- in the first column, and we want each subject to be a string, not a tuple),

-- and count the number of tuples associated with each subject

count_by_subject = foreach subjects generate flatten($0), COUNT($1) as count PARALLEL 50;

-- order the resulting tuples by their count in descending order (I think this is useful to speed the "shuffle" process?)

count_by_subject_ordered = order count_by_subject by (count)  PARALLEL 50;

--group the count_by_subject_ordered by count column

xxxx = group count_by_subject_ordered by (count) PARALLEL 50;

-- count the number of occurrences of each count

hist_points = foreach xxxx generate flatten($0), COUNT($1) as count PARALLEL 50;

-- while running locally, put results into

-- store hist_points into '/tmp/myoutput' using PigStorage();

store hist_points into '/user/hadoop/results' using PigStorage();

