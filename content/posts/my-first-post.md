+++ 
draft = false
date = 2021-11-11T11:00:45+08:00
title = "Pyspark读取和保存Json文件"
description = "PySpark Read And Write JSON file "
slug = ""
authors = ['he110w0r1d']
tags = ['pyspark', 'json', 'io']
categories = ['big data', 'spark']
externalLink = ""
series = []
+++


## 目录
- [将JSON文件读入DataFrame](#将JSON文件读入DataFrame)
  - [读取包含多个单行json的单个Json文件](#读取包含多行json的单个Json文件)
  - [使用自定义的schema读取json文件](#使用自定义的schema读取json文件)
  - [读取包含多行json列表的单个json文件](#读取包含多行json列表的单个json文件)
  - [一次读取多个json文件](#一次读取多个json文件)
  - [读取指定目录下的所有json文件](#读取指定目录下的所有json文件)
  - [递归读取指定目录下所有的json文件](#递归读取指定目录下所有的json文件)
- [将dataframe保存为json文件](#将dataframe保存为json文件)
   - [保存模式](#保存模式)
   - [保存选项](#保存选项)
 

---
## 将JSON文件读入DataFrame

### 读取包含多行json的单个Json文件

forexample.json文件:
```json
{"id": 0, "name": "bob", "time": "2021-07-23 08:10:59"}
{"id": 1, "name": "alice", "time": "2021-07-21 02:10:00"}
{"id": 2, "name": "jack", "time": "2021-07-24 08:15:51"}
```

```python
df = spark.read.json("forexample.json")
df.printSchema()
df.show()
```
输出：
```
root
 |-- id: long (nullable = true)
 |-- name: string (nullable = true)
 |-- time: string (nullable = true)

+---+-----+-------------------+
| id| name|               time|
+---+-----+-------------------+
|  0|  bob|2021-07-23 08:10:59|
|  1|alice|2021-07-21 02:10:00|
|  2| jack|2021-07-24 08:15:51|
+---+-----+-------------------+
```

### 使用自定义的schema读取json文件
读取json文件时，不支持数据类型推断，可在读取时指定数据类型。
```python
schema = StructType([
      StructField("id",IntegerType(),True),
      StructField("name",StringType(),True),
      StructField("time",TimestampType(),True)
  ])

df_with_schema = spark.read.schema(schema) \
        .json("./forexample.json")
df_with_schema.printSchema()
df_with_schema.show()
```
[查看更多数据类型](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql.html#data-types)

### 读取包含多行json列表的单个Json文件

multiline-forexample.json文件:
```json
[{
    "id": 0, 
    "name": "bob",
    "time": "2021-07-23 08:10:59"
 },
{
    "id": 1,
    "name": "alice", 
    "time": "2021-07-21 02:10:00"
},
{
    "id": 2, 
    "name": "jack",
    "time": "2021-07-24 08:15:51"
}]
```

```python
multiline_df = spark.read.option("multiline","true") \
      .json("multiline-forexample.json")
multiline_df.show()    
```


### 一次读取多个json文件

```python
df2 = spark.read.json(jsonFilePathList)
df2.show()  
```

### 读取指定目录下的所有json文件

```python
df3 = spark.read.json("path/*.json")
df3.show()
```
### 递归读取指定目录下所有的json文件
{{< notice tip >}}
Spark 3.0下支持
{{< /notice >}}

```python
recursive_loaded_df = spark.read\
    .option("recursiveFileLookup", "true")\
    .json(path)

recursive_loaded_df.show()    
```
或者
```python
recursive_loaded_df = spark.read.format("json")\
    .option("recursiveFileLookup", "true")\
    .load(path)
recursive_loaded_df.show()
```

---

## 将dataframe保存为json文件
```
df.write.mode('Overwrite').json("./zipcodes.json")
```

### 保存模式
overwrite – 覆盖：覆盖已存在的文件

append – 附加：将数据附加到已存在文件

ignore – 忽略：当文件已经存在时，忽略写操作

errorifexists or error – 抛出错误：默认选项，文件已经存在时，返回错误。

### 保存选项

timeZone、dateFormat、timestampFormat、encoding、lineSep、compression等

举个栗子：

将数据保存json文件时，格式化时间戳
```python
df_with_schema.write.mode('Overwrite')\
    .option("timestampFormat", "yyyy-MM-dd HH:mm:ss")\
    .json("./timestampFormat.json")
```

timestampFormat.json
```
{"id":0,"name":"bob","time":"2021-07-23 08:10:59"}
```

## 参考

1. https://sparkbyexamples.com/pyspark/pyspark-read-json-file-into-dataframe/
2. https://spark.apache.org/docs/latest/sql-data-sources-generic-options.html/