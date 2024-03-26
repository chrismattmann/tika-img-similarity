[Apache Tika](http://tika.apache.org/) File Similarity based on Jaccard distance, Edit distance & Cosine distance
===

This project demonstrates the usage of the [Tika-Python](http://github.com/chrismattmann/tika-python) package (Python port of Apache Tika) to compute file similarity based on metadata features.

The script can iterate over all files in the current directory, or specific files by command line, derive their metadata features, and  compute the union of all features. The union of all features becomes the "golden feature set" that all document features are compared to via intersect. The length of that intersect per file divided by the length of the unioned set becomes the similarity score.

Scores are sorted in reverse (descending) order which can be shown in three different Data-Driven document visualizaions. A companion project to this effort is [Auto Extractor](https://github.com/USCDataScience/autoextractor/wiki/Clustering-Tutorial) which uses [Apache Spark](http://spark.apache.org/) and [Apache Nutch](http://nutch.apache.org/) to take web crawl data, and produce D3-visualizations and clusters of similar pages.

Pre-requisite
===
- Install [Tika-Python](http://github.com/chrismattmann/tika-python)
- Install [Java Development Kit](https://www.oracle.com/java/technologies/javase-downloads.html)


Installation
===
```
git clone https://github.com/chrismattmann/tika-img-similarity
pip install tika editdistance
```
You can also check out [ETLlib](https://github.com/chrismattmann/etllib/tree/master/etl/imagesimilarity.py)

How to use
===

Optional: Compute similarity only on specific IANA MIME Type(s) inside a directory using **--accept**

Key-based comparison
--------------------
This compares metadata feature names as a golden feature set
```
#!/usr/bin/env python
python similarity.py -f [directory of files] [--accept [jpeg pdf etc...]]
or 
python similarity.py -c [file1 file2 file3 ...]
```
Value-based comparison
----------------------
This compares metadata feature names together with its value as a golden feature set
```
#!/usr/bin/env python
python value-similarity.py -f [directory of files] [--accept [jpeg pdf etc...]]
or 
python value-similarity.py -c [file1 file2 file3 ...]
```

Edit Distance comparison on Metadata Values
-------------------------------------------
- This computes pairwise similarity scores based on Edit Distance Similarity.
- **Similarity Score of 1 implies an identical pair of documents.**

```
#!/usr/bin/env python
python edit-value-similarity.py [-h] --inputDir INPUTDIR --outCSV OUTCSV [--accept [png pdf etc...]] [--allKeys]

--inputDir INPUTDIR  path to directory containing files

--outCSV OUTCSV      path to directory for storing the output CSV File, containing pair-wise Similarity Scores based on Edit distance

--accept [ACCEPT]    Optional: compute similarity only on specified IANA MIME Type(s)

--allKeys            Optional: compute edit distance across all metadata keys of 2 documents, else default to only intersection of metadata keys

```
```
Eg: python edit-value-similarity.py --inputDir /path/to/files --outCSV /path/to/output.csv --accept png pdf gif
```

Cosine Distance comparison on Metadata Values
---------------------------------------------
- This computes pairwise similarity scores based on Cosine Distance Similarity.
- **Similarity Score of 1 implies an identical pair of documents.**

```
#!/usr/bin/env python
python cosine_similarity.py [-h] --inputDir INPUTDIR --outCSV OUTCSV [--accept [png pdf etc...]]

--inputDir INPUTDIR  path to directory containing files

--outCSV OUTCSV      path to directory for storing the output CSV File, containing pair-wise Similarity Scores based on Cosine distance

--accept [ACCEPT]    Optional: compute similarity only on specified IANA MIME Type(s)

```

Similarity based on Stylistic/Authorship features
-------------------------------------------------
- This calculates pairwise cosine similarity on bag of signatures/features produced by extracting stylistic/authorship features from text.

```
#!/usr/bin/env python
python psykey.py --inputDir INPUTDIR --outCSV OUTCSV --wordlists WRODLIST_FOLDER

--inputDir INPUTDIR  path to directory containing files

--outCSV OUTCSV      path to directory for storing the output CSV File, containing pair-wise Similarity Scores based on Cosine distance of stylistic and authorship features

--wordlists WRODLIST_FOLDER    path to the folder that contains files that are word list belonging to different classes. eg Use the wordlist folder provided with the tika-similarity library. If adding your own, make sure that the file is .txt with one word per line. Also, the name of the file will be considered the name of the class.

```


D3 visualization
----------------

### Cluster viz 
- Jaccard Similarity
```
* python cluster-scores.py [-t threshold_value] (for generating cluster viz)
* open cluster-d3.html(or dynamic-cluster.html for interactive viz) in your browser
```
- Edit Distance & Cosine Similarity  
```
* python edit-cosine-cluster.py --inputCSV <PATH TO CSV FILE> --cluster <INTEGER OPTION> (for generating cluster viz)

  <PATH TO CSV FILE> - Path to CSV file generated by running edit-value-similarity.py or cosine_similarity.py
  <INTEGER OPTION> - Pass 0 to cluster based on x-coordinate, 1 to cluster based on y-coordinate, 2 to cluster based on similarity score

* open cluster-d3.html(or dynamic-cluster.html for interactive viz) in your browser
```

Default **threshold** value is 0.01.

<img src="docs/figs/cluster.png" width = "200px" height = "200px" style = "float:left">
<img src="docs/figs/interactive-cluster.png" width = "200px" height = "200px" style = "float:right">

### Circlepacking viz
- Jaccard Similarity
```
* python circle-packing.py (for generating circlepacking viz)
* open circlepacking.html(or dynamic-circlepacking.html for interactive viz) in your browser
```
- Edit Distance & Cosine Similarity  
```
* python edit-cosine-circle-packing.py --inputCSV <PATH TO CSV FILE> --cluster <INTEGER OPTION> (for generating circlepacking viz)

  <PATH TO CSV FILE> - Path to CSV file generated by running edit-value-similarity.py or cosine_similarity.py
  <INTEGER OPTION> - Pass 0 to cluster based on x-coordinate, 1 to cluster based on y-coordinate, 2 to cluster based on similarity score


* open circlepacking.html(or dynamic-circlepacking.html for interactive viz) in your browser
```
<img src="docs/figs/circlepacking.png" width = "200px" height = "200px" style = "float:left">
<img src="docs/figs/interactive-circlepacking.png" width = "200px" height = "200px" style = "float:right">

### Composite viz
This is a combination of cluster viz and circle packing viz.
The deeper color, the more the same attributes in the cluster.
```
* open compositeViz.html in your browser
```
![Image of composite viz](docs/figs/composite.png)


### Big data way
if you are dealing with big data, you can use it this way:
```
* python generateLevelCluster.py (for generating level cluster viz)
* open levelCluster-d3.html in your browser
```
You can set max number for each node **_maxNumNode**(default _maxNumNode = 10) in generateLevelCluster.py
![Image of level composite viz](docs/figs/level-composite.png)

### Chord Diagram viz

#### Getting the specific input for the visualizaion

The input of this part should be an output of Tika Similarity in a specific format - csv file with the columns : `x_coordinate`, `y_coordinate`, `similarity score`

For example, to build a chord diagram based on Tika Similarity related to jaccard similarity, the following must be done to get an input for this task:

- the Json files, based on which it is needed to figure out the jaccard similarity must be put into `splits` folder
- the `jaccard.similarity.py must be used` - it is possible to get it in the tika similarity repository 

```shell
git clone http://github.com/chrismattmann/tika-similarity
```

After that, to get the file with the coordinates and the similarity score, the following code must be implemented:

```shell
python ../jaccard_similarity.py --inputDir splits/ --outCSV jaccard.csv
```

As an output, the user would get a `jaccard.csv` file  with the columns : `x_coordinate`, `y_coordinate`, `similarity score`

Note: it is also possible to work with other tika similarity functionality, such as,e.g., Edit distance similarity (for that, it is needed to create an edit.csv file with the columns : `x_coordinate`, `y_coordinate`, `similarity score` or Cosine similarity by implementing the functions from the tutorial: https://github.com/chrismattmann/tika-similarity/wiki/Tutorial)

#### Creating the visualization

For transforming the file (which was in the previous step)- clusterizing that data- was used the following Python code:

```shell
python html/clusterization.py
```
Note: it is important to manually incude the csv filename (e.g. jaccard.csv) in the 6th line of the clusterization.py script.

Html file is transferred from `html` to the working directory for creating the template for the visualization 

```shell
cp html/chord.html .
```
Note: it is important to incude the csv filename (e.g. jaccard.csv) in the 43th line of the chord.html script.

For visualization the python server is launched:

```shell
python -mhttp.server 8082
```
#### Output:
  &emsp; Chord diagram (this fires up a server on port 8082, so then visit http://localhost:8082/chord.html) 
  


Questions, comments?
===================
Send them to [Chris A. Mattmann](mailto:chris.a.mattmann@jpl.nasa.gov).

Contributors
============
* Chris A. Mattmann, JPL
* Dongni Zhao, USC
* Harshavardhan Manjunatha, USC
* Thamme Gowda, USC
* Ayberk Yılmaz, USC
* Aravind Ram, USC
* Aishwarya Parameshwaran, USC
* Rashmi Nalwad, USC
* Asitang Mishra, JPL
* Suzanne Stathatos, JPL

License
===

This project is licensed under the [Apache License, version 2.0](http://www.apache.org/licenses/LICENSE-2.0).







