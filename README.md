# vperserv-loadteststats

Share loadtest statistics result for VPER (https://vper.world in development) comparing NodeJS, and Go.

# Objective

This loadtest focuses on compare general performance between NodeJS and Golang. Author didn't go fully overboard to optimize every aspect of backend code, but do so in certain level with more bias towards Go implementation one as author switches from NodeJS to Golang to prove the claim regarding to performance-wise.

# How

Server is Ubuntu 16.04 (4.4.0-91-generic) with 1G RAM, 1 single CPU 2.399 Ghz with 1 core located in Hongkong.
Load-test client is macOS 10.13.4 High Sierra machine 8 GB Ram, CPU 2.5 GHz Intel Core i5; initiated from Shenzhen, China connected through VPN server which located in Hongkong to maintain its locality.

Load-testing uses Apache Bench (ab) which is pre-installed on macOS machine.

> This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
> Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
> Licensed to The Apache Software Foundation, http://www.apache.org/

Both NodeJS and Go server has similar functionality. Go is the port version from NodeJS as author switched to use but things done in Go's way.

NodeJS => Node 8.10.0,  express 4.15.4, and pg 7.4.1
Go => Go 1.10.3, jackc/pgx (39bbc98), gorilla/mux (cb46983), valyala/fastjson (c58febb), valyala/quicktemplate (a91e094) 

There are 3 sets of loadtests done for both NodeJS, and Go server as follows

1. Concurrent connection (C)=1, Test timelimit in seconds (T)=60s, Limit number of request (R)=1000000
2. C=10, T=60, R=1000000
3. C=100, T=60, R=1000000
4. C=200, T=60, R=1000000
5. C=300, T=60, R=1000000
6. C=400, T=60, R=1000000
7. C=500, T=60, R=1000000

API end-point that serves incoming requests will make DB query in form `SELECT * FROM table WHERE x >= $1 AND x <= $2 AND y >= $3 AND y <= $4;` which is executed through prepared statement for performance. Such table at the time of load-testing has total number or records of 101447.

It responds back in JSON form as follows

```json
{
    "response": {
        "count": <integer>,
        "rows": [
            {
                "field1": <string>,
                "field2": <string>,
                "field3": <int64>,
                "field4": <int32>,
                "field5": <int32>
            },
            ...
        ]
    },
    "status_code": 1,
    "status_message": "OK"
}
```

and according to input SQL Query we made, total size of response is `0.683481` MB with 4649 items in `"rows"` field.

# `ab` command

Executed via `ab -t 60 -n 1000000 -c <number of concurrent connections> -k -H "<customheader>:<customvalue>" <url-end-point>`

in which substitute parameter as need. Notice that `-k` to enable Keep alive connection, and `-H` to specify custom header as our application uses it.

Also `-t` needs to comes before `-n` to effectively make load-test session lasts for specified duration of 60s. Otherwise it will only lasts for 30s by default.

# Result

Roughly concurrent connection at 400-500 is the deminish point for Go as it's starting to have failed requests. But NodeJS has more severe and significant failure of requests much more than Go. At C=500, around 50% performance (in terms of number of requests served) dropped for NodeJS, but compared to Go; it's just slightly.

Keep alive connection in HTTP as well plays an important role. If you disable without supplying `-k` in `ab` command, you will have much much less of requests served as shown in result.

## Peak CPU & Memory Usage (per process, not whole server) for Concurrency connection of 500

| Server | CPU (%) | Mem (%) |
| --- | --- | ---: |
| NodeJS | 30.0 | 8.7 |
| Go | 30.0 | 2.7 |

## Load-test Statistics

See in corresponding directories C1, C10, C100, C200, C300, C400, and C500.

# Last Word

I hope this load-test statistics would be helpful to you. In case you have question regarding the test itself, contact [@haxpor](https://github.com/haxpor).

# License

[Apache License 2.0](https://github.com/abzico/vperserv-loadteststats/blob/master/LICENSE)
