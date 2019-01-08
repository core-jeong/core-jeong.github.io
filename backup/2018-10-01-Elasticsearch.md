---
layout: post
current: post
cover:  assets/images/cloud-computing.png
navigation: True
title: Elasticsearch설치 및 기본가이드
date: 2016-10-01 17:00:00
tags: [Bigdata]
category: bigdata
class: post-template
author: core-jeong
---

Elasticsearch설치 및 기본가이드
===

## 1. 우분투에 Elasticsearch설치
### 1.1. 관련패키지 설치
```bash
$ apt-get -y install vim curl wget openjdk-8-jdk

$ java -version
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-8u181-b13-0ubuntu0.18.04.1-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```
### 1.2. elasticsearch 설치
```bash
$ wget https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-latest.deb

$ dpkg -i elasticsearch-latest.deb

$ systemctl enable elasticsearch.service
```

### 1.3. elasticsearch Start/Stop
```bash
$ service elasticsearch start #start

$ curl -XGET 'localhost:9200' #check
{
  "name" : "Jigsaw",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "a3JP2gGJSneVB1RbRF_S-Q",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}

$ service elasticsearch stop #stop
```

## 2. Elasticsearch기본 개념
### 2.1. Elasticsearch특징
* 역색인(Inverted Index)를 사용하여 빠르게 데이터 검색

### 2.2. 관계형DB 간 용어비교

| Elastic Search | Relational DB |
|:--------- | :--------- |
| Index | Database |
| Type | Table |
| Document | Row |
| Field | Column |
| Mapping | Schema |

### 2.3. 관계형DB간 명령비교

| Elastic Search | Relational DB | CRUD |
|:--------- | :--------- | :--------- |
| GET | Select | Read |
| PUT | Update | Update |
| POST | Insert | Create |
| DELETE | Delete | Delete |

## 3. Elasticsearch 사용법
### 3.1. Index 조작
#### 3.1.1. Index 읽기
* 명령어: "curl -XGET [URL]/[INDEX_NAME]"
```bash
$ curl -XGET http://localhost:9200/classes
{"error":{"root_cause":[{"type":"index_not_found_exception","reason":"no such index","resource.type":"index_or_alias","resource.id":"classes","index":"classes"}],"type":"index_not_found_exception","reason":"no such index","resource.type":"index_or_alias","resource.id":"classes","index":"classes"},"status":404}
```
* 깔끔한 명령어: "curl -XGET [URL]/[INDEX_NAME]?pretty"
```bash
$ curl -XGET http://localhost:9200/classes?pretty
{
  "error" : {
    "root_cause" : [ {
      "type" : "index_not_found_exception",
      "reason" : "no such index",
      "resource.type" : "index_or_alias",
      "resource.id" : "classes",
      "index" : "classes"
    } ],
    "type" : "index_not_found_exception",
    "reason" : "no such index",
    "resource.type" : "index_or_alias",
    "resource.id" : "classes",
    "index" : "classes"
  },
  "status" : 404
}
```

#### 3.1.2. Index 생성
* 명령어: "curl -XPUT [URL]/[INDEX_NAME]"

```bash
$ curl -XPUT http://localhost:9200/classes
{"acknowledged":true}

$ curl -XGET http://localhost:9200/classes?pretty
{
  "classes" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "creation_date" : "1538387320866",
        "number_of_shards" : "5",
        "number_of_replicas" : "1",
        "uuid" : "PGtxYiSwQeiAPIbIuLreEQ",
        "version" : {
          "created" : "2040699"
        }
      }
    },
    "warmers" : { }
  }
}
```

#### 3.1.3. Index 삭제
* 명령어: "curl -XDELETE [URL]/[INDEX_NAME]"
```bash
$ curl -XDELETE http://localhost:9200/classes
{"acknowledged":true}

$ curl -XGET http://localhost:9200/classes?pretty
{
  "error" : {
    "root_cause" : [ {
      "type" : "index_not_found_exception",
      "reason" : "no such index",
      "resource.type" : "index_or_alias",
      "resource.id" : "classes",
      "index" : "classes"
    } ],
    "type" : "index_not_found_exception",
    "reason" : "no such index",
    "resource.type" : "index_or_alias",
    "resource.id" : "classes",
    "index" : "classes"
  },
  "status" : 404
}
```

### 3.2. Document 조작
#### 3.2.1. Document 생성
* 명령어: "curl -XPOST [URL]/[INDEX_NAME]/[TYPE_NAME]/[ID] -d '{[DATA]}'"
* Index가 없어도 명시해주면 자동생성
```bash
$ curl -XPOST http://localhost:9200/classes/class/1/ -d '{"title":"Algorithm", "professor":"john"}'
{"_index":"classes","_type":"class","_id":"1","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}

$ curl -XGET http://localhost:9200/classes/class/1/?pretty
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "title" : "Algorithm",
    "professor" : "john"
  }
}
```

