---
layout: post
title: Elasticsearch优化
category: blog
description: Elasticsearch优化
---

## 索引结构的优化
### 设计
1. 索引变更
	
	之前主站索引结构是在index为kuaizhan索引下有五个type: site、page、post、category、`library_post_v1`、`library_site_v1`，缺点是单个index体积过大（size: 265Gi (531Gi)），且无法使用别名，不利于索引重建。

	所以我把site、page、post、category、`library_post_v1`、`library_site_v1`都设为单独的index，并使用后缀，全名为：`index_version`（比如`post_v1`)。
	
	使用别名，比如索引`post_v1`的别名为`post`，在程序中访问使用别名`post`，即使索引重建`post_v1`为`post_v2`，此时程序无需改动，将别名`post`指向`post_v2`即可。

	```
	/kuaizhan/site => /site_v1/site
	/kuaizhan/page => /page_v1/page
	/kuaizhan/post => /post_v1/post
	/kuaizhan/category => /category_v1/category
	/kuaizhan/library_post_v1 => /library_post_v1/library_post	/kuaizhan/library_site_v2 => /library_site_v1/library_site	```

### mapping设置
1. post

	```
	/kuaizhan/post
	{
	   "kuaizhan": {
	      "mappings": {
	         "post": {
	            "properties": {
	               "content": {
	                  "type": "string"
	               },
	               "desc": {
	                  "type": "string"
	               },
	               "is_published": {
	                  "type": "boolean"
	               },
	               "set_time": {
	                  "type": "long"
	               },
	               "site_id": {
	                  "type": "long"
	               },
	               "title": {
	                  "type": "string"
	               }
	            }
	         }
	      }
	   }
	}
	=>
	PUT /post_v1
	{
	   "mappings": {
	      "post": {
	         "dynamic": "strict",
	         "properties": {
	            "content": {
	               "type": "string"
	            },
	            "desc": {
	               "type": "string"
	            },
	            "is_published": {
	               "type": "boolean"
	            },
	            "set_time": {
	               "type": "long"
	            },
	            "site_id": {
	               "type": "long"
	            },
	            "title": {
	               "type": "string"
	            }
	         }
	      }
	   }
	}
	```

2. `library_post_v1`

	```
	/kuaizhan/library_post_v1
	{
	   "kuaizhan": {
	      "mappings": {
	         "library_post_v1": {
	            "properties": {
	               "area": {
	                  "type": "string"
	               },
	               "channel": {
	                  "type": "string"
	               },
	               "from": {
	                  "type": "long"
	               },
	               "heat": {
	                  "type": "string"
	               },
	               "kz_grade": {
	                  "type": "double"
	               },
	               "label": {
	                  "type": "string"
	               },
	               "level": {
	                  "type": "string"
	               },
	               "mp_category": {
	                  "type": "string"
	               },
	               "pic_id": {
	                  "type": "long"
	               },
	               "post_title": {
	                  "type": "string"
	               },
	               "query": {
	                  "properties": {
	                     "bool": {
	                        "properties": {
	                           "must": {
	                              "properties": {
	                                 "term": {
	                                    "properties": {
	                                       "level": {
	                                          "type": "string"
	                                       }
	                                    }
	                                 }
	                              }
	                           }
	                        }
	                     }
	                  }
	               },
	               "score": {
	                  "type": "long"
	               },
	               "set_time": {
	                  "type": "long"
	               },
	               "site_id": {
	                  "type": "long"
	               },
	               "site_name": {
	                  "type": "string"
	               },
	               "size": {
	                  "type": "long"
	               },
	               "sort": {
	                  "properties": {
	                     "set_time": {
	                        "type": "string"
	                     }
	                  }
	               }
	            }
	         }
	      }
	   }
	}
	=>
	PUT /library_post_v1
	{
	   "mappings": {
	      "library_post": {
	         "dynamic": "strict",
	         "properties": {
	            "post_title": {
	               "type": "string"
	            },
	            "site_name": {
	               "type": "string"
	            },
	            "site_id": {
	               "type": "long"
	            },
	            "area": {
	               "type": "string"
	            },
	            "channel": {
	               "type": "string"
	            },
	            "label": {
	               "type": "string"
	            },
	            "mp_category": {
	               "type": "string"
	            },
	            "pic_id": {
	               "type": "long"
	            },
	            "level": {
	               "type": "string"
	            },
	            "heat": {
	               "type": "string"
	            },
	            "kz_grade": {
	               "type": "double"
	            },
	            "set_time": {
	               "type": "long"
	            }
	         }
	      }
	   }
	}
	```

