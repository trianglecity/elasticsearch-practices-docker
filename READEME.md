
##
##	Elasticsearch on Docker
##

NOTICE 1 : the examples are based on

	https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

	https://www.elastic.co/guide/en/x-pack/current/setting-up-authentication.html

	https://www.elastic.co/guide/en/elasticsearch/guide/current/_indexing_employee_documents.html

NOTICE 2: two terminals are required.

##

[terminal-1 1] git clone this-source-code-folder

[terminal-1 2] cd downloaded-source-code-folder

[terminal-1 3] docker pull docker.elastic.co/elasticsearch/elasticsearch:5.4.0

[terminal-1 4] docker-compose --version

	docker-compose version: 1.3.1
	CPython version: 2.7.10
	OpenSSL version: OpenSSL 1.0.2d 9 Jul 2015


[terminal-1 5] sudo -i (if docker-compose --version is less than 1.6)

[terminal-1 6] curl -L https://github.com/docker/compose/releases/download/1.6.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

	  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  Dload  Upload   Total   Spent    Left  Speed
	100   600    0   600    0     0   1483      0 --:--:-- --:--:-- --:--:--  1481
	100 7754k  100 7754k    0     0  1810k      0  0:00:04  0:00:04 --:--:-- 2637k

[terminal-1 7] sudo chmod +x /usr/local/bin/docker-compose

[terminal-1 8] sudo cp /usr/local/bin/docker-compose /usr/bin

[terminal-1 9] docker-compose --version
	
	docker-compose version 1.6.0, build d99cad6
	
	
[terminal-1 10] sudo sysctl -w vm.max_map_count=262144

[terminal-1 11] sudo docker-compose up

[terminal-1 appendix] docker-compose.yml looks like this
	

	version: '2'
	services:
	  elasticsearch1:
	    image: docker.elastic.co/elasticsearch/elasticsearch:5.4.0
	    container_name: elasticsearch1
	    environment:
	      - cluster.name=docker-cluster
	      - bootstrap.memory_lock=true
	      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
	    ulimits:
	      memlock:
	        soft: -1
	        hard: -1
	    mem_limit: 1g
	    volumes:
	      - esdata1:/usr/share/elasticsearch/data
	    ports:
	      - 9200:9200
	    networks:
	      - esnet
	  elasticsearch2:
	    image: docker.elastic.co/elasticsearch/elasticsearch:5.4.0
	    environment:
	      - cluster.name=docker-cluster
	      - bootstrap.memory_lock=true
	      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
	      - "discovery.zen.ping.unicast.hosts=elasticsearch1"
	    ulimits:
	      memlock:
	        soft: -1
	        hard: -1
	    mem_limit: 1g
	    volumes:
	      - esdata2:/usr/share/elasticsearch/data
	    networks:
	      - esnet
	
	volumes:
	  esdata1:
	    driver: local
	  esdata2:
	    driver: local
	


##

 
[terminal-2 1] sudo systemctl status docker

	[sudo] password: 
	● docker.service - Docker Application Container Engine
	   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
	   Active: active (running) since  5h 49min ago
	     Docs: https://docs.docker.com
	 Main PID: 879 (dockerd)
	   Memory: 129.0M
	      CPU: 15.694s
	   CGroup: /system.slice/docker.service
	           ├─  879 /usr/bin/dockerd -H fd://
	           ├─ 1012 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --shim docker-containerd-shim --metrics-interval=0 --...
	           ├─ 9908 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 9200 -container-ip 172.19.0.2 -container-port 9200
	           ├─ 9913 docker-containerd-shim 2fe7a65d24f9e815e3a3091473e0f1606cfce0f7a3a444376f1fbba24470400f /var/run/docker/libcontainerd/2fe7a65d24f9e815...
	           └─10098 docker-containerd-shim a022c8227f199d0e3b0f5811c05b2b113b978851d6d2305618415945f6e6290a /var/run/docker/libcontainerd/a022c8227f199d0e...


[terminal-2 2] sudo docker ps -a

	PORTS                              NAMES
	9200/tcp, 9300/tcp                 gitesdocker_elasticsearch2_1
	0.0.0.0:9200->9200/tcp, 9300/tcp   elasticsearch1

	The node elasticsearch1 listens on localhost:9200 while elasticsearch2 talks to elasticsearch1 over a Docker network


[terminal-2 3] curl -u elastic http://127.0.0.1:9200/_cat/health

	Enter host password for user 'elastic': changeme
	1495331007 01:43:27 docker-cluster green 2 2 10 5 0 0 0 0 - 100.0%

