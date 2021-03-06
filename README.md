#* 分布式事务的解决方案*

walker is a useful library designed to resolve cross transaction of RDBMS in complex distributed systems


```java
  I am a slow walker
  But I never stop walking
  The important thing in life is to have a great aim and the determination to attain it
```

### 感谢各位
[star member](https://github.com/devpage/walker/stargazers)
|
[fork member](https://github.com/devpage/walker/network/members)
| 提醒一下 如果只关注 但不参与开发的话 watch就足够了,这样你可以看到最新的代码

****
## 目录
* [由来](#由来)
* [声明](#声明)
* [联系我](#联系我)
* [时序图](#时序图)
* [模块](#模块)
* [数据结构](#数据结构)
    - [分支事务](#分支事务)
      *  [分支事务状态](#分支事务状态)  
    - [分支补偿](#分支事务)
      *  [分支补偿状态](#分支补偿状态)  
* [原理](#原理) 
    * [应用侧](#应用侧) 
    * [协调者](#协调者) 
* [代码示例](#代码示例) 
* [接入方法](#接入方法)
* [重要核心](#重要核心)
* [性能优化](#性能优化)
* [扩展用法](#扩展用法)
* [思想参考](#思想参考) 
****


### 由来
- [ ] 其他人的实现方式 对应用环境要求太高,通用性差 (明白道理之后 自己随意改造即可)
- [x] 我不得不站出来做一个更好理解的

### 声明
1. walker当前处在 **开发中** 阶段，由于只有我1人开发，基本都是下班或者星期天在家开发，所以进度不是很快 
1. 你可以看看已有代码的实现方式，或许对你会有一些提示\帮助 
1. 欢迎提交你的issue，代码只是简单的show you the code，思想的共鸣才是更重要的交流

### 联系我
|Author|宋|说明|
|---|---|---|
|E-mail|1422204321@qq.com|加好友时 备注: github_walker_你的名字|

### 时序图
 [PlantUML](_doc/walker.puml)

### 模块
|模块名称|主要功能|
|---|---
|walker|项目介绍
|walker-admin|[项目介绍](walker-admin)
|其他模块|自行脑补

### 数据结构
* core
    * 语言
        * Java
* bridge
    * 语言
        * Java
* protocol
    * 语言
        * Java
* rabbitmq
    * 语言
        * Java
* coordinator
    * 语言
        * Java
        
### 原理
> 正统的TCC原理(try commit cancel):
>> 在spring的事务管理器外层包装一层 walker的封装, 在单一RDBMS数据库事务的基础上 通过逻辑来做分布式事务一致性

### 代码示例
```java
   
    @Autowired
    private RemoteFeignService remoteFeignService;

    @TccTransaction(id = "must be unique id",
                needJudge = false,
                mode = CompensateMode.ASYNC,
                commitMethodName = "diy_commit", commitMethodArgs = {"#yourBusinessModel.gid", "#yourBusinessModel.boolParam"},
                cancelMethodName = "diy_cancel", cancelMethodArgs = {"#yourBusinessModel.doubleParam", "#yourBusinessModel.longParam", "#p0.gid"})
    @Transactional(rollbackFor = {Exception.class})
    public void aroundWithWalkerTransactionManagement(YourBusinessModel yourBusinessModel) {

        /**
         * input your business code
         */
        aroundWithSpringTransactionManagement(yourBusinessModel);
    }

    @Transactional(rollbackFor = {Exception.class})
    public void aroundWithSpringTransactionManagement(YourBusinessModel yourBusinessModel) {

        /**
         * input your business code
         */
        remoteFeignService.pay(yourBusinessModel);

        System.out.println("I will rollback with my brother");

        withWalkerTransactionManagement(yourBusinessModel.getGid(), yourBusinessModel.isBoolParam(), yourBusinessModel.getDoubleParam(), yourBusinessModel.getLongParam());
    }


    @TccTransaction(id = "must be unique id!",
            needJudge = false,
            mode = CompensateMode.ASYNC,
            commitMethodName = "diy_commit", commitMethodArgs = {"#gid", "#b"},
            cancelMethodName = "diy_cancel", cancelMethodArgs = {"#paramOfDouble", "#typeOfLong", "#gid"})
    @Transactional(rollbackFor = {Exception.class})
    public void withWalkerTransactionManagement(String gid, boolean b, double paramOfDouble, long typeOfLong) {
        /**
         * do you understand ?
         *
         */
        System.out.println("I will rollback with my brother, too");
    }



    @Transactional
    public Participant diy_commit(String gid, boolean boolParam) {
        WalkerContext walkerContext = WalkerContextlManager.getContext();
        Map<String, Object> commitBody = Maps.newLinkedHashMap();
        commitBody.putIfAbsent("cancel_active_gid", gid);
        commitBody.putIfAbsent("根据需要填写你的参数1", boolParam);
        return Participant.create(walkerContext.getMasterGid(), walkerContext.getCurrentBranchGid(), "your_business_commit_url", commitBody, "something that need callback param from coordinator");
    }

    @Transactional
    public Participant diy_cancel(double paramOfDouble, long typeOfLong, String businessGid) {
        WalkerContext walkerContext = WalkerContextlManager.getContext();
        Map<String, Object> cancelBody = Maps.newLinkedHashMap();
        cancelBody.putIfAbsent("cancel_active_gid", businessGid);
        cancelBody.putIfAbsent("根据需要填写你的参数1", paramOfDouble);
        cancelBody.putIfAbsent("根据需要填写你的参数2", typeOfLong);
        return Participant.create(walkerContext.getMasterGid(), walkerContext.getCurrentBranchGid(), "your_business_cancel_url", cancelBody, "something that need callback param from coordinator");
    }

```
### 接入方法
参考我的写法 或 引入如下的lib
```xml
<dependency>
  <groupId>walker</groupId>
  <artifactId>walker-core</artifactId>
  <version>x.y.z</version>
</dependency>
<dependency>
  <groupId>walker</groupId>
  <artifactId>walker-rabbitmq</artifactId>
  <version>x.y.z</version>
</dependency>
 
```
### 重要核心

### 性能优化

### 扩展用法

### 思想参考
- 支付宝分布式事务设计草案:  [支付宝分布式事务设计草案PDF](_doc/支付宝分布式事务设计草案.pdf)