3. category

	```
	/kuaizhan/category
	{
		"kuaizhan": {
			"mappings": {
				"category": {
					"properties": {
						"create_time":{ 
							"type":"long"
						},
						"name": {
							"type":"string"
						},
						"site_id": {
							"type":"long"
						}
					}
				}
			}
		}
	}
	=>
	PUT /category_v1
	{
	   "mappings": {
	      "category": {
	         "dynamic": "strict",
	         "properties": {
	            "create_time": {
	               "type": "long"
	            },
	            "name": {
	               "type": "string"
	            },
	            "site_id": {
	               "type": "long"
	            }
	         }
	      }
	   }
	}
	```

4. `library_site_v2`

	```
	/kuaizhan/library_site_v2
	{
		"kuaizhan": {
			"mappings": {
				"library_site_v2": {
					"properties": {
						"channel": {
							"type":"string"
						},
						"desc": {
							"type":"string"
						},
						"grade": {
							"type":"double"
						},
						"name": {
							"type":"string"
						}
					}
				}
			}
		}
	}
	=>
	PUT /library_site_v1
	{
	   "mappings": {
	      "library_site": {
	         "dynamic": "strict",
	         "properties": {
	            "channel": {
	               "type": "string"
	            },
	            "desc": {
	               "type": "string"
	            },
	            "grade": {
	               "type": "double"
	            },
	            "name": {
	               "type": "string"
	            }
	         }
	      }
	   }
	}
	```

5. page

	```
	/kuaizhan/page
	{
	   "kuaizhan": {
	      "mappings": {
	         "page": {
	            "properties": {
	               "is_published": {
	                  "type": "boolean"
	               },
	               "settings": {
	                  "properties": {
	                     "analysis": {
	                        "properties": {
	                           "analyzer": {
	                              "properties": {
	                                 "my_ngram_analyzer": {
	                                    "properties": {
	                                       "tokenizer": {
	                                          "type": "string"
	                                       }
	                                    }
	                                 }
	                              }
	                           },
	                           "tokenizer": {
	                              "properties": {
	                                 "my_ngram_tokenizer": {
	                                    "properties": {
	                                       "max_gram": {
	                                          "type": "string"
	                                       },
	                                       "min_gram": {
	                                          "type": "string"
	                                       },
	                                       "token_chars": {
	                                          "type": "string"
	                                       },
	                                       "type": {
	                                          "type": "string"
	                                       }
	                                    }
	                                 }
	                              }
	                           }
	                        }
	                     }
	                  }
	               },
	               "site_id": {
	                  "type": "long"
	               },
	               "title": {
	                  "type": "string"
	               }
	            }
	         }
	      }
	   }
	}
	=>
	PUT /page_v1
	{
	   "mappings": {
	      "page": {
	         "dynamic": "strict",
	         "properties": {
	            "is_published": {
	               "type": "boolean"
	            },
	            "site_id": {
	               "type": "long"
	            },
	            "title": {
	               "type": "string"
	            }
	         }
	      }
	   }
	}
	```

6. site
	
	```
	/kuaizhan/site
	{
	   "kuaizhan": {
	      "mappings": {
	         "site": {
	            "properties": {
	               "domain": {
	                  "type": "string"
	               },
	               "name": {
	                  "type": "string"
	               },
	               "seo_description": {
	                  "type": "string"
	               },
	               "seo_keyword": {
	                  "type": "string"
	               },
	               "site_id": {
	                  "type": "long"
	               },
	               "user_id": {
	                  "type": "long"
	               }
	            }
	         }
	      }
	   }
	}
	=>
	PUT /site_v1
	{
	   "mappings": {
	      "site": {
	         "dynamic": "strict",
	         "properties": {
	            "domain": {
	               "type": "string"
	            },
	            "name": {
	               "type": "string"
	            },
	            "site_id": {
	               "type": "long"
	            },
	            "user_id": {
	               "type": "long"
	            }
	         }
	      }
	   }
	}
	```
	
在更新es时，我们并没有对更新的字段进行检查，而默认情况下es的字段是可以动态添加的，因此可以看到原来索引的mapping被一些错误的操作改得面目全非，对es的效率产生影响。

在设置新索引的mapping时，把dynamic设置为strict，当es检测到新的字段时会报错并拒绝更新文档。

	The dynamic setting controls whether new fields can be added dynamically or not. It accepts three settings:
	true: Newly detected fields are added to the mapping. (default)
	false: Newly detected fields are ignored. New fields must be added explicitly.
	strict: If new fields are detected, an exception is thrown and the document is rejected.
	
那么，当我们需要增加新字段时应该怎么办呢？

	PUT /site_v1/site/_mapping
	{
	   "site": {
	      "properties": {
	         //your new mapping properties
	      }
	   }
	}

### 设置别名
![](/images/posts/2017-07-06-Elasticsearch优化/alias.png)