[terminal-2 4] curl -u elastic http://127.0.0.1:9200

	Enter host password for user 'elastic': changeme
	
	{
	  "name" : "RFtkjkh",
	  "cluster_name" : "docker-cluster",
	  "cluster_uuid" : "_9I2hNHkT-CvFAH2hofHVA",
	  "version" : {
	    "number" : "5.4.0",
	    "build_hash" : "780f8c4",
	    "build_date" : "T17:43:27.229Z",
	    "build_snapshot" : false,
	    "lucene_version" : "6.5.0"
	  },
	  "tagline" : "You Know, for Search"
	}
	
	
[terminal-2 5] curl -XPOST -u elastic http://127.0.0.1:9200/laraget/example?pretty -d '{"nubmers": "one two three"}'
	
	
	Enter host password for user 'elastic': changeme
	
	{
	  "_index" : "laraget",
	  "_type" : "example",
	  "_id" : "AVwpBySF9m6hUdM2gg1-",
	  "_version" : 1,
	  "result" : "created",
	  "_shards" : {
	    "total" : 2,
	    "successful" : 1,
	    "failed" : 0
	  },
	  "created" : true
	}
	
[terminal-2 6] curl -XPOST -u elastic http://127.0.0.1:9200/laraget/example/_search?pretty -d '{}'

	Enter host password for user 'elastic': changeme
	
	{
	  "took" : 56,
	  "timed_out" : false,
	  "_shards" : {
	    "total" : 5,
	    "successful" : 5,
	    "failed" : 0
	  },
	  "hits" : {
	    "total" : 1,
	    "max_score" : 1.0,
	    "hits" : [
	      {
	        "_index" : "laraget",
	        "_type" : "example",
	        "_id" : "AVwpBySF9m6hUdM2gg1-",
	        "_score" : 1.0,
	        "_source" : {
	          "nubmers" : "one two three"
	        }
	      }
	    ]
	  }
	}
	

[terminal-2 7] curl -XPUT -u elastic http://127.0.0.1:9200/_bulk --data-binary @shakespeare.json


[terminal-2 8] curl -XPOST -u elastic http://127.0.0.1:9200/shakespeare/act/_search?pretty -d '{}'

	Enter host password for user 'elastic': changeme
	
	{
	  "took" : 370,
	  "timed_out" : false,
	  "_shards" : {
	    "total" : 5,
	    "successful" : 5,
	    "failed" : 0
	  },
	  "hits" : {
	    "total" : 179,
	    "max_score" : 1.0,
	    "hits" : [
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "645",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 646,
	          "play_name" : "Henry IV",
	          "speech_number" : 54,
	          "line_number" : "",
	          "speaker" : "HOTSPUR",
	          "text_entry" : "ACT II"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "2227",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 2228,
	          "play_name" : "Henry IV",
	          "speech_number" : 81,
	          "line_number" : "",
	          "speaker" : "FALSTAFF",
	          "text_entry" : "ACT IV"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "16919",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 16920,
	          "play_name" : "As you like it",
	          "speech_number" : 37,
	          "line_number" : "",
	          "speaker" : "DUKE SENIOR",
	          "text_entry" : "ACT III"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "18565",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 18566,
	          "play_name" : "Antony and Cleopatra",
	          "speech_number" : 52,
	          "line_number" : "",
	          "speaker" : "ROSALIND",
	          "text_entry" : "ACT I"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "19181",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 19182,
	          "play_name" : "Antony and Cleopatra",
	          "speech_number" : 31,
	          "line_number" : "",
	          "speaker" : "CLEOPATRA",
	          "text_entry" : "ACT II"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "20110",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 20111,
	          "play_name" : "Antony and Cleopatra",
	          "speech_number" : 73,
	          "line_number" : "",
	          "speaker" : "MENAS",
	          "text_entry" : "ACT III"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "21066",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 21067,
	          "play_name" : "Antony and Cleopatra",
	          "speech_number" : 67,
	          "line_number" : "",
	          "speaker" : "DOMITIUS ENOBARBUS",
	          "text_entry" : "ACT IV"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "22427",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 22428,
	          "play_name" : "A Comedy of Errors",
	          "speech_number" : 132,
	          "line_number" : "",
	          "speaker" : "OCTAVIUS CAESAR",
	          "text_entry" : "ACT I"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "22712",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 22713,
	          "play_name" : "A Comedy of Errors",
	          "speech_number" : 24,
	          "line_number" : "",
	          "speaker" : "OF SYRACUSE",
	          "text_entry" : "ACT II"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "act",
	        "_id" : "33382",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 33383,
	          "play_name" : "Hamlet",
	          "speech_number" : 65,
	          "line_number" : "",
	          "speaker" : "HAMLET",
	          "text_entry" : "ACT II"
	        }
	      }
	    ]
	  }
	}
		
	
