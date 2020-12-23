# CURL命令模拟Http Get/Post请求

##### Get请求

 curl [protocol://address:port/url?args](https://link.jianshu.com/?t=protocol://address:port/url?args)
`curl http://127.0.0.1:8080/check_your_status?user=Summer&passwd=12345678`

curl "http://www.baidu.com"  如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地

curl -i "http://www.baidu.com"  显示全部信息

curl -l "http://www.baidu.com" 只显示头部信息

curl -v "http://www.baidu.com" 显示get请求全过程解析

##### Post请求

 curl -d "args" "[protocol://address:port/url](https://link.jianshu.com/?t=protocol://address:port/url)"
`curl -d "user=Summer&passwd=12345678" "http://127.0.0.1:8080/check_your_status"`

这种方法是参数直接在header里面的
如需将输出指定到文件可以通过重定向进行操作.
curl -H "Content-Type:application/json" -X POST --data (json.data) URL
`curl -H "Content-Type:application/json" -X POST --data '{"message": "sunshine"}' http://localhost:8000/`
这种方法是json数据直接在body里面的