* 파일로 생성: "curl -XPOST [URL]/[INDEX_NAME]/[TYPE_NAME]/[ID] -d @[FILE_PATH]"
```bash
$ cat ./oneclass.json
{
        "title" : "Machine Learning",
        "professor" : "Jeong",
        "major" : "Computer Science",
        "semester" : ["spring", "fall"],
        "student_cont" : 100,
        "unit" : 3,
        "rating" : 5
}

$ curl -XPOST http://localhost:9200/classes/class/1/ -d @oneclass.json
{"_index":"classes","_type":"class","_id":"1","_version":2,"_shards":{"total":2,"successful":1,"failed":0},"created":false}

$ curl -XGET http://localhost:9200/classes/class/1/?pretty
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 2,
  "found" : true,
  "_source" : {
    "title" : "Machine Learning",
    "professor" : "Jeong",
    "major" : "Computer Science",
    "semester" : [ "spring", "fall" ],
    "student_cont" : 100,
    "unit" : 3,
    "rating" : 5
  }
}
```

#### 3.2.2. Document 업데이트
* 명령어: "curl -XPOST [URL]/[INDEX_NAME]/[TYPE_NAME]/[ID]/_update?pretty -d '{[DATA]}'"
```bash
$ curl -XPOST http://localhost:9200/classes/class/1/_update?pretty -d '{"doc":{"unit":1}}'
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 5,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  }
}

$ curl -XGET http://localhost:9200/classes/class/1/?pretty
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 5,
  "found" : true,
  "_source" : {
    "title" : "Algorithm",
    "professor" : "john",
    "unit" : 1
  }
}

$ curl -XPOST http://localhost:9200/classes/class/1/_update?pretty -d '{"doc":{"unit":2}}'
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 6,
  "_shards" : {
    "total" : 0,
    "successful" : 0,
    "failed" : 0
  }
}

$ curl -XGET http://localhost:9200/classes/class/1/?pretty
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 6,
  "found" : true,
  "_source" : {
    "title" : "Algorithm",
    "professor" : "john",
    "unit" : 2
  }
}
```
#### 3.2.3. Bulk POST
* 여러개의 Document를 한번에 Elasticsearch에 삽입하는 기법
* 명령어: "curl -XPOST [URL]/[INDEX_NAME]/_update?pretty --data-binary @[FILE_PATH]"
```bash
$ cat classes.json
{ "index" : { "_index" : "classes", "_type" : "class", "_id" : "1" } }
{"title" : "Machine Learning","Professor" : "Minsuk Heo","major" : "Computer Science","semester" : ["spring", "fall"],"student_count" : 100,"unit" : 3,"rating" : 5, "submit_date" : "2016-01-02", "school_location" : {"lat" : 36.00, "lon" : -120.00}}
...

$ curl -XPOST http://localhost:9200/_bulk?pretty --data-binary @classes.json
{
  "took" : 1530,
  "errors" : false,
  "items" : [ {
    "index" : {
      "_index" : "classes",
      "_type" : "class",
      "_id" : "1",
      "_version" : 7,
      "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
      },
      "status" : 200
    }
  },
  ...
}

$ curl -XGET http://localhost:9200/classes/class/1?pretty
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 7,
  "found" : true,
  "_source" : {
    "title" : "Machine Learning",
    "Professor" : "Minsuk Heo",
    "major" : "Computer Science",
    "semester" : [ "spring", "fall" ],
    "student_count" : 100,
    "unit" : 3,
    "rating" : 5,
    "submit_date" : "2016-01-02",
    "school_location" : {
      "lat" : 36.0,
      "lon" : -120.0
    }
  }
}
```