[terminal-2 9]  curl -XPOST -u elastic http://127.0.0.1:9200/shakespeare/line/_search?pretty -d '{}'

	Enter host password for user 'elastic': changeme

	{
	  "took" : 286,
	  "timed_out" : false,
	  "_shards" : {
	    "total" : 5,
	    "successful" : 5,
	    "failed" : 0
	  },
	  "hits" : {
	    "total" : 110486,
	    "max_score" : 1.0,
	    "hits" : [
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "14",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 15,
	          "play_name" : "Henry IV",
	          "speech_number" : 1,
	          "line_number" : "1.1.12",
	          "speaker" : "KING HENRY IV",
	          "text_entry" : "Did lately meet in the intestine shock"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "19",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 20,
	          "play_name" : "Henry IV",
	          "speech_number" : 1,
	          "line_number" : "1.1.17",
	          "speaker" : "KING HENRY IV",
	          "text_entry" : "The edge of war, like an ill-sheathed knife,"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "22",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 23,
	          "play_name" : "Henry IV",
	          "speech_number" : 1,
	          "line_number" : "1.1.20",
	          "speaker" : "KING HENRY IV",
	          "text_entry" : "Whose soldier now, under whose blessed cross"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "24",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 25,
	          "play_name" : "Henry IV",
	          "speech_number" : 1,
	          "line_number" : "1.1.22",
	          "speaker" : "KING HENRY IV",
	          "text_entry" : "Forthwith a power of English shall we levy;"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "25",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 26,
	          "play_name" : "Henry IV",
	          "speech_number" : 1,
	          "line_number" : "1.1.23",
	          "speaker" : "KING HENRY IV",
	          "text_entry" : "Whose arms were moulded in their mothers womb"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "26",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 27,
	          "play_name" : "Henry IV",
	          "speech_number" : 1,
	          "line_number" : "1.1.24",
	          "speaker" : "KING HENRY IV",
	          "text_entry" : "To chase these pagans in those holy fields"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "29",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 30,
	          "play_name" : "Henry IV",
	          "speech_number" : 1,
	          "line_number" : "1.1.27",
	          "speaker" : "KING HENRY IV",
	          "text_entry" : "For our advantage on the bitter cross."
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "40",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 41,
	          "play_name" : "Henry IV",
	          "speech_number" : 2,
	          "line_number" : "1.1.38",
	          "speaker" : "WESTMORELAND",
	          "text_entry" : "Whose worst was, that the noble Mortimer,"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "41",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 42,
	          "play_name" : "Henry IV",
	          "speech_number" : 2,
	          "line_number" : "1.1.39",
	          "speaker" : "WESTMORELAND",
	          "text_entry" : "Leading the men of Herefordshire to fight"
	        }
	      },
	      {
	        "_index" : "shakespeare",
	        "_type" : "line",
	        "_id" : "44",
	        "_score" : 1.0,
	        "_source" : {
	          "line_id" : 45,
	          "play_name" : "Henry IV",
	          "speech_number" : 2,
	          "line_number" : "1.1.42",
	          "speaker" : "WESTMORELAND",
	          "text_entry" : "A thousand of his people butchered;"
	        }
	      }
	    ]
	  }
	}
	
[terminal-2 10] curl -XPUT -u elastic http://127.0.0.1:9200/megacorp/employee/1 -d '{  "first_name" : "John",  "last_name" :  "Smith",  "age" :        25,     "about" :      "I love to go rock climbing",     "interests": [ "sports", "music" ] }'
	

	{"_index":"megacorp","_type":"employee","_id":"1","_version":2,"result":"updated","_shards":{"total":2,"successful":2,"failed":0},"created":false}

[terminal-2 11] curl -XPOST -u elastic http://127.0.0.1:9200/megacorp/employee/_search?pretty -d '{}'

	Enter host password for user 'elastic': changeme

	{
	  "took" : 3,
	  "timed_out" : false,
	  "_shards" : {
	    "total" : 5,
	    "successful" : 5,
	    "failed" : 0
	  },
	  "hits" : {
	    "total" : 1,
	    "max_score" : 1.0,
	    "hits" : [
	      {
	        "_index" : "megacorp",
	        "_type" : "employee",
	        "_id" : "1",
	        "_score" : 1.0,
	        "_source" : {
	          "first_name" : "John",
	          "last_name" : "Smith",
	          "age" : 25,
	          "about" : "I love to go rock climbing",
	          "interests" : [
	            "sports",
	            "music"
	          ]
	        }
	      }
	    ]
	  }
	}
	
