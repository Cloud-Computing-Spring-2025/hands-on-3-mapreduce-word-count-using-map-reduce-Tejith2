# Word Count using Hadoop MapReduce

## Project Overview
The objective of this project is to implement a **Word Count** program using **Hadoop MapReduce**. This program processes a given text dataset and counts the occurrences of each word, displaying the output in descending order of frequency. The implementation is done using Java and executed in a **Hadoop cluster** running inside Docker containers.

## Approach and Implementation
The project follows the **MapReduce** programming model to perform word count:

### **1. Mapper Class**
- The `WordMapper.java` file defines the **mapper**.
- It reads each line of the input file, tokenizes it into words, and emits each word with a count of `1`.

#### **Code Snippet:**
```java
public class WordMapper extends MapReduceBase implements Mapper<LongWritable, Text, Text, IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(LongWritable key, Text value, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
        StringTokenizer tokenizer = new StringTokenizer(value.toString());
        while (tokenizer.hasMoreTokens()) {
            word.set(tokenizer.nextToken());
            output.collect(word, one);
        }
    }
}
```

### **2. Reducer Class**
- The `WordReducer.java` file defines the **reducer**.
- It aggregates the counts for each word and emits the final count.

#### **Code Snippet:**
```java
public class WordReducer extends MapReduceBase implements Reducer<Text, IntWritable, Text, IntWritable> {
    public void reduce(Text key, Iterator<IntWritable> values, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
        int sum = 0;
        while (values.hasNext()) {
            sum += values.next().get();
        }
        output.collect(key, new IntWritable(sum));
    }
}
```

### **3. Controller Class**
- The `Controller.java` class is responsible for setting up and running the Hadoop job.
- It configures input and output paths, assigns mapper and reducer classes, and submits the job.

## **Execution Steps**

### **1. Start the Hadoop Cluster**
Run the following command to launch the Hadoop cluster:
```bash
docker compose up -d
```

### **2. Build the Project using Maven**
```bash
mvn clean install
```

### **3. Move JAR File to Shared Folder**
```bash
mv target/*.jar shared-folder/input/jar/
```

### **4. Copy JAR File to Hadoop Container**
```bash
docker cp shared-folder/input/jar/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### **5. Copy Input Dataset to Hadoop Container**
```bash
docker cp shared-folder/input/data/input.txt resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### **6. Access the Hadoop ResourceManager Container**
```bash
docker exec -it resourcemanager /bin/bash
```

### **7. Set Up HDFS and Upload Dataset**
```bash
hadoop fs -mkdir -p /input/dataset
hadoop fs -put /opt/hadoop-3.2.1/share/hadoop/mapreduce/input.txt /input/dataset
```

### **8. Run the MapReduce Job**
```bash
hadoop jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar com.example.controller.Controller /input/dataset/input.txt /output
```

### **9. View the Output**
```bash
hadoop fs -cat /output/*
```

### **10. Retrieve Output from HDFS to Local System**
```bash
hdfs dfs -get /output /opt/hadoop-3.2.1/share/hadoop/mapreduce/
exit
docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output/ shared-folder/output/
```

## **Challenges Faced & Solutions**
1. **Issue:** JAR file not found in the container.
   - **Solution:** Ensured the correct path and used `docker cp` to transfer the file.

2. **Issue:** Incorrect file permissions on the JAR file.
   - **Solution:** Used `chmod +r` inside the container to ensure the JAR file had the correct permissions.

3. **Issue:** Hadoop job failing due to missing input file.
   - **Solution:** Verified the file upload using `hadoop fs -ls /input/dataset`.

## **Sample Input and Output**

### **Input (`input.txt`)**
```text
Hadoop is an open-source framework that facilitates the distributed processing of vast amounts of data across a network of computers. It is designed to scale from a single system to thousands of nodes, utilizing local storage and computation to enhance efficiency. Rather than relying on specialized hardware for reliability, Hadoop incorporates built-in mechanisms to detect and recover from failures at the application level. One of its core components, MapReduce, enables large-scale data processing by dividing tasks into parallel operations across multiple nodes.
```

### **Expected Output**
```text
Hadoop  2
It      1
MapReduce,      1
One     1
Rather  1
a       2
across  2
amounts 1
an      1
and     2
application     1
at      1
built-in        1
by      1
components,     1
computation     1
computers.      1
core    1
data    2
designed        1
detect  1
distributed     1
dividing        1
efficiency.     1
enables 1
enhance 1
facilitates     1
failures        1
for     1
framework       1
from    2
hardware        1
incorporates    1
into    1
is      2
its     1
large-scale     1
level.  1
local   1
mechanisms      1
multiple        1
network 1
nodes,  1
nodes.  1
of      5
on      1
open-source     1
operations      1
parallel        1
processing      2
recover 1
reliability,    1
relying 1
scale   1
single  1
specialized     1
storage 1
system  1
tasks   1
than    1
that    1
the     2
thousands       1
to      4
utilizing       1
vast    1
```