#### 3.2.4. Elasticsearch Mapping
* Mapping은 DB의 Schema와 같음
* 매핑없이 Elasticsearch에 데이터를 넣을 수 있지, 무결성이 보장되지 않아 매우 위험한 일
* 명령어:  "curl -XPUT [URL]/[INDEX_NAME]/[TYPE_NAME]/_mapping -d @[FILE_PATH]"
```bash
$ curl -XPUT http://localhost:9200/classes
{"acknowledged":true}

$ curl -XGET http://localhost:9200/classes?pretty
{
  "classes" : {
    "aliases" : { },
    "mappings" : { }, # mappings이 비어있는것을 확인
    "settings" : {
      "index" : {
        "creation_date" : "1538474963757",
        "number_of_shards" : "5",
        "number_of_replicas" : "1",
        "uuid" : "E37ZppPiSUGoRfSyWIJ0Jw",
        "version" : {
          "created" : "2040699"
        }
      }
    },
    "warmers" : { }
  }
}

$ cat classesRating_mapping.json
cat ./classesRating_mapping.json
{
  "class" : {
    "properties" : {
      "title" : {
        "type" : "string"
      },
...
    }
  }
}

$  curl -XPUT http://localhost:9200/classes/class/_mapping -d @classesRating_mapping.json
{"acknowledged":true}

$ curl -XGET http://localhost:9200/classes/?pretty
{
  "classes" : {
    "aliases" : { },
    "mappings" : {
      "class" : {
        "properties" : {
          "major" : {
            "type" : "string"
          },
        }
      }
    }
  }
}
...

$ curl -XPOST http://localhost:9200/_bulk?pretty --data-binary @classes.json
{
 "took" : 481,
 "errors" : false,
 "items" : [ {
   "index" : {
     "_index" : "classes",
     "_type" : "class",
     "_id" : "1",
     "_version" : 1,
     "_shards" : {
       "total" : 2,
       "successful" : 1,
       "failed" : 0
     },
     "status" : 201
   }
 },
 ...
}

$ curl -XGET http://localhost:9200/classes/class/1/?pretty
{
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "title" : "Machine Learning",
    "Professor" : "Minsuk Heo",
...
  }
}
```

#### 3.2.5. Elasticsearch Search
* 명령어: "curl -XGET [URL]/[INDEX_NAME]/[TYPE_NAME]/_search?pretty"
```bash
$ cat simple_basketball.json
{ "index" : { "_index" : "basketball", "_type" : "record", "_id" : "1" } }
{"team" : "Chicago Bulls","name" : "Michael Jordan", "points" : 30,"rebounds" : 3,"assists" : 4, "submit_date" : "1996-10-11"}
{ "index" : { "_index" : "basketball", "_type" : "record", "_id" : "2" } }
{"team" : "Chicago Bulls","name" : "Michael Jordan","points" : 20,"rebounds" : 5,"assists" : 8, "submit_date" : "1996-10-11"}

$ curl -XPOST localhost:9200/_bulk --data-binary @simple_basketball.json
{"took":938,"errors":false,"items":[{"index":{"_index":"basketball","_type":"record","_id":"1","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"status":201}},{"index":{"_index":"basketball","_type":"record","_id":"2","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"status":201}}]}

$ curl -XGET http://localhost:9200/basketball/record/_search?pretty
{
  "took" : 107,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
...
}
```
* URI검색: "curl -XGET '[URL]/[INDEX_NAME]/[TYPE_NAME]/_search?q=[ELEMENT_NAME]:[SEARCH_VALUE]&pretty'"
* q는 query의 약어
```bash
$ curl -XGET 'http://localhost:9200/basketball/record/_search?q=points:30&pretty'
{
  "took" : 23,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.30685282,
    "hits" : [ {
      "_index" : "basketball",
      "_type" : "record",
      "_id" : "1",
      "_score" : 0.30685282,
      "_source" : {
        "team" : "Chicago Bulls",
        "name" : "Michael Jordan",
        "points" : 30,
        "rebounds" : 3,
        "assists" : 4,
        "submit_date" : "1996-10-11"
      }
    } ]
  }
}
```

* Request Body검색: "curl -XGET [URL]/[INDEX_NAME]/[TYPE_NAME]/_search -d '{[DATA]}'"
* 많은 옵션이 있음
```bash
$ curl -XGET localhost:9200/basketball/record/_search -d '
> {
> "query" : {
> "term" : {"points":30}
> }
> }'
{"took":21,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":1,"max_score":0.30685282,"hits":[{"_index":"basketball","_type":"record","_id":"1","_score":0.30685282,"_source":{"team" : "Chicago Bulls","name" : "Michael Jordan", "points" : 30,"rebounds" : 3,"assists" : 4, "submit_date" : "1996-10-11"}}]}}
```