[terminal-2 12] curl -XPUT -u elastic http://127.0.0.1:9200/megacorp/employee/2 -d '{ "first_name" :  "Jane",  "last_name" :   "Smith",     "age" :         32,     "about" :       "I like to collect rock albums",     "interests":  [ "music" ] }'

	Enter host password for user 'elastic': changeme

	{"_index":"megacorp","_type":"employee","_id":"2","_version":1,"result":"created","_shards":{"total":2,"successful":2,"failed":0},"created":true}
	

[terminal-2 13] curl -XPUT -u elastic http://127.0.0.1:9200/megacorp/employee/3 -d '{ "first_name" :  "Douglas",  "last_name" :   "Fir",     "age" :         35,     "about":        "I like to build cabinets",     "interests":  [ "forestry" ] }'

	Enter host password for user 'elastic': changeme

	{"_index":"megacorp","_type":"employee","_id":"3","_version":1,"result":"created","_shards":{"total":2,"successful":2,"failed":0},"created":true}


[terminal-2 14]  curl -XPOST -u elastic http://127.0.0.1:9200/megacorp/employee/_search?pretty -d '{}'

	Enter host password for user 'elastic': changeme

	{
	  "took" : 36,
	  "timed_out" : false,
	  "_shards" : {
	    "total" : 5,
	    "successful" : 5,
	    "failed" : 0
	  },
	  "hits" : {
	    "total" : 3,
	    "max_score" : 1.0,
	    "hits" : [
	      {
	        "_index" : "megacorp",
	        "_type" : "employee",
	        "_id" : "2",
	        "_score" : 1.0,
	        "_source" : {
	          "first_name" : "Jane",
	          "last_name" : "Smith",
	          "age" : 32,
	          "about" : "I like to collect rock albums",
	          "interests" : [
	            "music"
	          ]
	        }
	      },
	      {
	        "_index" : "megacorp",
	        "_type" : "employee",
	        "_id" : "1",
	        "_score" : 1.0,
	        "_source" : {
	          "first_name" : "John",
	          "last_name" : "Smith",
	          "age" : 25,
	          "about" : "I love to go rock climbing",
	          "interests" : [
	            "sports",
	            "music"
	          ]
	        }
	      },
	      {
	        "_index" : "megacorp",
	        "_type" : "employee",
	        "_id" : "3",
	        "_score" : 1.0,
	        "_source" : {
	          "first_name" : "Douglas",
	          "last_name" : "Fir",
	          "age" : 35,
	          "about" : "I like to build cabinets",
	          "interests" : [
	            "forestry"
	          ]
	        }
	      }
	    ]
	  }
	}
	

[terminal-2 15] curl -XGET -u elastic http://127.0.0.1:9200/megacorp/employee/1 -d '{}'

	Enter host password for user 'elastic': changeme

	{"_index":"megacorp","_type":"employee","_id":"1","_version":2,"found":true,"_source":{
	    "first_name" : "John",
	    "last_name" :  "Smith",
	    "age" :        25,
	    "about" :      "I love to go rock climbing",
	    "interests": [ "sports", "music" ]
	}}
	
[terminal-2 16]  curl -XGET -u elastic http://127.0.0.1:9200/megacorp/employee/_search -d '{}'

	Enter host password for user 'elastic': changeme

	{"took":1,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":3,"max_score":1.0,"hits":	[
	{"_index":"megacorp","_type":"employee","_id":"2","_score":1.0,"_source":{ "first_name" :  "Jane",  "last_name" :   "Smith",     "age" :         32,     "about" :       "I like to collect rock albums",     "interests":  [ "music" ] }},
	{"_index":"megacorp","_type":"employee","_id":"1","_score":1.0,"_source":{
	    "first_name" : "John",
	    "last_name" :  "Smith",
	    "age" :        25,
	    "about" :      "I love to go rock climbing",
	    "interests": [ "sports", "music" ]
		}},
	{"_index":"megacorp","_type":"employee","_id":"3","_score":1.0,"_source":{ "first_name" :  "Douglas",  "last_name" :   "Fir",     "age" :         35,     "about":        "I like to build cabinets",     "interests":  [ "forestry" ] }}]}}