```
POST /_aliases
{
   "actions": [
      {
         "add": {
            "index": "post_v1",
            "alias": "post"
         }
      },
      {
         "add": {
            "index": "library_post_v1",
            "alias": "library_post"
         }
      },
      {
         "add": {
            "index": "library_site_v1",
            "alias": "library_site"
         }
      },
      {
         "add": {
            "index": "category_v1",
            "alias": "category"
         }
      },
      {
         "add": {
            "index": "page_v1",
            "alias": "page"
         }
      },
      {
         "add": {
            "index": "site_v1",
            "alias": "site"
         }
      }
   ]
}
```

## 索引更新优化

### 更新频率
如果把ES看做另一个数据库，那么它总是会比系统原有的数据库滞后，因为数据会先存入原有数据库，再同步到ES。那么滞后的时间就是一个敏感的参数。根据业务的不同，差别很大。一般而言，用户提交或修改数据后，是希望能立刻查询到修改的结果的，所以理论上是越短越好，但频繁的更新会给ES服务器带来很大的开销。

### 同步VS异步
更新可以采用同步和异步两种方式。

>同步： 直接向es发送请求。

>异步： 将es的更新放入队列。

同步的缺点是显而易见的，因为需要进行一次http请求与外部的服务器相连，影响原有的操作效率，甚至会导致timeout异常。因此在产品环境中一定要使用异步更新，它不仅规避了效率和稳定性问题，也让我们有了更大的灵活性在更新之前做更多的数据处理工作，例如后面要提到的bulk操作。