#### 3.2.6. Elasticsearch Aggregations
* Document내에서 조합을 통해 값을 도출할때 사용
* 포멧
```text
"aggregations" : {
  "<aggregation_name>" : {
    "<aggregation_type>" : {
      <aggregation_body>
    }
    [, "meta" : { [<meta_data_body>] } ]?
    [, "aggregations" : { [<sub_aggregation>]+ } ]?
  }
  [,"<aggregation_name_2>" : { ... } ]*
}
```
* 명령어 : "curl -XGET [URL]/_search?pretty --data-binary @[FILE_PATH]"
#### 3.2.6.1. Metric Aggregations
* 평균, 최대값, 최소값 등 산술계산에 사용
```bash
$ cat avg_points_aggs.json
{
        "size" : 0, # 보고깊은 결과값만 보기 위해
        "aggs" : {
                "avg_score" : { # 어그리게이션 이름 명시
                        "avg" : { # 평균값을 구할것이라고 명시
                                "field" : "points" # 어떤 필드값을 사용할건지 명시
                        }
                }
        }
}

$ curl -XGET http://localhost:9200/_search?pretty --data-binary @avg_points_aggs.json
{
  "took" : 285,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 27,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "avg_score" : {
      "value" : 25.0
    }
  }
}

$ cat max_points_aggs.json
{
        "size" : 0,
        "aggs" : {
                "max_score" : {
                        "max" : {
                                "field" : "points"
                        }
                }
        }
}

$ curl -XGET http://localhost:9200/_search?pretty --data-binary @max_points_aggs.json
{
  "took" : 78,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 27,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "max_score" : {
      "value" : 30.0
    }
  }
}

$ cat min_points_aggs.json
{
        "size" : 0,
        "aggs" : {
                "min_score" : {
                        "min" : {
                                "field" : "points"
                        }
                }
        }
}

$ curl -XGET http://localhost:9200/_search?pretty --data-binary @min_points_aggs.json
{
  "took" : 54,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 27,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "min_score" : {
      "value" : 20.0
    }
  }
}

$ cat sum_points_aggs.json
{
        "size" : 0,
        "aggs" : {
                "sum_score" : {
                        "sum" : {
                                "field" : "points"
                        }
                }
        }
}

$  curl -XGET http://localhost:9200/_search?pretty --data-binary @sum_points_aggs.json
{
  "took" : 33,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 27,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "sum_score" : {
      "value" : 50.0
    }
  }
}

$ cat stats_points_aggs.json
{
        "size" : 0,
        "aggs" : {
                "stats_score" : {
                        "stats" : {
                                "field" : "points"
                        }
                }
        }
}

$ curl -XGET http://localhost:9200/_search?pretty --data-binary @stats_points_aggs.json
{
  "took" : 49,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 27,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "stats_score" : {
      "count" : 2,
      "min" : 20.0,
      "max" : 30.0,
      "avg" : 25.0,
      "sum" : 50.0
    }
  }
}
```

#### 3.2.6.2. Bucket Aggregations
* Documents를 조합해서 어떠한 값을 나타내는 방법
* 관계형DB에서 group-by와 같다
```bash
$ curl -XPOST http://localhost:9200/_bulk --data-binary @twoteam_basketball.json
{"took":205,"errors":false,"items":[{"index":{"_index":"basketball","_type":"record","_id":"1","_version":2,"_shards":{"total":2,"successful":1,"failed":0},"status":200}},{"index":{"_index":"basketball","_type":"record","_id":"2","_version":2,"_shards":{"total":2,"successful":1,"failed":0},"status":200}},{"index":{"_index":"basketball","_type":"record","_id":"3","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"status":201}},{"index":{"_index":"basketball","_type":"record","_id":"4","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"status":201}}]}

$ cat terms_aggs.json
{
        "size" : 0,
        "aggs" : {
                "players" : { # 어그리게이션 이름
                        "terms" : { # 사용할 어그리게이션
                                "field" : "team" # 사용할 필드
                        }
                }
        }
}

$ curl -XGET http://localhost:9200/_search?pretty --data-binary @terms_aggs.json
{
  "took" : 207,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 29,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "players" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [ {
        "key" : "chicago",
        "doc_count" : 2
      }, {
        "key" : "la",
        "doc_count" : 2
      } ]
    }
  }
}

# 통계
$ cat stats_by_team.json
{
        "size" : 0,
        "aggs" : {
                "team_stats" : {
                        "terms" : {
                                "field" : "team"
                        },
                        "aggs" : {
                                "stats_score" : {
                                        "stats" : {
                                                "field" : "points"
                                        }
                                }
                        }
                }
        }
}
$ curl -XGET http://localhost:9200/_search?pretty --data-binary @stats_by_team.json
{
  "took" : 51,
  "timed_out" : false,
  "_shards" : {
    "total" : 15,
    "successful" : 15,
    "failed" : 0
  },
  "hits" : {
    "total" : 29,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "team_stats" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [ {
        "key" : "chicago",
        "doc_count" : 2,
        "stats_score" : {
          "count" : 2,
          "min" : 20.0,
          "max" : 30.0,
          "avg" : 25.0,
          "sum" : 50.0
        }
      }, {
        "key" : "la",
        "doc_count" : 2,
        "stats_score" : {
          "count" : 2,
          "min" : 30.0,
          "max" : 40.0,
          "avg" : 35.0,
          "sum" : 70.0
        }
      } ]
    }
  }
}
```

> 본 포스트는 Youtube Minsuk Heo 허민석님의 강의를 정리한 내용입니다. 문제가 될 경우 삭제하겠습니다.