[terminal-2 17] curl -XGET -u elastic http://127.0.0.1:9200/megacorp/employee/_search?q=last_name:Smith -d '{}'

	Enter host password for user 'elastic': changeme

	{"took":55,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":2,"max_score":0.2876821,"hits":[{"_index":"megacorp","_type":"employee","_id":"2","_score":0.2876821,"_source":{ "first_name" :  "Jane",  "last_name" :   "Smith",     "age" :         32,     "about" :       "I like to collect rock albums",     "interests":  [ "music" ] }},{"_index":"megacorp","_type":"employee","_id":"1","_score":0.2876821,"_source":{
	    "first_name" : "John",
	    "last_name" :  "Smith",
	    "age" :        25,
	    "about" :      "I love to go rock climbing",
	    "interests": [ "sports", "music" ]
	}}]}}


[terminal-2 18] curl -XGET -u elastic http://127.0.0.1:9200/megacorp/employee/_search -d '{
	    "query" : {
	        "match" : {
	            "last_name" : "Smith"
	        }
	    }
	}'

	Enter host password for user 'elastic': changeme

	{"took":28,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":2,"max_score":0.2876821,"hits":[{"_index":"megacorp","_type":"employee","_id":"2","_score":0.2876821,"_source":{ "first_name" :  "Jane",  "last_name" :   "Smith",     "age" :         32,     "about" :       "I like to collect rock albums",     "interests":  [ "music" ] }},{"_index":"megacorp","_type":"employee","_id":"1","_score":0.2876821,"_source":{
	    "first_name" : "John",
	    "last_name" :  "Smith",
	    "age" :        25,
	    "about" :      "I love to go rock climbing",
	    "interests": [ "sports", "music" ]
	}}]}}
	
	
[terminal-2 19] curl -XGET -u elastic http://127.0.0.1:9200/megacorp/employee/_search -d '{
	    "query" : {
	        "bool" : {
	            "must" : {
	                "match" : {
	                    "last_name" : "smith" 
	                }
	            },
	            "filter" : {
	                "range" : {
	                    "age" : { "gt" : 30 } 
	                }
	            }
	        }
	    }
	}' 
	
	Enter host password for user 'elastic': changeme

	{"took":93,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":1,"max_score":0.2876821,"hits":[{"_index":"megacorp","_type":"employee","_id":"2","_score":0.2876821,"_source":{ "first_name" :  "Jane",  "last_name" :   "Smith",     "age" :         32,     "about" :       "I like to collect rock albums",     "interests":  [ "music" ] }}]}}

[terminal-2 20] curl -XGET -u elastic http://127.0.0.1:9200/megacorp/employee/_search -d '{
	    "query" : {
	        "match" : {
	            "about" : "rock climbing"
	        }
	    }
	}'

	Enter host password for user 'elastic': changeme

	{"took":5,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":	{"total":2,"max_score":0.53484553,"hits":[{"_index":"megacorp","_type":"employee","_id":"1","_score":0.53484553,"_source":{
 	   "first_name" : "John",
	    "last_name" :  "Smith",
	    "age" :        25,
	    "about" :      "I love to go rock climbing",
	    "interests": [ "sports", "music" ]
	}},{"_index":"megacorp","_type":"employee","_id":"2","_score":0.26742277,"_source":{ "first_name" :  "Jane",  "last_name" :   "Smith",     "age" :         32,     "about" :       "I like to collect rock albums",     "interests":  [ "music" ] }}]}}

	
[terminal-2 21] curl -XGET -u elastic http://127.0.0.1:9200/megacorp/employee/_search -d '{
	    "query" : {
	        "match_phrase" : {
	            "about" : "rock climbing"
	        }
	    }
	}'
	
	Enter host password for user 'elastic':
	{"took":96,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":	{"total":1,"max_score":0.53484553,"hits":[{"_index":"megacorp","_type":"employee","_id":"1","_score":0.53484553,"_source":{
	    "first_name" : "John",
	    "last_name" :  "Smith",
	    "age" :        25,
	    "about" :      "I love to go rock climbing",
	    "interests": [ "sports", "music" ]
	}}]}}
	
	
[] curl -XGET -u elastic http://127.0.0.1:9200/megacorp/employee/_search -d '{
	    "query" : {
	        "match_phrase" : {
	            "about" : "rock climbing"
	        }
	    },
	    "highlight": {
	        "fields" : {
	            "about" : {}
	        }
	    }
	}'

	Enter host password for user 'elastic': changeme

	{"took":318,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":1,"max_score":0.53484553,"hits":[{"_index":"megacorp","_type":"employee","_id":"1","_score":0.53484553,"_source":{
	    "first_name" : "John",
	    "last_name" :  "Smith",
	    "age" :        25,
	    "about" :      "I love to go rock climbing",
	    "interests": [ "sports", "music" ]
	},"highlight":{"about":["I love to go <em>rock</em> <em>climbing</em>"]}}]}}
	
	
		
[terminal-2 7] sudo docker-compose down

##