### 使用Bulk
针对每一条数据的增删改就请求一次es，对我们的应用程序和es服务器来说，都是很低效的。幸运的是，es提供了bulk api，并且也推荐使用它来进行批量的更新。一个简单的批量的请求体如下，它可以同时包含增删改操作，并且可以是多条。
	
	{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 
	{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
	{ "title":    "My first blog post" }
	{ "index":  { "_index": "website", "_type": "blog" }}
	{ "title":    "My second blog post" }
	{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
	{ "doc" : {"title" : "My updated blog post"} }

我的更新策略是：在KZSearch中封装4个函数（create,index,update,delete），他们都会调用一个核心的函数update_es，这个函数将数据放到redis list中，从而实现异步更新es。

函数会根据更新文档的id将数据放到不同的list中，这样做主要是分流，使单个list的数据不会过大，同时也保证了异步更新时对同一文档多次操作的时序性。

	  /**
     * 更新es核心函数, 先写到redis(list)中，通过脚本run-update-es-task使用bulk请求批量写入es
     * @param type $action
     * @param type $type
     * @return type
     */
    public static function update_es($type, $id, $action, $doc = array(), $index = SEARCH_SERVICE_INDICES) {
        global $_redis;
        $arr = json_encode(array(
            'action' => $action,
            'index' => $index,
            'type' => $type,
            'id' => $id,
            'doc' => $doc
        ));
        $key = crc32($id) % 10;
        $redis_key = 'kz-es-bulk-list-' . $key;
        $_redis->lpush($redis_key, $arr);
    }

我们需要一些程序来消费掉list中的数据，即run-update-es-task.php脚本，在预发机器上使用supervisor来运行这个脚本，脚本将根据传入的参数去处理相应list中的数据：

supervisor部分配置：

	[program:run-update-es-task-0]
	command=/opt/kuaizhan/bin/php /opt/web/pre/kuaizhan_cl/search/cli/run-update-es-task.php 0
	numprocs=1
	directory=/home/nobody/
	logfile=/opt/logs/run-update-es-task-0.log
	autostart=true
	autorestart=true
	user=nobody
	stopsignal=KILL

run-update-es-task.php脚本：

	<?php

	/**
	 * @description 更新es任务
	 * @url         /
	 * @history     2017-04-11(liusheng)
	 *
	 */
	/**[S1]**/
	require_once(dirname(__FILE__) . "/../../a-head/head-nosession.php");
	/**[E1]**/
	
	/**[S2]**/
	if ($argc < 2) {
	    echo "missing required arguments.\n";
	    exit;
	}
	/**[E2]**/
	
	/**[S3]**/
	$key = $argv[1];
	$redis_key = 'kz-es-bulk-list-' . $key;
	$perpage = 100;
	$list = array();
	
	function bulk() {
	    global $_raven_client, $list;
	    if (empty($list)) {
	        return;
	    }
	
	    foreach ($list as $li_str) {
	        // $key_array[] = $key;
	        $li = json_decode($li_str, true);
	        $action = $li['action'];
	        $index = $li['index'];
	        $type = $li['type'];
	        $id = $li['id'];
	        $doc = $li['doc'];
	        $arr = array(
	            '_index' => $index,
	            '_type' => $type,
	            '_id' => $id
	        );
	        if ($action === 'update') {
	            $arr['_retry_on_conflict'] = 3;
	        }
	
	        $action_part = json_encode(array($action => $arr));
	        $body_parts[] = addcslashes($action_part, "\n");
	        if (!empty($doc)) {
	            $body_parts[] = addcslashes(json_encode($doc), "\n");
	        }
	    }
	
	    $postdata = implode("\n", $body_parts) . "\n";
	    $host = SEARCH_SERVICE_HOST;
	    $url = "http://" . SEARCH_SERVICE_IP . "/_bulk";
	    $return = BHttp::do_post($url, $postdata, $host, 60, array(
	        "Content-Type: application/json"
	    ));
	    $body = json_decode($return, true);
	    if (!empty($body['errors'])) {
	        $error = "";
	        foreach ($body['items'] as $item) {
	            $key = key($item);
	            $status = $item[$key]['status'];
	            if ($status == 404) {
	                continue;
	            } else {
	                $error .= json_encode($item);
	            }
	        }
	        if (!empty($error)) {
	            $_raven_client->captureMessage("search bulk api invoke failed. [url={$url}, host={$host}, error={$error}]", array(), "info", true, $body);
	        }
	    }
	    $list = array();
	}
	
	function sig_handler($signo) {
	    switch ($signo) {
	        //killproc函数先发SIGTERM信号，再发SIGHUP信号，最后发SIGKILL信号（SIGKILL不能被捕获）
	        case SIGTERM:
	        case SIGHUP:
	        //The SIGINT signal is sent to a process by its controlling terminal when a user wishes to interrupt the process.
	        case SIGINT:
	            bulk();
	            echo "Received signal $signo: exit.\n";
	            exit(1);
	            break;
	        default:
	            break;
	    }
	}
	//安装信号触发器器
	pcntl_signal(SIGTERM, "sig_handler");
	pcntl_signal(SIGHUP, "sig_handler");
	pcntl_signal(SIGINT, "sig_handler");
	/**[E3]**/
	
	/**[S4]**/
	while (true) {
	
	    for ($i = 0; $i < $perpage; $i++) {
	        $li = $_redis->rPop($redis_key);
	        if (empty($li)) {
	            if ($i == 0) {
	                //如果当前redis list为空，休息一秒
	                sleep(1);
	            }
	            //如果当前redis list没有更多数据，直接批量发送
	            break 1;
	        }
	        $list[] = $li;
	    }
	
	    bulk();
	
	    //接收到信号时，调用注册的sig_handler
	    pcntl_signal_dispatch();
	}
	/**[E4]**/
	
	/**[S5]**/
	/**[E5]**/

### 队列监控
为了及时掌握plf-kz-es-bulk-list-0~9这10个list的数据情况，防止出现list中数据堆积问题，我使用crontab每5分钟运行一次脚本check-es-redis-list.php，检查10个list的数据量，若超过一定阈值，则发出报警邮件和短信。

另外，开发了后台一个后台工具，用于展现list的实时数据，访问http://www.kuaizhan.sohuno.com/dev/batch-monitor。

### 错误处理
脚本中对list使用的是pop操作，将redis中对应数据删除了。但是如果因为某种原因出错，则还需要将这条数据重新加入队列中，以此来实现重试操作。 需要注意的是，批量操作时，ES会将所有的数据更新状态都返回，系统需要根据是否出现error来从返回的结果中提取出错的数据，仅仅将这些数据重新加入队列，而不要简单的将所有数据都重新加回，增加负载。

这一部分暂未实现，需要先观察错误信息。

## 上线步骤
1. 设置6个索引的mapping和alias
2. 代码上预发，测试es增删查改
3. 试跑索引重建脚本，观察线上服务是否稳定，并预计脚本运行时间
4. 代码上线，正式开始跑脚本
5. 脚本跑完后，检查数据一致性
6. 删除冗余更新代码，将查询es的索引从kuaizhan切换到新建好的索引

## 参考
### 工具
使用elasticsearch的chrome插件，非常方便

![](/images/posts/2017-07-06-Elasticsearch优化/es-chrome-plugin.png)

### Bulk相关
1. [Elasticsearch--更新策略](http://jiangpeng.info/blogs/2014/11/25/elasticsearch-update-strategy.html)
2. [Keeping Elasticsearch in Sync](https://www.elastic.co/blog/found-keeping-elasticsearch-in-sync)
3. [docs-bulk.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.1/docs-bulk.html)
4. [dynamic.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.1/dynamic.html)
5. [Elasticsearch: updating the mappings and settings of an existing index](https://gist.github.com/nicolashery/6317643)

### Elasticsearch升级5.0相关
1. [What's new in Elasticsearch 5.0](https://writequit.org/org/es/presentations/whats-new-elasticsearch-5.0.html)
2. [Upgrading Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html)
3. [Elastic Stack 5.0升级踩坑记](https://zhuanlan.zhihu.com/p/23271404)