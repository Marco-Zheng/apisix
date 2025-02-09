---
title: referer-restriction
---

<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

## Summary

- [**Name**](#name)
- [**Attributes**](#attributes)
- [**How To Enable**](#how-to-enable)
- [**Test Plugin**](#test-plugin)
- [**Disable Plugin**](#disable-plugin)

## Name

The `referer-restriction` can restrict access to a Service or a Route by
whitelisting/blacklisting request header Referrers.

## Attributes

| Name      | Type          | Requirement | Default | Valid | Description                              |
| --------- | ------------- | ----------- | ------- | ----- | ---------------------------------------- |
| whitelist | array[string] | optional    |         |       | List of hostname to whitelist. The hostname can be started with `*` as a wildcard |
| blacklist | array[string] | optional    |         |       | List of hostname to blacklist. The hostname can be started with `*` as a wildcard |
| message | string | optional    | Your referer host is not allowed | [1, 1024] | Message returned in case access is not allowed. |
| bypass_missing  | boolean       | optional    | false   |       | Whether to bypass the check when the Referer header is missing or malformed |

One of `whitelist` or `blacklist` must be specified, and they can not work together.
The message can be user-defined.

## How To Enable

Creates a route or service object, and enable plugin `referer-restriction`.

```shell
curl http://127.0.0.1:9080/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/index.html",
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    },
    "plugins": {
        "referer-restriction": {
            "bypass_missing": true,
            "whitelist": [
                "xx.com",
                "*.xx.com"
            ]
        }
    }
}'
```

## Test Plugin

Request with `Referer: http://xx.com/x`:

```shell
$ curl http://127.0.0.1:9080/index.html -H 'Referer: http://xx.com/x'
HTTP/1.1 200 OK
...
```

Request with `Referer: http://yy.com/x`:

```shell
$ curl http://127.0.0.1:9080/index.html -H 'Referer: http://yy.com/x'
HTTP/1.1 403 Forbidden
...
{"message":"Your referer host is not allowed"}
```

Request without `Referer`:

```shell
$ curl http://127.0.0.1:9080/index.html
HTTP/1.1 200 OK
...
```

## Disable Plugin

When you want to disable the `referer-restriction` plugin, it is very simple,
you can delete the corresponding json configuration in the plugin configuration,
no need to restart the service, it will take effect immediately:

```shell
$ curl http://127.0.0.1:9080/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/index.html",
    "plugins": {},
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    }
}'
```

The `referer-restriction` plugin has been disabled now. It works for other plugins.
