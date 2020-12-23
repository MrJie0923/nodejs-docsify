### Nacos 服务注册

---

#### 客户端往注册中心发送心跳 

心跳java对象`com.alibaba.nacos.client.naming.beat.BeatInfo`

具体执行发送心跳代码``com.alibaba.nacos.client.naming.beat.BeatReactor`

```java
class BeatTask implements Runnable {
    BeatInfo beatInfo;

    public BeatTask(BeatInfo beatInfo) {
        this.beatInfo = beatInfo;
    }

    public void run() {
        if (!this.beatInfo.isStopped()) {
            long result = BeatReactor.this.serverProxy.sendBeat(this.beatInfo);
            long nextTime = result > 0L ? result : this.beatInfo.getPeriod();
            BeatReactor.this.executorService.schedule(BeatReactor.this.new BeatTask(this.beatInfo), nextTime, TimeUnit.MILLISECONDS);
        }
    }
}
```

#### NamingProxy类解析

* 注册服务

  ```java
  public void registerService(String serviceName, String groupName, Instance instance) throws NacosException {
          LogUtils.NAMING_LOGGER.info("[REGISTER-SERVICE] {} registering service {} with instance: {}", new Object[]{this.namespaceId, serviceName, instance});
          Map<String, String> params = new HashMap(9);
          params.put("namespaceId", this.namespaceId);
          params.put("serviceName", serviceName);
          params.put("groupName", groupName);
          params.put("clusterName", instance.getClusterName());
          params.put("ip", instance.getIp());
          params.put("port", String.valueOf(instance.getPort()));
          params.put("weight", String.valueOf(instance.getWeight()));
          params.put("enable", String.valueOf(instance.isEnabled()));
          params.put("healthy", String.valueOf(instance.isHealthy()));
          params.put("ephemeral", String.valueOf(instance.isEphemeral()));
          params.put("metadata", JSON.toJSONString(instance.getMetadata()));
          this.reqAPI(UtilAndComs.NACOS_URL_INSTANCE, params, (String)"POST");
      }
  public String callServer(String api, Map<String, String> params, String curServer, String method) throws NacosException {
          long start = System.currentTimeMillis();
          long end = 0L;
          this.checkSignature(params);
          List<String> headers = this.builderHeaders();
          String url;
          if (!curServer.startsWith("https://") && !curServer.startsWith("http://")) {
              if (!curServer.contains(":")) {
                  curServer = curServer + ":" + this.serverPort;
              }
  
              url = HttpClient.getPrefix() + curServer + api;
          } else {
              url = curServer + api;
          }
  
          HttpResult result = HttpClient.request(url, headers, params, "UTF-8", method);
          end = System.currentTimeMillis();
          MetricsMonitor.getNamingRequestMonitor(method, url, String.valueOf(result.code)).observe((double)(end - start));
          if (200 == result.code) {
              return result.content;
          } else if (304 == result.code) {
              return "";
          } else {
              throw new NacosException(500, "failed to req API:" + curServer + api + ". code:" + result.code + " msg: " + result.content);
          }
      }
  ```

  

* 更新服务列表信息

  ```java
  //Nacos客户端主动刷新服务列表时间间隔为30s
  private void initRefreshSrvIfNeed() {
          if (!StringUtils.isEmpty(this.endpoint)) {
              ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1, new ThreadFactory() {
                  public Thread newThread(Runnable r) {
                      Thread t = new Thread(r);
                      t.setName("com.alibaba.nacos.client.naming.serverlist.updater");
                      t.setDaemon(true);
                      return t;
                  }
              });
              executorService.scheduleWithFixedDelay(new Runnable() {
                  public void run() {
                      NamingProxy.this.refreshSrvIfNeed();
                  }
              }, 0L, this.vipSrvRefInterMillis, TimeUnit.MILLISECONDS);
              this.refreshSrvIfNeed();
          }
      }
  ```

* 取消注册,调用Nacos API`DELETE`删除服务

  ```java
  public void deregisterService(String serviceName, Instance instance) throws NacosException {
          LogUtils.NAMING_LOGGER.info("[DEREGISTER-SERVICE] {} deregistering service {} with instance: {}", new Object[]{this.namespaceId, serviceName, instance});
          Map<String, String> params = new HashMap(8);
          params.put("namespaceId", this.namespaceId);
          params.put("serviceName", serviceName);
          params.put("clusterName", instance.getClusterName());
          params.put("ip", instance.getIp());
          params.put("port", String.valueOf(instance.getPort()));
          params.put("ephemeral", String.valueOf(instance.isEphemeral()));
          this.reqAPI(UtilAndComs.NACOS_URL_INSTANCE, params, (String)"DELETE");
      }
  ```

  

