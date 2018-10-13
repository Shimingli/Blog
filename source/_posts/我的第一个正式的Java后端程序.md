---
title: 我的第一个正式的Java后端程序
date: 2018-10-13 09:38:50
tags: Java后端
---
* 项目背景： 公司有家新零售店，出租方是要根据每天的销售额度去收取租金，需要把后台的每天的订单数据给推送到第三方。关键这个数据不在我们的后台，是在有赞后台，由于公司人手紧张，领导直接安排我做，好吧！安卓工作量不饱和，后端来凑！
* 通过`Http`请求获取第三方原始数据，然后把原始数据拼接成想要的`json`，然后使用`WebServices`推送到第三方!就是这么的一个过程。
<!--  more  -->
#### 1、对接微信群，对接文档开干
![微信群](https://upload-images.jianshu.io/upload_images/5363507-f47839eb9b50a46f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/5363507-3f16b2ea433e34e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* what fuck？？？黑人问号，  `WebServices` 第一次撸后端代码，你就给我来个`WebServices`的，我怎么玩？喝点水，冷静下，在此感谢公司的后端大佬宏真哥给出的思路！

* 在这里有个插曲，别人提供的对接文档是错误，真的是搞死我这个小白了，在这里想骂人，心疼宝宝几秒，谁不是个宝宝啊！
![image.png](https://upload-images.jianshu.io/upload_images/5363507-873adc18cfd49b6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 找到文档提供的接口，如果能够正常的访问的话，那么第一步就应该可以了 
![正常访问的结果](https://upload-images.jianshu.io/upload_images/5363507-3f0b8c44d52a8080.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2、生成对应的jar包
* 感谢文章 [Intellij Idea 下 生成WebServiceClient (WS客户端)](https://blog.csdn.net/qq_35067322/article/details/53558775?locationNum=8&fps=1)
* 拿到测试的地址了，访问也正常了，使用`ide`生成代码，来开始动起来，如下步骤
![1](https://upload-images.jianshu.io/upload_images/5363507-9c61a9e3fc64bdd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2](https://upload-images.jianshu.io/upload_images/5363507-ca7bccd5aa804827.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 我遇到了一个问题，在此感谢 [软件包 junit.framework 不存在的解决方法](https://blog.csdn.net/zhang103886108/article/details/44410753)
* 如果一切顺利的话，那么就会看到这个界面
![3](https://upload-images.jianshu.io/upload_images/5363507-7e015ba69c71636e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![4](https://upload-images.jianshu.io/upload_images/5363507-e546249fb5601126.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 3、那么现在的问题是把这堆代码打成jar
* 感谢这个博客对我的帮助 [Idea 将新写的项目中的一个java文件搞成jar包方法](https://blog.csdn.net/m0_37543304/article/details/70171088)
![1](https://upload-images.jianshu.io/upload_images/5363507-b9fce4773778103a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2](https://upload-images.jianshu.io/upload_images/5363507-a8e22df201320a31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 生成了如下的文件 
![3](https://upload-images.jianshu.io/upload_images/5363507-1d4e2c4366f1ab24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![4](https://upload-images.jianshu.io/upload_images/5363507-5d4192e640b0ebf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 点击   `Build` 
![5](https://upload-images.jianshu.io/upload_images/5363507-457b3801d62800d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 接下来就去输出目录找jar包了
![6](https://upload-images.jianshu.io/upload_images/5363507-d99a275dc2c44b31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 这就是我可以使用的  `jar`,如果后续需要改为正式的地址，还需要生成一个正式的`jar`

#### 3、把生成的 `jar`放到`Maven Nexus3`
* 我没有权限！哎哎
![image.png](https://upload-images.jianshu.io/upload_images/5363507-c077ff7df3b6e78e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 还好我们架构师帮我上传了
![2](https://upload-images.jianshu.io/upload_images/5363507-53f5ddd4529f1bd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 配置`pom`
```
       <dependency>
            <groupId>net.ticp.asiatic</groupId>
            <artifactId>ftp-pingan-sdk</artifactId>
            <version>1.0.0</version>
        </dependency>
```
* 开始运行项目， `what fuck ？？？`
![问题](https://upload-images.jianshu.io/upload_images/5363507-2afd3acb7629d925.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 完蛋，这个错误不认识啊！问大佬，再次感谢大佬的相助。
![1](https://upload-images.jianshu.io/upload_images/5363507-cd08167241de48d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4、写代码逻辑
* 再次感谢公司的另外一个大佬毛毛，因为这个流程是我请求第三方的后台然后获取关键数据，然后把关键数据整合通过`WebServices`推送给第三方。我一个做安卓的在这方面经验不足，他这边完成了一个小`Demo`，也没叫我看，我自己看他的代码，改动的逻辑，啊哈哈
```

/**
 * author： Created by shiming on 2018/9/30 17:14
 * mailbox：lamshiming@sina.com
 */
@Service
@ElasticJobConf(name="YouzanPushOrderPingAnSynJob",cron = "*/5 * * * * ?")
 public class YouzanPushOrderPingAnSynJob implements SimpleJob {

    @Resource
    private YZClientService yzClientService;
    /**
     * 每页数量
     */
    private static final Long PAGE_SIZE = 20L;

    private static final Logger logger = LoggerFactory.getLogger(YouzanPushOrderPingAnSynJob.class);
    @Override
    public void execute(ShardingContext shardingContext) {
        YouzanRetailOpenDeliveryorderQuery youzanRetailOpenDeliveryorderQuery = buildQueryApi();
        YouzanRetailOpenDeliveryorderQueryResult result = yzClientService.invoke(youzanRetailOpenDeliveryorderQuery);
        // 总页数
        Long pages = parsePages(result.getPaginator().getTotalCount());
        int j=0;
        int k=0;
        OrderPushUtils.m=0;
        // 分页查询
        for (long pageNo = 1; pageNo <= pages ; pageNo++) {
            // 设置分页
            setPage(youzanRetailOpenDeliveryorderQuery,pageNo);
            // 查询结果
            result = yzClientService.invoke(youzanRetailOpenDeliveryorderQuery);
            // 处理查询结果
            YouzanRetailOpenDeliveryorderQueryResult.OpenDeliveryOrderDTO[] deliveryOrders = result.getDeliveryOrders();
            if (deliveryOrders!=null&&deliveryOrders.length>0){
                for (int i=0;i<deliveryOrders.length;i++){
                    //平安店的数据 这样才能正确 而且还是线下店才好
                    if (YouzanConst.ORDER_FORM_PINGANDIAN.equals(deliveryOrders[i].getWarehouseCode())&&YouzanConst.SALE_WAY_OFFLINE.equals(deliveryOrders[i].getSaleWay())) {
                        // 这里就是满足平安店的订单
                        OrderPushUtils.pushOrder(deliveryOrders[i]);
                        j++;
                        System.out.println("一共有多少平安店的单 ：："+j);

                        if (i==2){
                            String s = new Gson().toJson(result);
                            System.out.println("result=="+s);
                        }
                    }
                    if (YouzanConst.ORDER_FORM_CHEGONGMIAO.equals(deliveryOrders[i].getWarehouseCode())&&YouzanConst.SALE_WAY_OFFLINE.equals(deliveryOrders[i].getSaleWay())) {
                        k++;
                        System.out.println("一共有车公庙的单 ：："+k);
                    }
                }
            }
        }
    }
    int i=1;

    private YouzanRetailOpenDeliveryorderQuery buildQueryApi() {
        // 当前时间前一天的时间
        Date currentDate = new Date(System.currentTimeMillis()-86400*1000*i);
        i++;
        // 查询开始时间
        Date queryStart = DateUtils.getDateStart(currentDate);
        // 查询结束时间
        Date queryEnd = DateUtils.getDateEnd(currentDate);
        String queryEndTime = DateUtils.formatDateTime(queryEnd);
        String queryStartTime = DateUtils.formatDateTime(queryStart);
        logger.info("查询有赞后台的开始时间"+queryStartTime);
        logger.info("查询有赞后台的结束时间"+queryEndTime);
        Date date = new Date();
        String nowTime = DateUtils.formatDateTime(date);
        logger.info("当前查询的时间"+nowTime);
        YouzanRetailOpenDeliveryorderQueryParams youzanRetailOpenDeliveryorderQueryParams = new YouzanRetailOpenDeliveryorderQueryParams();
        youzanRetailOpenDeliveryorderQueryParams.setPageNo(1L);
        youzanRetailOpenDeliveryorderQueryParams.setPageSize(1L);
        youzanRetailOpenDeliveryorderQueryParams.setCreateTimeStart(DateUtils.formatDate(queryStart));
        youzanRetailOpenDeliveryorderQueryParams.setCreateTimeEnd(DateUtils.formatDate(queryEnd));
        youzanRetailOpenDeliveryorderQueryParams.setCreateTimeStart(queryStartTime);
        youzanRetailOpenDeliveryorderQueryParams.setCreateTimeEnd(queryEndTime);
        YouzanRetailOpenDeliveryorderQuery youzanRetailOpenDeliveryorderQuery = new YouzanRetailOpenDeliveryorderQuery();
        youzanRetailOpenDeliveryorderQuery.setAPIParams(youzanRetailOpenDeliveryorderQueryParams);
       return youzanRetailOpenDeliveryorderQuery;
    }

    private void setPage(YouzanRetailOpenDeliveryorderQuery queryApi,Long pageNo){
        YouzanRetailOpenDeliveryorderQueryParams queryAPIParams = (YouzanRetailOpenDeliveryorderQueryParams)queryApi.getAPIParams();
        queryAPIParams.setPageNo(pageNo);
        queryAPIParams.setPageSize(PAGE_SIZE);
    }

    private Long parsePages(long totalCount){
        if (totalCount == -1) {
            return 1L;
        }
        return totalCount / PAGE_SIZE + ((totalCount % PAGE_SIZE == 0) ? 0 : 1);
    }
}

```
* `     OrderPushUtils.pushOrder(deliveryOrders[i]);` 这个方法我要去区分是哪家零售店的数据，同时在组装json数据，就不贴出来了

#### 5、万万没有想到
* 第三方接口有`bug`，导致推送到第三方的数据总额不正确，我这一口老血啊，吐你一脸。
* 每个订单的销售收款总额竟然不等于里面的子订单的总和！
![问题](https://upload-images.jianshu.io/upload_images/5363507-4cdfbfd6d3bfc6e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-d3e411351fdb9bdf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-ba2f3a0b1d6c0b86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 最后到了这一步,等待他们发版本
![image.png](https://upload-images.jianshu.io/upload_images/5363507-2f41e9a8e61ddf15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6、最后说明几点
* 感谢公司的大佬对我的帮助
* 特别感谢宏真大佬的支持
* 感谢一下博客对我的帮助
  *   [软件包 junit.framework 不存在的解决方法](https://blog.csdn.net/zhang103886108/article/details/44410753)
  *    [Idea 将新写的项目中的一个java文件搞成jar包方法](https://blog.csdn.net/m0_37543304/article/details/70171088)
  *   [Intellij Idea 下 生成WebServiceClient (WS客户端)](https://blog.csdn.net/qq_35067322/article/details/53558775?locationNum=8&fps=1)
* [本文生成资料](https://github.com/Shimingli/WebServicesDemo)
* 同时感谢我的好同学波哥对我的大力帮助，并且还提供了`Demo`，非常感谢！
* [波哥给的资料](https://github.com/Shimingli/TrasAddForMultiple)



