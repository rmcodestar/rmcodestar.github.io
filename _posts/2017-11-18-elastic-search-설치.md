---
layout: post
title: Elasticsearch 설치하기
category: Backend
tag: Elasticsearch
---

### elasticSearch 설치하기 앞서
* elasicsearch : 5.6.3
* required : JDK1.7+

### brew를 이용해서 설치하기
```Sh
$ brew install elasticsearch
```

```shell
==> Downloading https://artifacts.elastic.co/downloads/elasticsearch/elasticsear

######################################################################## 100.0%

==> Caveats

Data:    /usr/local/var/elasticsearch/elasticsearch_rmcodestar/

Logs:    /usr/local/var/log/elasticsearch/elasticsearch_rmcodestar.log

Plugins: /usr/local/opt/elasticsearch/libexec/plugins/

Config:  /usr/local/etc/elasticsearch/

plugin script: /usr/local/opt/elasticsearch/libexec/bin/elasticsearch-plugin

To have launchd start elasticsearch now and restart at login:

  brew services start elasticsearch

Or, if you don't want/need a background service you can just run:

  elasticsearch

==> Summary

🍺  /usr/local/Cellar/elasticsearch/5.6.3: 104 files, 35.9MB, built in 43 seconds

```

* 위 방법이 아니어도 zip, tar로 다운받아서 압축을 풀어도 된다 ([5.6.3버전 다운로드 링크](https://www.elastic.co/downloads/past-releases/elasticsearch-5-6-3))



### elasticsearch 실행해보기

* run elasticsearch 

  ```shell
  $ elasticsearch  # /bin 에 있는 elasticsearch 실행파일
  ```
  ​

* http://localhost:9200/ 접속 시 아래 json이 응답되는지 확인

  ```json
  {
    "name" : "ahwF6mH",
    "cluster_name" : "elasticsearch_rmcodestar",
    "cluster_uuid" : "n9hs23tIQoiQ1LWDP75WQQ",
    "version" : {
      "number" : "5.6.3",
      "build_hash" : "1a2f265",
      "build_date" : "2017-10-06T20:33:39.012Z",
      "build_snapshot" : false,
      "lucene_version" : "6.6.1"
    },
    "tagline" : "You Know, for Search"
  }
  ```




### 설치가 되었는지 테스트 해보기

#### sample 다운로드하기

download test data and sh : https://github.com/dakrone/elasticsearch-in-action

```Sh
$ git clone https://github.com/dakrone/elasticsearch-in-action.git -b 5.x
$ elasticsearch-in-action/populate.sh
```



#### search test

elasticsearch가 제대로 동작했는지 알아보기 위해서 아래 url를 호출해보자

* url : `http://localhost:9200/get-together/group/_search?q=elasticsearch&stored_fields=name,location&size=1&pretty`


* 주의할 점
  * ​
  * The parameter [fields] is no longer supported, please use [stored_fields] 

```json
// 20171017220420
// http://localhost:9200/get-together/group/_search?q=elasticsearch&stored_fields=name,location&size=1&pretty

{
  "took": 40,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1.2422469,
    "hits": [
      {
        "_index": "get-together",
        "_type": "group",
        "_id": "2",
        "_score": 1.2422469
      }
    ]
  }
}
```



### 모니터링해보기

chome의 elasticsearch head 확장 플러그인을 이용하여 모니터링을 할 수 있다
