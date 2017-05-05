# Jenkins [![Build Status](https://secure.travis-ci.org/silas/node-jenkins.png?branch=master)](http://travis-ci.org/silas/node-jenkins)

这是一个 Node [Jenkins](http://jenkins-ci.org/) 客户端.

## 文档 TOC

 * jenkins: [初始化](#init), [信息](#info)
 * build: [get](#build-get), [log](#build-log), [logStream](#build-log-stream), [stop](#build-stop)
 * job: [build](#job-build), [get config](#job-config-get), [set config](#job-config-set), [copy](#job-config-copy), [create](#job-create), [destroy](#job-destroy), [disable](#job-disable), [enable](#job-enable), [exists](#job-exists), [get](#job-get), [list](#job-list)
 * node: [get config](#node-config-get), [create](#node-create), [destroy](#node-destroy), [disconnect](#node-disconnect), [disable](#node-disable), [enable](#node-enable), [exists](#node-exists), [get](#node-get), [list](#node-list)
 * queue: [list](#queue-list), [item](#queue-item), [cancel](#queue-cancel)
 * view: [get config](#view-config-get), [set config](#view-config-set), [create](#view-create), [destroy](#view-destroy), [exists](#view-exists), [get](#view-get), [list](#view-list), [add job](#view-add), [remove job](#view-remove)

<a name="promise"></a>
### Promise

在 Node 高版本中可以通过设置 `promisify` 为 `true` 打开支持，更老版本可以通过包装器(例如 `bluebird.fromCallback`)完成

<a name="common-options"></a>
### 通用选项

这些选项会在每次调用时生效，但并非所有 Jenkins 服务端都支持

 * depth (Number, default: 0): 返回数据量控制 (参考 [设置 depth 参数](https://wiki.jenkins-ci.org/display/JENKINS/Remote+access+API#RemoteaccessAPI-Depthcontrol))
 * tree (String, optional): 路径描述符 (参考 Jenkins 官方 API )

<a name="init"></a>
### jenkins([options])

初始化一个 Jenkins 客户端

选项

 * baseUrl (String): Jenkins 服务器 URL
 * crumbIssuer (Boolean, default: false): 开启 CSRF 保护
 * headers (Object, optional): 每次请求头部的设定
 * promisify (Boolean|Function, optional): 将回调转为 Promise

用法

``` javascript
var jenkins = require('jenkins')({ baseUrl: 'http://user:pass@localhost:8080', crumbIssuer: true });
```

<a name="info"></a>
### jenkins.info(callback)

获取 Jenkins 服务端信息

用法

``` javascript
jenkins.info(function(err, data) {
  if (err) throw err;

  console.log('info', data);
});
```

样例输出

``` json
{
  "assignedLabels": [
    {}
  ],
  "description": null,
  "jobs": [
    {
      "color": "blue",
      "name": "example",
      "url": "http://localhost:8080/job/example/"
    }
  ],
  "mode": "NORMAL",
  "nodeDescription": "the master Jenkins node",
  "nodeName": "",
  "numExecutors": 2,
  "overallLoad": {},
  "primaryView": {
    "name": "All",
    "url": "http://localhost:8080/"
  },
  "quietingDown": false,
  "slaveAgentPort": 12345,
  "unlabeledLoad": {},
  "useCrumbs": false,
  "useSecurity": false,
  "views": [
    {
      "name": "All",
      "url": "http://localhost:8080/"
    }
  ]
}
```

<a name="build-get"></a>
### jenkins.build.get(options, callback)

获取构建信息

传参

 * name (String): 任务名称
 * number (Integer): 构建编号

用法

``` javascript
jenkins.build.get('example', 1, function(err, data) {
  if (err) throw err;

  console.log('build', data);
});
```

样例输出

``` json
{
  "actions": [],
  "buildable": true,
  "builds": [
    {
      "number": 1,
      "url": "http://localhost:8080/job/example/1/"
    }
  ],
  "color": "blue",
  "concurrentBuild": false,
  "description": "",
  "displayName": "example",
  "displayNameOrNull": null,
  "downstreamProjects": [],
  "firstBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "healthReport": [
    {
      "description": "Build stability: No recent builds failed.",
      "iconUrl": "health-80plus.png",
      "score": 100
    }
  ],
  "inQueue": false,
  "keepDependencies": false,
  "lastBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "lastCompletedBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "lastFailedBuild": null,
  "lastStableBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "lastSuccessfulBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "lastUnstableBuild": null,
  "lastUnsuccessfulBuild": null,
  "name": "example",
  "nextBuildNumber": 2,
  "property": [],
  "queueItem": null,
  "scm": {},
  "upstreamProjects": [],
  "url": "http://localhost:8080/job/example/"
}
```

<a name="build-log"></a>
### jenkins.build.log(options, callback)

获取构建日志

传参

* name (String): 任务名称
* number (Integer): 构建编号
* start (Integer, 可选): start offset
* type (String, enum: text, html, 默认: text): 指定输出类型
* meta (Boolean, default: false): return object with text (log data), more (boolean if there is more log data), and size (used with start to offset on subsequent calls)

用法

``` javascript
jenkins.build.log('example', 1, function(err, data) {
  if (err) throw err;

  console.log('log', data);
});
```

<a name="build-log-stream"></a>
### jenkins.build.logStream(options, callback)

获取构建日志流

传参

* name (String): 任务名称
* number (Integer): 构建编号
* type (String, enum: text, html, default: text): 指定输出类型
* delay (Integer, default: 1000): poll interval in milliseconds

用法

``` javascript
var log = jenkins.build.logStream('example', 1);

log.on('data', function(text) {
  process.stdout.write(text);
});

log.on('error', function(err) {
  console.log('error', err);
});

log.on('end', function() {
  console.log('end');
});
```

<a name="build-stop"></a>
### jenkins.build.stop(options, callback)

中断构建

传参

 * name (String): 任务名称
 * number (Integer): 构建编号

用法

``` javascript
jenkins.build.stop('example', 1, function(err) {
  if (err) throw err;
});
```

<a name="job-build"></a>
### jenkins.job.build(options, callback)

开始构建

传参

 * name (String): 任务名称
 * parameters (Object, 可选): 构建参数
 * token (String, 可选): 鉴权标识

用法

``` javascript
jenkins.job.build('example', function(err, data) {
  if (err) throw err;

  console.log('queue item number', data);
});
```

``` javascript
jenkins.job.build({ name: 'example', parameters: { name: 'value' } }, function(err) {
  if (err) throw err;
});
```

<a name="job-config-get"></a>
### jenkins.job.config(options, callback)

获取任务配置（XML形式）

传参

 * name (String): 任务名称

用法

``` javascript
jenkins.job.config('example', function(err, data) {
  if (err) throw err;

  console.log('xml', data);
});
```

<a name="job-config-set"></a>
### jenkins.job.config(options, callback)

修改任务配置（XML形式）

传参

 * name (String): 任务名称
 * xml (String): 配置文件

用法

``` javascript
jenkins.job.config('example', xml, function(err) {
  if (err) throw err;
});
```

<a name="job-config-copy"></a>
### jenkins.job.copy(options, callback)

从已有的项目中克隆新的任务

传参

 * name (String): 新任务名称
 * from (String): 源任务名称

用法

``` javascript
jenkins.job.copy('fromJob', 'example', function(err) {
  if (err) throw err;
});
```

<a name="job-create"></a>
### jenkins.job.create(options, callback)

从配置文件创建新的任务

传参

 * name (String): 任务名称
 * xml (String): 配置文件

用法

``` javascript
jenkins.job.create('example', xml, function(err) {
  if (err) throw err;
});
```

<a name="job-destroy"></a>
### jenkins.job.destroy(options, callback)

删除任务

传参

 * name (String): 任务名称

用法

``` javascript
jenkins.job.destroy('example', function(err) {
  if (err) throw err;
});
```

<a name="job-disable"></a>
### jenkins.job.disable(options, callback)

停用一个任务

传参

 * name (String): 任务名称

用法

``` javascript
jenkins.job.disable('example', function(err) {
  if (err) throw err;
});
```

<a name="job-enable"></a>
### jenkins.job.enable(options, callback)

启动一个任务

传参

 * name (String): 任务名称

用法

``` javascript
jenkins.job.enable('example', function(err) {
  if (err) throw err;
});
```

<a name="job-exists"></a>
### jenkins.job.exists(options, callback)

监测任务是否存在

传参

 * name (String): 任务名称

用法

``` javascript
jenkins.job.exists('example', function(err, exists) {
  if (err) throw err;

  console.log('exists', exists);
});
```

<a name="job-get"></a>
### jenkins.job.get(options, callback)

获取任务信息，得到 json 格式配置文件

传参

 * name (String): 任务名称

用法

``` javascript
jenkins.job.get('example', function(err, data) {
  if (err) throw err;

  console.log('job', data);
});
```

样例输出

``` json
{
  "actions": [],
  "buildable": true,
  "builds": [
    {
      "number": 1,
      "url": "http://localhost:8080/job/example/1/"
    }
  ],
  "color": "blue",
  "concurrentBuild": false,
  "description": "",
  "displayName": "example",
  "displayNameOrNull": null,
  "downstreamProjects": [],
  "firstBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "healthReport": [
    {
      "description": "Build stability: No recent builds failed.",
      "iconUrl": "health-80plus.png",
      "score": 100
    }
  ],
  "inQueue": false,
  "keepDependencies": false,
  "lastBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "lastCompletedBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "lastFailedBuild": null,
  "lastStableBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "lastSuccessfulBuild": {
    "number": 1,
    "url": "http://localhost:8080/job/example/1/"
  },
  "lastUnstableBuild": null,
  "lastUnsuccessfulBuild": null,
  "name": "example",
  "nextBuildNumber": 2,
  "property": [],
  "queueItem": null,
  "scm": {},
  "upstreamProjects": [],
  "url": "http://localhost:8080/job/example/"
}
```

<a name="job-list"></a>
### jenkins.job.list(callback)

列出所有的任务

用法

``` javascript
jenkins.job.list(function(err, data) {
  if (err) throw err;

  console.log('jobs', data);
});
```

样例输出

``` json
[
  {
    "color": "blue",
    "name": "example",
    "url": "http://localhost:8080/job/example/"
  }
]
```

<a name="node-config-get"></a>
### jenkins.node.config(options, callback)

获取节点配置信息（XML 形式）

传参

 * name (String): 节点名称

用法

``` javascript
jenkins.node.config('example', function(err, data) {
  if (err) throw err;

  console.log('xml', data);
});
```

<a name="node-create"></a>
### jenkins.node.create(options, callback)

创建一个新的节点

传参

 * name (String): 节点名称

用法

``` javascript
jenkins.node.create('slave', function(err) {
  if (err) throw err;
});
```

<a name="node-destroy"></a>
### jenkins.node.destroy(options, callback)

删除节点

传参

 * name (String): 节点名称

用法

``` javascript
jenkins.node.destroy('slave', function(err) {
  if (err) throw err;
});
```

<a name="node-disconnect"></a>
### jenkins.node.disconnect(options, callback)

断开与指定节点的链接

传参

 * name (String): 节点名称
 * message (String, optional): reason for being disconnected

用法

``` javascript
jenkins.node.disconnect('slave', 'no longer used', function(err) {
  if (err) throw err;
});
```

<a name="node-disable"></a>
### jenkins.node.disable(options, callback)

停用一个节点

传参

 * name (String): 节点名称
 * message (String, optional): reason for being disabled

用法

``` javascript
jenkins.node.disable('slave', 'network failure', function(err) {
  if (err) throw err;
});
```

<a name="node-enable"></a>
### jenkins.node.enable(options, callback)

启用一个节点

传参

 * name (String): 节点名称

用法

``` javascript
jenkins.node.enable('slave', function(err) {
  if (err) throw err;
});
```

<a name="node-exists"></a>
### jenkins.node.exists(options, callback)

检查节点是否存在

传参

 * name (String): 节点名称

用法

``` javascript
jenkins.node.exists('slave', function(err, exists) {
  if (err) throw err;

  console.log('exists', exists);
});
```

<a name="node-get"></a>
### jenkins.node.get(options, callback)

获取节点信息

传参

 * name (String): 节点名称

用法

``` javascript
jenkins.node.get('slave', function(err, data) {
  if (err) throw err;

  console.log('node', data);
});
```

样例输出

``` json
{
  "actions": [],
  "displayName": "slave",
  "executors": [
    {},
    {}
  ],
  "icon": "computer-x.png",
  "idle": true,
  "jnlpAgent": true,
  "launchSupported": false,
  "loadStatistics": {},
  "manualLaunchAllowed": true,
  "monitorData": {
    "hudson.node_monitors.ArchitectureMonitor": null,
    "hudson.node_monitors.ClockMonitor": null,
    "hudson.node_monitors.DiskSpaceMonitor": null,
    "hudson.node_monitors.ResponseTimeMonitor": {
      "average": 5000
    },
    "hudson.node_monitors.SwapSpaceMonitor": null,
    "hudson.node_monitors.TemporarySpaceMonitor": null
  },
  "numExecutors": 2,
  "offline": true,
  "offlineCause": null,
  "offlineCauseReason": "",
  "oneOffExecutors": [],
  "temporarilyOffline": false
}
```

<a name="node-list"></a>
### jenkins.node.list(callback)

列出所有可用节点

传参

 * full (Boolean, default: false): 包含节点的执行次数

用法

``` javascript
jenkins.node.list(function(err, data) {
  if (err) throw err;

  console.log('nodes', data);
});
```

样例输出

``` json
{
  "busyExecutors": 0,
  "computer": [
    {
      "actions": [],
      "displayName": "master",
      "executors": [
        {},
        {}
      ],
      "icon": "computer.png",
      "idle": true,
      "jnlpAgent": false,
      "launchSupported": true,
      "loadStatistics": {},
      "manualLaunchAllowed": true,
      "monitorData": {
        "hudson.node_monitors.ArchitectureMonitor": "Linux (amd64)",
        "hudson.node_monitors.ClockMonitor": {
          "diff": 0
        },
        "hudson.node_monitors.DiskSpaceMonitor": {
          "path": "/var/lib/jenkins",
          "size": 77620142080
        },
        "hudson.node_monitors.ResponseTimeMonitor": {
          "average": 0
        },
        "hudson.node_monitors.SwapSpaceMonitor": {
          "availablePhysicalMemory": 22761472,
          "availableSwapSpace": 794497024,
          "totalPhysicalMemory": 515358720,
          "totalSwapSpace": 805302272
        },
        "hudson.node_monitors.TemporarySpaceMonitor": {
          "path": "/tmp",
          "size": 77620142080
        }
      },
      "numExecutors": 2,
      "offline": false,
      "offlineCause": null,
      "offlineCauseReason": "",
      "oneOffExecutors": [],
      "temporarilyOffline": false
    },
    {
      "actions": [],
      "displayName": "slave",
      "executors": [
        {},
        {}
      ],
      "icon": "computer-x.png",
      "idle": true,
      "jnlpAgent": true,
      "launchSupported": false,
      "loadStatistics": {},
      "manualLaunchAllowed": true,
      "monitorData": {
        "hudson.node_monitors.ArchitectureMonitor": null,
        "hudson.node_monitors.ClockMonitor": null,
        "hudson.node_monitors.DiskSpaceMonitor": null,
        "hudson.node_monitors.ResponseTimeMonitor": {
          "average": 5000
        },
        "hudson.node_monitors.SwapSpaceMonitor": null,
        "hudson.node_monitors.TemporarySpaceMonitor": null
      },
      "numExecutors": 2,
      "offline": true,
      "offlineCause": null,
      "offlineCauseReason": "",
      "oneOffExecutors": [],
      "temporarilyOffline": false
    }
  ],
  "displayName": "nodes",
  "totalExecutors": 2
}
```

<a name="queue-list"></a>
### jenkins.queue.list(callback)

列出节点中当前任务列表

用法

``` javascript
jenkins.queue.list(function(err, data) {
  if (err) throw err;

  console.log('queues', data);
});
```

样例输出

``` json
{
  "items": [
    {
      "actions": [
        {
          "causes": [
            {
              "shortDescription": "Started by user anonymous",
              "userId": null,
              "userName": "anonymous"
            }
          ]
        }
      ],
      "blocked": true,
      "buildable": false,
      "buildableStartMilliseconds": 1389418977387,
      "id": 20,
      "inQueueSince": 1389418977358,
      "params": "",
      "stuck": false,
      "task": {
        "color": "blue_anime",
        "name": "example",
        "url": "http://localhost:8080/job/example/"
      },
      "url": "queue/item/20/",
      "why": "Build #2 is already in progress (ETA:N/A)"
    }
  ]
}
```

<a name="queue-item"></a>
### jenkins.queue.item(options, callback)

查看任务列表中的项目

传参

 * number (Integer): 任务列表周某项的 ID

用法

``` javascript
jenkins.queue.item(130, function(err, data) {
  if (err) throw err;

  console.log('item', data);
});
```

样例输出

``` json
{
  "actions": [
    {
      "causes": [
        {
          "shortDescription": "Started by user anonymous",
          "userId": null,
          "userName": "anonymous"
        }
      ]
    }
  ],
  "blocked": false,
  "buildable": false,
  "id": 130,
  "inQueueSince": 1406363479853,
  "params": "",
  "stuck": false,
  "task": {
    "name": "test-job-b7ef0845-6515-444c-96a1-d2266d5e0f18",
    "url": "http://localhost:8080/job/test-job-b7ef0845-6515-444c-96a1-d2266d5e0f18/",
    "color": "blue"
  },
  "url": "queue/item/130/",
  "why": null,
  "executable" : {
    "number" : 28,
    "url" : "http://localhost:8080/job/test-job-b7ef0845-6515-444c-96a1-d2266d5e0f18/28/"
  }
}
```



<a name="queue-cancel"></a>
### jenkins.queue.cancel(options, callback)

将执行队列中某一项停掉

传参

 * number (Integer): 任务列表周某项的 ID

用法

``` javascript
jenkins.queue.cancel(23, function(err) {
  if (err) throw err;
});
```

<a name="view-config-get"></a>
### jenkins.view.config(options, callback)

获取视图的配置（XML 形式）

传参

 * name (String): 任务名称

用法

``` javascript
jenkins.view.config('example', function(err, data) {
  if (err) throw err;

  console.log('xml', data);
});
```

<a name="view-config-set"></a>
### jenkins.job.config(options, callback)

更新视图配置（XML 形式）

传参

 * name (String): 任务名称
 * xml (String): 配置文件

用法

``` javascript
jenkins.view.config('example', xml, function(err) {
  if (err) throw err;
});
```

<a name="view-create"></a>
### jenkins.view.create(options, callback)

创建视图

传参

 * name (String): 视图名称
 * type (String, enum: list, my): 视图类型

用法

``` javascript
jenkins.view.create('example', 'list', function(err) {
  if (err) throw err;
});
```

<a name="view-destroy"></a>
### jenkins.view.destroy(options, callback)

删除视图

传参

 * name (String): view name

用法

``` javascript
jenkins.view.destroy('example', function(err) {
  if (err) throw err;
});
```

<a name="view-exists"></a>
### jenkins.view.exists(options, callback)

检查视图是否存在

传参

 * name (String): view name

用法

``` javascript
jenkins.view.exists('example', function(err, exists) {
  if (err) throw err;

  console.log('exists', exists);
});
```

<a name="view-get"></a>
### jenkins.view.get(options, callback)

获取视图信息，json 返回

传参

 * name (String): view name

用法

``` javascript
jenkins.view.get('example', function(err, data) {
  if (err) throw err;

  console.log('view', data);
});
```

样例输出

``` json
{
  "description": null,
  "jobs": [
    {
      "name": "test",
      "url": "http://localhost:8080/job/example/",
      "color": "blue"
    }
  ],
  "name": "example",
  "property": [],
  "url": "http://localhost:8080/view/example/"
}
```

<a name="view-list"></a>
### jenkins.view.list(callback)

列出所有视图

用法

``` javascript
jenkins.view.list(function(err, data) {
  if (err) throw err;

  console.log('views', data);
});
```

样例输出

``` json
{
  "views": [
    {
      "url": "http://localhost:8080/",
      "name": "All"
    },
    {
      "url": "http://localhost:8080/view/example/",
      "name": "Test"
    }
  ],
  "useSecurity": false,
  "useCrumbs": false,
  "unlabeledLoad": {},
  "slaveAgentPort": 0,
  "quietingDown": false,
  "primaryView": {
    "url": "http://localhost:8080/",
    "name": "All"
  },
  "assignedLabels": [
    {}
  ],
  "mode": "NORMAL",
  "nodeDescription": "the master Jenkins node",
  "nodeName": "",
  "numExecutors": 2,
  "description": null,
  "jobs": [
    {
      "color": "notbuilt",
      "url": "http://localhost:8080/job/example/",
      "name": "test"
    }
  ],
  "overallLoad": {}
}
```

<a name="view-add"></a>
### jenkins.view.add(options, callback)

将任务添加到视图

传参

 * name (String): 视图名称
 * job (String): 任务名称

用法

``` javascript
jenkins.view.add('example', 'jobExample', function(err) {
  if (err) throw err;
});
```

<a name="view-remove"></a>
### jenkins.view.remove(options, callback)

将任务移除视图

传参

 * name (String): 视图名称
 * job (String): 任务名称

用法

``` javascript
jenkins.view.remove('example', 'jobExample', function(err) {
  if (err) throw err;
});
```

## License

本项目遵循 MIT License (see the LICENSE file).

## Notes

[python-jenkins](https://github.com/openstack/python-jenkins) (BSD License, see NOTES)
为本项目提供了大量思路。
