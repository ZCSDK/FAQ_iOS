###1 如何转人工 * 相似问题：  如何转人工？  转人工失败？  无法转接人工？  

* 接待模式：  
1 机器人优先：可配置是否显示转人工按钮  2 人工客服优先：如果客服不在线会自动切到机器人模式  3 仅人工客服：如果客服不在线，直接结束会话  4 仅机器人客服：不显示转人工按钮，无法转接人工客服  
智齿SDK分为机器人客服和人工客服两种接入场景，两种场景可自由切换；可通过配置自由选择初始接入模式,转人工逻辑会根据接入模式自动实现。  
1、接口配置：由工作台统一管理,*"设置->APP->客服设置(选择你的APP)->客服模式"*  
	关键词配置：*“设置->机器人->转人工关键词设置”*2、本地配置：

**[注意：本地配置会覆盖接口配置，如果本地设置了大于0的配置，接口配置无效。]**

```
    if([ZCLibClient getZCLibClient].libInitInfo == nil){
        [ZCLibClient getZCLibClient].libInitInfo = [ZCLibInitInfo new];
    }
    // 默认是0
    [ZCLibClient getZCLibClient].libInitInfo.service_mode = 0;
    
    
    //指定客服ID，可不配置
    initInfo.receptionistId = @"";
    // 指定客服0 可转入其他客服 1 必须转入指定客服，默认是0
    initInfo.tran_flag = 0;
```

 * ZCKitInfo.h中相关配置说明：
 
**isShowTansfer** ：默认YES，机器人优先模式，是否直接显示转人工按钮(值为NO时，会在机器人无法回答时显示转人工按钮)。   
**unWordsCount** ：机器人优先模式和isShowTansfer配合使用，通过记录机器人未知说辞的次数设置是否直接显示转人工按钮。  
**isOpenActiveUser** ：是否开启智能转人工,(如输入“转人工”，直接转接人工)，需要隐藏转人工按钮，请参见isShowTansfer和unWordsCount属性。  
**activeKeywords**:智能转人工关键字配置，关键字作为key{@"转人工",@"1",@"R":@"1"}   

[效果图]

###2 如何设置技能组： * 相似问题：    如果获取技能组？    指定技能组不起作用？    有客服在线但是提示无客服在线？  
技能组怎么溢出？  

* 技能组  
	客服的分组，一个客服可在多个组，可在工作台设置*"设置->APP->客服设置(选择你的APP)->分组接待"*  是否开启或关闭，如果关闭了配置无效。

1、SDK端指定技能组；如果当前指定技能组没有客服在线，没有设置溢出的情况会提醒暂无客服在线，无法正常转接到其他人工客服。获取技能组信息*"设置->在线->在线技能组设置->获取技能组信息"*：

```

    ZCLibInitInfo *initInfo = [ZCLibClient getZCLibClient].libInitInfo;
    initInfo.groupid = @"";
    initInfo.group_name = @"";

```  
2、SDK端配置技能组溢出：

* 参数说明：  
 actionType:执行动作类型：to_group:转接指定技能组。 
 optionId:是否溢出  指定技能组时：3：溢出，4：不溢出。  
 deciId:指定的技能组。  
 spillId:溢出条件  指定客服组时：4:技能组无客服在线,5:技能组所有客服忙碌,6:技能组不上班,7:智能判断  

```
ZCLibInitInfo *initInfo = [ZCLibClient getZCLibClient].libInitInfo; 
//指定溢出组
initInfo.transferaction = @[@{@"actionType":@"to_group",@"optionId":@"3",@"deciId":@"162bb6bb038d4a9ea018241a30694064",@"spillId":@"7"}];


// 电商版本跨公司溢出逻辑
// 溢出方式 默认0不开启 1-全部溢出，2-忙碌时溢出，3-不在线时溢出
initInfo.flow_type = 0;
// 转接到的公司技能组
initInfo.flow_groupid = @"";
// 转接到的公司ID
initInfo.flow_companyid = @"";
    
```
3、SDK端不做任何设置：
显示转人工时，自动弹出技能组列表供用户选项；如果指定了客服或技能组，则不会触发此弹层点击转人工会直接转到指定的人工客服。



###3 如何确定用户身份
* 相似问题：  
	同一个设备未指定用户id如何区别用户？  
	设置了相同的id，会怎么样？  
partnerid（2.8.0 之前为 userId）： 在线接入时智齿客服系统用于确认用户身份唯一标识，字符串类型无限制，建议设置值。如果不传 partnerid ，SDK会使用 CFUUIDCreate 方法生成一个编码存在 Keychain 中。    

<font color=#FF0000 >千万不要设置固定的值，这会导致相同的id共享所有的信息(比如把游客都设置为000，后果非常严重，无法修复)。</font>

```
ZCLibInitInfo *initInfo = [ZCLibClient getZCLibClient].libInitInfo; 
initInfo. partnerid =  @"您的用户IDxxx";

```

###4 如何设置自定义参数  
* 相似问题：  
	如何扩展用户信息？   
	怎么传递用户信息到工作台？  
	传递的自定义参数无法统计到？  
	传递的自定义参数工作台无法查看？  

目前SDK自定义参数是通过启动客服页面时统一传递，所以只需要在启动客服页面之前添加到配置信息中即可，没有单独的接口主动触发同步信息。由于这种实现方式的限制，如果已经进入到聊天页面以后修改的自定义信息，是无法实时传递到工作台中。    
自定义参数分为2种类型，分别如下：  
 *不固定key*
 仅在工作台转人工后人工可见，只存在于访客信息中不会同步到客户中心，无法在统计中查看到
 
params(2.8.0以前为customInfo)：字典类型，key：value随便填写，最终会转成json传给后端展示。

 ```
 ZCLibInitInfo *initInfo = [ZCLibClient getZCLibClient].libInitInfo; 
 initInfo.params = @{@"key1":@"value1"};
 
 ```
 
 
 *固定key*
获取key*"设置->自定义字段->客户字段->显示ID"*，人工客服可见(需要把自定义字段添加到工作台中才可以*”设置->自定义字段->工作台字段->配置显示字段“*)，会同步到客户中心，在统计中可以查看到。  

customer_fields(2.8.0以前为customerFields)：字典类型，key为显示ID：value随便填写。

 ```
 ZCLibInitInfo *initInfo = [ZCLibClient getZCLibClient].libInitInfo; 
 initInfo.customer_fields = @{@"customField1":@"我是添加的固定自定义字段"};
 
 ```

###5 推送的实现方式
* 相似问题：  
 收不到推送？
 推送怎么处理？
 推送的未读消息数不对？
 如果关闭智齿消息通道？

智齿SDK会有一个自己的长连接通道，用户可自由关闭或打开，目前2.8.2版本打开通道可能会导致APP后台挂起时间缩短，可以离开智齿SDK是关闭智齿的长连接通道，此时有消息会走苹果的APNS推送，不会影响消息的提醒。

如果配置APNS（详情见接入文档），要支持APNS推送，在创建appkey时必须上传p12证书，密码不能为空*“设置->APP->添加APP”*
```//本地通道未读消息数 
[ZCLibClient getZCLibClient].receivedBlock = ^(id obj,int unRead,NSDictionary *object){ 
//       NSLog(@"当前消息数：%d \n %@",unRead,obj) ;
});
/ @note 检查当前消息通道是否建立，没有就重新建立， 如果调用 closeIMConnection 后，可调用此方法重新建立链接 [[ZCLibClient getZCLibClient] checkIMConnected];   /* 检查当前监听是否被移除，如果移除就重新注册(重新激活 网络监听 ZCNotification_NetworkChange UIApplicationDidBecomeActiveNotification UIApplicationDidEnterBackgroundNotifica )，经常和checkIMConnected一起使用 */ [[ZCLibClient getZCLibClient] checkIMObserverWithRegister]; /** 移除IM所有监听，可能会影响应用在后台存活时长，如果调用此方法后需要checkIMObserverWithCreate重新激活 网络监听 ZCNotification_NetworkChange UIApplicationDidBecomeActiveNotification UIApplicationDidEnterBackgroundNotification */ [[ZCLibClient getZCLibClient] removeIMAllObserver]; /** @note 关闭当前消息通道，使其不再接受消息，如果配置推送了，消息会走apns送达 */   [[ZCLibClient getZCLibClient] closeIMConnection]; // 关闭通道，清理内存，退出智齿客户(如果当前是人工咨询状态客户会离线，如果是机器人状态会直接中断当前会话，下次进入是新会话) // 说明：调用此方法后将不能接收到离线消息，除非再次进入智齿SDK重新激活 // isClosePush:YES ,是关闭push；NO离线用户，但是可以收到push推送 +(void) closeAndoutZCServer:(BOOL) isClosePush; //事例 [ZCLibClient closeAndoutZCServer:YES];   
```* 开启留言转离线消息功能
	客户在客服无法提供服务时可提交留言，提交的留言可转化为待处理工单，或者以离线消息形式发送给客服。 需要在后台配置：在 *“设置 -> 留言设置 -> 开启 留言转离线消息”*开启留言功能后，客户在客服无法提供服务时可提交留言，提交的留言可转化为待处理工单，或者以离线消息形式发送给客服。 



###6、接⼊入SDK后台挂起时间收到影响
* 相似问题：    添加智⻮齿SDK后后台挂起时间变短了了怎么办?
  
由于智⻮齿SDK后台有一个⻓长连接监听消息，会使⽤到系统相关的⽹网络、屏幕退出、进入相关监听事件，导致退出后台后APP很容易易被系统杀死;如果对相关问题敏感的APP，可通过如下代码暂停智⻮齿相关服务，进⼊入APP后再启动即可，消息不不会丢失，如果配置了推送，关闭通道后消息会以推送的形式提醒，2.8.2以后版本已做了相关优化可能以后也可能会被系统检测到直接清理掉。
具体配置如下： 

```

- (void)applicationDidEnterBackground:(UIApplication *)application {
  [[ZCLibClient getZCLibClient] closeIMConnection];
  [[ZCLibClient getZCLibClient] removeIMAllObserver];
}
- (void)applicationDidBecomeActive:(UIApplication *)application {
     [[ZCLibClient getZCLibClient] checkIMConnected];
      [[ZCLibClient getZCLibClient] checkIMObserverWithRegister];
 }
```
  
  
  ###7 导航栏相关问题：
* 相似问题：   
	出现了两种导航栏？
	导航栏高度不正确？
	智齿聊天页面显示不全？
	* 消息列列表向下偏移，出现两种导航栏   
1.集成的项⽬目中有设置    self.navigationController.navigationBar.translucent= YES; 
解决⽅方法 self.navigationController.navigationBar.translucent= NO; 
2.启动VC的视图将要消失的⽅方法中有设置 导航栏显隐的操作 
解决⽅方法 启动VC 的视图将要消失的⽅方法中有设置 导航栏显影的操作如果在启动SDK时设置系统导航栏的隐藏，请在启动VC的视图将要出现的⽅方法中保持 导航栏隐藏。 
3.集成项⽬目中有使⽤用第三⽅方的navc管理理库或者重写navc 延展等   解决⽅方法 使⽤用第三⽅方 UINavigationController+FDFullscreenPopGesture 引起的适配问题 解决⽅方法如下: 
在fd_viewWillAppear:(BOOL) animated中，判断当前Controller为ZCChatController设置

```
#import <SobotKit/ZCChatController.h> #import <SobotKit/ZCUIBaseController.h> 
    [self.navigationController setNavigationBarHidden:YES animated:NO];
```
同理在 fd_viewWillDisappear:(BOOL) animated中添加还原自己的配置

```
[self.navigationController setNavigationBarHidden:NO animated:NO];

```4.启动SDK时 直接设置系统导航栏隐藏SDK内部将使用自定义View为导航栏 

```//2.5.0到2.5.5的版本进⼊入SDK时设置   self.navigationController.navigationBarHidden = YES; //2.6.0的版本 设置ZCKitInfo 的navcBarHidden 属性 
kitInfo.navcBarHidden = YES; //SDK 内部不不在使⽤用系统导航栏，使⽤用SDK⾃自定义View做导航栏。 //5.修改导航栏的UI // kitInfo.customB-nnerColor = [UIColor redColor]; // 顶部⽂文字的颜⾊色 // kitInfo.topViewTextColor = [UIColor redColor]; ```8、如何自定义UI 智齿SDK开放了丰富的可以配置UI选项，所有的选项都在ZCKitInfo.h中配置，详情说明可参加智齿接入文档或直接在ZCKitInfo中查看相关属性注释。如果ZCKitInfo开放的属性无法满足你的需求，你也可以再智齿SDK接入文档最下面下载UI源码自行修改后编译使用。
**[说明：从2.8.0以后，替换图片可以直接在项目中放一个同名图片即可，内部加载会优先使用项目中的图片资源，如果项目中没有会去智齿bundle中加载]**
ZCKitInfo.h配置方式如下：

```
ZCKitInfo *uiInfo=[ZCKitInfo new];
uiInfo.isOpenRecord = NO;

```###9、留言配置 
* 相似问题：   
	不显示留言？  
	无法跳转留言？
	如果监听留言点击事件？   
 
首先开启留言功能*“设置->在线->留言设置->留言功能”*. 

* 留言转工单 
 	默认留言都是提交一条功能，可添加相对复杂的配置，如图片、文件、自定义字段。
 	
* 转离线消息
	留言后直接给客服发送了一条离线消息，模板相对简单，只能显示普通文本。如果SDK没有显示留言功能，首先检查当前是否开启了留言功能；如果自定义字段无法显示，判断是否设置了留言转离线消息；如果点击留言没有任何反应，请检查是否设置留言点击监听，拦截了留言点击事件。

SDK相关配置：  
如果想自己实现留言页面，sdk中留言可跳转到自定义页面，如有此需求，可以使用如下方法进行设置：在启动支持客服时实现<ZCChatControllerDelegate>代理 ，其它target就是要实现的代理对象，如果没有需要直接传nil即可：

```[ZCSobot startZCChatVC:uiInfo with:self target:vc pageBlock:^(ZCChatController *object, ZCPageBlockType type) { } messageLinkClick:^(NSString *link) {}];实现代理方法：-(void)openLeaveMsgClick:(NSString*)tipMsg{// 点击留言的代理事件    NSLog(@"tipMsg ==%@",tipMsg);}

```###10、转人工失败
* 相似问题：   
	无法转接人工？
	总是提示无客服在线？
排查过程：
1、确定当前是否有客服在线    2、指定了客服ID，所对接的客服不在线 （忙碌、离线、不在上班时间）3、指定的技能组中没有客服在线 忙碌、离线、不在上班时间）  4、设置了溢出，但是溢出的技能组中没有客服在线 （忙碌或者离线或者不在上班时间）###11、留言记录无法查看
* 相似问题：   
	加载不到留言记录？
请使用2.8.2及以后版本，第一次留言记录提交由于需要同步用户信息可能有2秒左右的延迟，所以创建完立即查看可能没有数据。

###12、留言记录有别人的记录
* 相似问题：   
	留言记录串消息？
	
智齿客户中心判断客户的唯一标准包含用户对接ID、电话和邮箱，3者任意一个相同都会合并为同一个用户，当发现留言记录的消息相同是，确定以上信息是否有相同的。###13、无法进入留言* 相似问题：   
	留言点击无效？1 如果设置了自定义留言代理，如果拦截了点击事件，将会点击留言会执行以下代码：

```/** 点击留言跳转到用户自己留言页面，SDK将不在处理留言 @param tipMsg 提示语 */-(void)openLeaveMsgClick:(NSString*)tipMsg;```
2 SDK进入留言页面之前会调用接口模板信息，如果调用留言模板接口加载异常也不会进入留言界面，如果不能排除此问题，请联系人工客服解决。###14、如何设置工单组
* 相似问题：   
	工单组和技能组有什么区别？   

技能组为在线客服接待技能组，工单组为工单客服的工作组；SDK默认没有工单业务，但是留言转工单就默认添加了一条工单，所以参与到工单业务中。SDK目前无法指定工单组，但是指定接待技能组，会默认同步到工单技能组。在客服工作台*“设置 -> 工单技能组设置 -> (添加技能组 和 添加组成员)”*

###15、如何显示固定呼叫电话
* 相似问题：   
	怎么设置固定拨打电话号码？ 	
	怎么设置结束会话？
	怎么显示评价按钮？聊天右上角 可以显示 关闭，拨号，评价，更多，四种按钮，其中可以配置 拨号，评价，关闭 ，三种按钮， 默认一直显示“更多”按钮；拨号，评价 按钮由于位置原因，如果同时开启，拨号按钮优先显示，关闭按钮只能在人工模式下显示，点击关闭会直接离线用户。从左到右的显示顺序是：关闭、拨号、评价和更多。配置方式如下：

```

    
ZCKitInfo *uiInfo=[ZCKitInfo new];
// 导航栏左上角 是否显示 关闭按钮 默认不显示，关闭按钮，点击后无法监听后台消息 
uiInfo.isShowClose = YES;
//针对关闭按钮，单独设置是否显示评价界面，默认不显示
uiInfo.isShowCloseSatisfaction = YES;


// 导航栏右上角 是否显示 评价按钮  默认不显示
uiInfo.isShowEvaluation = YES;

// 导航栏右上角 是否显示 拨号按钮 默认不显示    
uiInfo.isShowTelIcon = YES;
// 要拨打的电话号码
uiInfo.customTel = @"4003095678";
        

```
###16、初始化和启动页面的作用和区别
* 相似问题：   
	进入SDK就闪退回来？ 	
	什么调用初始化？	
	可以多次调用初始化吗？
初始化是为了验证客户appkey是否正常，从2.7.2版本开始使用；为了提升启动页面的体验专门分离出来的接口，所以2.7.2以后的版本如果直接调用直接导致后续接口调用失败，返回到上一页面。一般是在 AppDelegate 中应用加载时调用，或者在调用启动页面接口之前调用，可根据初始化的block是否返回成功再决定是否启动SDK。

```
    [[ZCLibClient getZCLibClient] initSobotSDK:@"appkey" result:^(id object) {
       // 初始化结果
    }];
```启动SDK页面
启动页面会上传 ZCKitInfo.h的UI相关参数，ZCLibInitInfo接口配置信息为一个全局的单例，配置一次全局生效，启动SDK接口如下：

```
[ZCSobot startZCChatVC:uiInfo with:object target:nil pageBlock:^(id object, ZCPageBlockType type) {
            if([object isKindOfClass:[ZCChatView class]] && type == ZCPageBlockLoadFinish){
                UITextView *tv = [((ZCChatView *)object) getChatTextView];
                
            }
        } messageLinkClick:^BOOL(NSString *link) {
            

            
            return NO;
        }];
        

```
 ###17、如何配置引导语
* 相似问题：   
	怎么配置机器人引导语？   
	为什么不显示人工欢迎语？   机器人引导语：*“设置->机器人->机器人信息->常见问题引导”*开启常见问题引导后，客户将在接入机器人后按顺序收到系统欢迎语及常见问题引导两条消息机器人欢迎语：*“设置->在线->会话自动应答->APP”*
开启后，客户接入机器人会话，系统将使用此说辞作为欢迎语

人工欢迎语：*“设置->在线->会话自动应答->APP”*
开启后，客户接入人工客服会话，系统将使用此说辞作为欢迎语###18、如何配置通告
* 相似问题：   
	通告为啥不可点击？   
	什么是通告？   
	
通告为一条独立的提醒消息，醒目的展示在聊天页面；可配置是否永远置顶在聊天页面的最上面，App端默认置顶显示且显示为单行公告提示，设置方式如下：*“设置->在线->会话自动应答->APP”*；
如果希望通告可以跳转展示更多的内容，请在配置的时候添加相关的跳转链接。###19、如何配置自动提醒
* 相似问题：   
	有哪些自动提醒？
	自动提醒不显示？
	提示不显示？
	自动提示不显示？

智齿SDK支持多种提示文案，设置方式如下：*“设置->在线->会话自动应答->APP”*：
* 未知问题说辞
机器人客服无法理解客户的问题时，回复此说辞* 客服不在线说辞
开启后，人工客服不在线时，系统将回复此说辞* 客户排队说辞 开启后，客户转人工咨询遇到排队时，将提示此说辞；如需要在提示中显示排队数，请输入“#人数#”调取，例如您在队伍中的第#人数#个
* 排队超时接入说辞 
开启后，普通客户转人工咨询遇到排队等待超时后，将优先接入，并提示此说辞；在提示中输入“#人数#”显示排队数，输入“#时长#”则显示等待时长
* 客服结束会话说辞 
开启后，客服主动结束会话，将提示此说辞；如需要在提示中显示接待客服名称，请输入“#”调取，例如#客服#结束了本次会话* 客户超时提示 开启后，当客户超过设置的时长未回复时，系统将自动给客户推送此说辞
* 客服超时提示 开启后，当客服超过设置的时长未回复时，系统将自动给客户推送此说辞您还可以设置客服超时转接，点击设置-在线客服分配-超时分配进行客服超时设置
* 客户超时下线 当客户超过设置的时长未回复时，系统将结束该次会话。 人工接待时，会同步给客户推送此说辞﻿###20、多机器人配置
* 相似问题：   
	如何制定机器人？
	如何开启多机器人？

指定机器人：
根据不同的业务直接指定具体的机器人编号(获取机器人编号*“设置->机器人信息->
robotFlag=xx”*)；设置代码如下：

```
ZCLibInitInfo *initInfo = [ZCLibClient getZCLibClient].libInitInfo;
initInfo.robotid = @"";

```	


多机器人：
首先在智齿工作台配置多机器人选项*“设置->APP->客服设置(选择创建的应用)->切换接待机器人（开启）”*，开启后在机器人接入是会默认显示切换机器人按钮。


###21、消息撤回* 相似问题：   
	为什么没有消息撤回功能？
	消息撤回不生效？消息撤回功能需要在工作台*“设置->在线->客服工作台设置->人工消息撤回”*,开启后，客服可撤回已发送2分钟以内的消息，客户将不能看到已撤回的消息。
当客服撤回消息以后，SDK端会显示客服xxx撤回了一条消息。###22、未读消息数* 相似问题：   
	未读消息获取不到？
	未读消息不准备？
	怎么获取未读消息？

智齿SDK的未读消息，目前仅限于人工客服聊天时才有；当用户正在与人工客服沟通离开聊天页面了，SDK的长连接还是会实时获取到客服发送的消息；同时会从客户离开聊天页面开始计算未读消息数量。由于消息数为本地临时存储，所以每次应用kill后重新启动消息数会清零。
在ZCLibClient.h中如何获取未读消息数

```**获取未读消息数@return 未读消息数(进入智齿聊天页面会清空)*/-(int) getUnReadMessage;获取离线消息的回调中，也能获取未读消息： [ZCLibClient getZCLibClient].receivedBlock = ^(id obj,int unRead,NSDictionary *object){//        NSLog(@"当前消息数：%d \n %@",unRead,obj);    };    清空未读消息：/** 清空用户下的所有未读消息(本地清空) @param userId 接入的用户ID */-(void) clearUnReadNumber:(NSString *) partnerid;```


###23、如何配置询前表单
* 相似问题：   
	如何单独关闭询前表单？
	询前表单：
询前表单功能用于转人工咨询前的客户信息收集，将收集的信息应用到客服工作台；开启后，客户转人工前必须填写一份表单才能接入人工咨询。


开启询前表单：*“设置->在线->询前表单设置”*，如果仅仅想关闭SDK转人工是的询前表单，在2.8.0以后版本，提供如下配置实现：

```
ZCKitInfo *kitInfo = [ZCKitInfo new];
kitInfo.isCloseInquiryForm = YES;
```
24、如何使用客户服务中心

* 相似问题：   
	什么是客户服务中心？
	怎么进入客户服务中心？
	

客户服务中心：客户在发起咨询前，可看到配置的客户服务中心页，帮助客户快速解决问题，减少人工客服的咨询压力；
开启客户服务中心：*“设置->APP->客服设置->选择创建的应用（设置）->客户服务中心”*
SDK进入客户服务中心配置：

```

ZCKitInfo *kitInfo = [ZCKitInfo new];
    [ZCSobot openZCServiceCentreVC:kitInfo with:self onItemClick:^(ZCUIBaseController *object, ZCOpenType type) {
    });

```
###25、如何设置文字颜色

* 相似问题：   
	如何设置文字颜色？
	自定义颜色不满足怎么办？
	SDK启动聊天页面可以设置ZCKitInfo.h的相关属性实现自定义UI的样式，详情见接入文档或直接查看项目ZCKitInfo.h注释说明，如果ZCKitInfo中的属性不能完全满足，需求单独提给我们需求，我们评估后添加，也可以在接入文档中下载我们的UI源码自己编译修改；
SDK中加载图片优先在项目中查找，如果项目中没有才会加载bundle中的图片资源，所以如果想替换图片可以除了可以直接替换bundle中的图片资源外，也可以放一个同名图片在项目中达到目的，常见颜色属性如下：```
ZCKitInfo *kitInfo = [ZCKitInfo new];
// 提交评价按钮的文字颜色
kitInfo.submitEvaluationColor = UIColor.redColor;
//顶部文字颜色
//topViewTextColor//左边气泡文字颜色//leftChatTextColor//右边气泡文字颜色//rightChatTextColor//时间文字的颜色//timeTextColor


//左边气泡中链接颜色//chatLeftLinkColor//右边气泡中链接颜色//chatRightLinkColor

```###26、如何替换相关图片

* 相似问题：   
	使用pod集成，如何替换图片？
	  加SDK中加载图片优先在项目中查找，如果项目中没有才会加载bundle中的图片资源，所以如果想替换图片可以除了可以直接替换bundle中的图片资源外，也可以放一个同名图片在项目中达到目的；
###27、如果支持多语言
* 相似问题：   
	支持哪些语言？
	没有我要的语言？智齿iOS SDK的多语言文件在SobotKit.bundle中，目前已包含中文、英文、繁体中文、阿拉伯语，所有语言文件是通过获取系统语言前缀加上“_lproj”命名文件夹，然后把需要的翻译文件已固定名称“SobotLocalizable.strings”放入对应的文件夹中即可，例如英文“SobotKit.bundle/en_lproj/SobotLocalizable.strings”，默认不识别语言统一使用英文，如果需要扩展自己的语言，按照以上规则添加对应的翻译文件放入指定的位置即可。
###28、返回SDK时如何触发评价 
* 相似问题：   
	评价完如何结束会话？   
	返回评价？   
	关闭评价？    

目前SDK支持配置返回时弹出评价和关闭页面弹出评价，详细配置如下：

```

ZCKitInfo *kitInfo = [ZCKitInfo new];
// 返回时是否开启满意度评价,默认NO
kitInfo.isOpenEvaluation = YES;
// 评价完人工是否关闭会话,默认NO
kitInfo.isCloseAfterEvaluation = YES;
    
//导航栏左上角 是否显示 关闭按钮
kitInfo.isShowClose = YES;
//针对关闭按钮，单独设置是否显示评价界面，默认不显示
kitInfo.isShowCloseSatisfaction = YES;
    
```


29、如何自定义键盘中的功能按钮* 相似问题：   
	如何添加+中的按钮？   
	如何按钮点击事件？   
	键盘中的功能按钮(点击输入框最右侧+图标显示的按钮)，目前只支持扩展不支持删除。默认根据配置机器人会有“留言”、“评价”；人工会有：“图片”、“拍摄”、“文件（iOS12以上才有）”、“留言”、“评价”；扩展配置如下：

```

 NSMutableArray *arr = [[NSMutableArray alloc] init];
    for (int i = 0; i<1; i++) {
        ZCLibCusMenu *menu1 = [[ZCLibCusMenu alloc] init];
        menu1.title = [NSString stringWithFormat:@"订单"];
        menu1.url = [NSString stringWithFormat:@"sobot://sendOrderMsg"];
        menu1.imgName = @"zcicon_sendpictures";
        [arr addObject:menu1];
        
        ZCLibCusMenu *menu2 = [[ZCLibCusMenu alloc] init];
        menu2.title = [NSString stringWithFormat:@"商品"];
        menu2.url = [NSString stringWithFormat:@"sobot://sendProductInfo"];
        menu2.imgName = @"zcicon_sendpictures";
        
        [arr addObject:menu2];
    }
    
    添加人工配置
    uiInfo.cusMoreArray = arr;

/**
 *  自定义机器人快捷入口
 *  填充内容为： NSDictionary
 *  url: 快捷入口链接(点击后会调用初始化linkBock)
 *  title: 按钮标题
 *  lableId: 自定义快捷入口的ID
 *
 **/
@property (nonatomic,strong) NSMutableArray * cusMenuArray;



/**
 * 自定义输入框下方更多(+号图标)按钮下面内容(不会替换原有内容，会在原有基础上追加)
 * 修改人工模式的按钮
 * 填充内容为：ZCLibCusMenu.h
 *  title:按钮名称
 *  url：点击链接(点击后会调用初始化linkBock)
 *  imgName:本地图片名称，如xxx@2x.png,icon=xxx
 */
@property (nonatomic,strong) NSMutableArray * cusMoreArray;


```
###30、添加标签链接
* 相似问题：   
	什么是标签链接？   
	标签链接怎么监听？   

标签链接(*"设置->APP->客服设置->添加标签链接"*)，开启后在咨询端输入框上方出现设置的标签，客户可以点击标签快速查看对应内容；
iOS SDK所有链接监听，均在启动页面的block中实现；具体如下：

```
[ZCSobot startZCChatVC:uiInfo with:object target:nil pageBlock:^(id object, ZCPageBlockType type) {
            if([object isKindOfClass:[ZCChatView class]] && type == ZCPageBlockLoadFinish){
                UITextView *tv = [((ZCChatView *)object) getChatTextView];
                
            }
        } messageLinkClick:^BOOL(NSString *link) {
            
			//在这里监听链接点击，如果自己实现就返回YES，否则返回NO
            
            return NO;
        }];
        

```


###31、2.7.14和2.8.2有什么区别
* 相似问题：   
	为什么要升级最新版本？   
	旧版本还可以使用吗？

智齿对已经发布的SDK版本始终能提供接口服务，但是由于系统升级和外部环境变化，老版本SDK无法始终支持新的手机软件环境，比如iOS13的推送适配，刘海屏的出现、暗黑模式的升级、UIWebView的过时等问题，新版本会随时适配最新的技术变更，所以最好能使用最新的SDK版本。

由于在2.8.0之后的版本做了UI重构和对接参数统一规范，在2.8.0之后的版本会使用新的UI和参数名称；对于已经使用的用户不管从体验和对接都有很大变化，所以现在SDK会有2个版本同时发布维护，后期会合并为一个版本。
* 主要不同：1 UI风格改版，比如会话气泡、各种图标、颜色等；  2 顶部标题变化，默认只显示头像，没有文字标题(可配置显示)；  3 聊天框不在显示头像和名称；  

###32、如何设置录音开关，实现的流程是什么
* 相似问题：   
	如何关闭机器人语音？     
	机器人语音无法识别？     
	如何关闭人工语音？    
	语音有哪些限制？     

语音功能包含语音的播放和录音发送功能，所以需要用到系统的麦克风权限，否则录音功能无法正常使用。

* 机器人语音
	机器人语音需要单独付费购买,在统计质检中会有对比翻译，如果没有购买机器人语音，所有的机器人语音将无法正常返回结果；如果确认没有购买机器语音建议关闭机器人语音功能，代码配置如下：
	
```

ZCKitInfo *uiInfo=[ZCKitInfo new];
// 默认NO ,开启机器人语音，（付费，否则语音无法识别）
uiInfo.isOpenRobotVoice = NO;

```
	
* 人工语音
	人工语音是默认可用功能，只要是人工接待状态均可发送语音；发送的语音客服可以直接收听。如果不想让人工看到语音文件，可以在SDK端配置关闭语言功能，SDK端将不再显示发送语音按钮，代码配置如下：
	
```

ZCKitInfo *uiInfo=[ZCKitInfo new];
// 默认YES ,是否可以发送人工语音
uiInfo.isOpenRecord = NO;

```
		
 
###33、如何使用地理位置信息* 相似问题：   
	怎么发送位置？        
	如何获取坐标？       
	会不会影响系统审核？    

智齿SDK支持客户发送地理位置类型消息，客户可通过SDK实现消息展示和点击监听；由于地理位置需要敏感权限信息，智齿不会主动获取地址位置信息，使用客户定位插件不一样，智齿希望灵活使用客户的定位方式和选择位置功能，所以需要自行传递位置到SDK以及打开显示。


配置发送按钮，添加发送按钮后转接到人工后会自动显示在键盘区域，代码如下：

```

ZCKitInfo *uiInfo=[ZCKitInfo new];
// 默认NO ,人工状态，是否可以发送位置
uiInfo.canSendLocation = YES;



```

监听发送获取地理位置发送一条位置信息，监听位置点击事件，跳转到位置页面，代码如下：

```
[ZCSobot startZCChatVC:uiInfo with:object target:nil pageBlock:^(id object, ZCPageBlockType type) {
            if([object isKindOfClass:[ZCChatView class]] && type == ZCPageBlockLoadFinish){
                UITextView *tv = [((ZCChatView *)object) getChatTextView];
                
            }
        } messageLinkClick:^BOOL(NSString *link) {
				//在这里监听链接点击，如果自己实现就返回YES，否则返回NO
            // 当收到link = sobot://sendlocation 调用智齿接口发送位置信息            // 当收到link = sobot://openlocation?latitude=xx&longitude=xxx&address=xxx 可根据自己情况处理相关业务   
        		// 发送位置，可以参考接入文档代码
             if( [link hasPrefix:@"sobot://sendlocation"]){
             		// 发送位置信息                [ZCSobot sendLocation:@{                                        @"lat":@"40.001693",                                        @"lng":@"116.353276",                                        @"localLabel":@"北京市海淀区学清路38号金码大厦",                                        @"localName":@"云景四季餐厅",                                        @"file":fullPath}];
             		return YES;
             }    
             else if([link hasPrefix:@"sobot://openlocation"]){                // 解析经度、纬度、地址：latitude=xx&longitude=xxx&address=xxx                // 跳转到地图的位置//                NSLog(link);                // 测试打开地图 高德网页版                NSString *urlString = @"";                urlString = [[NSString stringWithFormat:@"http://uri.amap.com/marker?position=%@,%@&name=%@&coordinate=gaode&src=%@&callnative=0",@116.353276,@40.001693,@"北京市海淀区学清路38号金码大厦A座23层金码大酒店",@"智齿SDK"] stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];                [[UIApplication sharedApplication]openURL:[NSURL URLWithString:urlString]];                return YES;            }        
            return NO;
        }];
        

```

###34、如果使用商品或订单卡片信息* 相似问题：   
	如何发送商品信息？        
	如何自动发送商品信息？       
	自动发送订单信息？          
	发送订单信息？    
	发送商品信息？  
	
* 自动发送
	SDK支持配置自动发送商品和订单信息，在人工成功接待以后主动发送一条配置信息给人工客服，代码实现如下：
	
```

ZCKitInfo *uiInfo=[ZCKitInfo new];

// 配置商品信息
ZCProductInfo *productInfo = [ZCProductInfo new];
productInfo.thumbUrl = @"https://static.sobot.com/chat/admins/assets/images/logo.png";
productInfo.title = @"商品名称";
productInfo.desc = @"描述描述描述描述描述描述";
productInfo.label = @"标签标签标签标签标签标签标签";
productInfo.link = @"www.sobot.com";

// 设置发送信息
uiInfo.productInfo = productInfo;
// 商品卡片信息是否自动发送,默认NO（转人工成功时，自动发送商品卡片信息）
uiInfo.isSendInfoCard = YES;


// 配置订单信息
ZCOrderGoodsModel *model = [ZCOrderGoodsModel new];
model.orderStatus = 1;
model.createTime = [NSString stringWithFormat:@"%.0f",[[NSDate date] timeIntervalSince1970]*1000];
model.goodsCount = @"3";
model.orderUrl  = @"https://www.sobot.com";
model.orderCode = @"1000234242342345";
model.goods =@[@{@"name":@"商品名称",@"pictureUrl":@"http://pic25.nipic.com/20121112/9252150_150552938000_2.jpg"},@{@"name":@"商品名称",@"pictureUrl":@"http://pic31.nipic.com/20130801/11604791_100539834000_2.jpg"}];
    
model.totalFee = @"4890";

// 设置发送信息
uiInfo.orderGoodsInfo = model;

// 转人工后，是否主动发送一条信息，默认NO
uiInfo.autoSendOrderMessage = YES;


```
	
	
* 手动发送   
由于商品信息和订单信息只有人工客服才能识别，所以目前仅能对人工发送（虽然2.8.2版本以前开放了发送接口但是不能准确监听当前是不是人工客服接待，可能会导致发送失败；2.8.2以后可以通过ZCServerConnectStatus状态的监听调用发送接口实现）；通过配置发送按钮，在人工状态下会自动显示，监听点击事件可以实时改变发送的内容，配置如下：

```

// 配置发送按钮
ZCKitInfo *uiInfo=[ZCKitInfo new];

NSMutableArray *arr = [[NSMutableArray alloc] init];
for (int i = 0; i<1; i++) {
    ZCLibCusMenu *menu1 = [[ZCLibCusMenu alloc] init];
    menu1.title = [NSString stringWithFormat:@"订单"];
    // 此处配置的链接和监听的时候对应
    menu1.url = [NSString stringWithFormat:@"sobot://sendOrderMsg"];
    menu1.imgName = @"zcicon_sendpictures";
    [arr addObject:menu1];
    
    ZCLibCusMenu *menu2 = [[ZCLibCusMenu alloc] init];
    menu2.title = [NSString stringWithFormat:@"商品"];
    // 此处配置的链接和监听的时候对应
    menu2.url = [NSString stringWithFormat:@"sobot://sendProductInfo"];
    menu2.imgName = @"zcicon_sendpictures";
    
    [arr addObject:menu2];
}
    
// 注意一定要添到正确的扩展序列
uiInfo.cusMoreArray = arr;


// 监听发送
[ZCSobot startZCChatVC:uiInfo with:object target:nil pageBlock:^(id object, ZCPageBlockType type) {
    if([object isKindOfClass:[ZCChatView class]] && type == ZCPageBlockLoadFinish){
        UITextView *tv = [((ZCChatView *)object) getChatTextView];
        
    }
} messageLinkClick:^BOOL(NSString *link) {
		//在这里监听链接点击，如果自己实现就返回YES，否则返回NO
            // 当收到link = sobot://sendOrderMsg 调用智齿接口发送订单信息            // 当收到link = sobot://sendProductInfo调用发送商品信息
    if ([link hasPrefix:@"sobot://sendOrderMsg"]){                ZCOrderGoodsModel *model = [ZCOrderGoodsModel new];                model.orderStatus = 1;                model.createTime = [NSString stringWithFormat:@"%.0f",[[NSDate date] timeIntervalSince1970]*1000];                model.goodsCount = @"3";                model.orderUrl  = @"https://www.sobot.com";                model.orderCode = @"1000234242342345";                model.goods =@[@{@"name":@"商品名称",@"pictureUrl":@"http://pic25.nipic.com/20121112/9252150_150552938000_2.jpg"},@{@"name":@"商品名称",@"pictureUrl":@"http://pic31.nipic.com/20130801/11604791_100539834000_2.jpg"}];                model.totalFee = @"4890";                [ZCSobot sendOrderGoodsInfo:model];                return YES;            }            else if([link hasPrefix:@"sobot://sendProductInfo"]) {                ZCProductInfo *productInfo = [ZCProductInfo new];                productInfo.thumbUrl = @"https://static.sobot.com/chat/admins/assets/images/logo.png";                productInfo.title = @"标题标题标题标题标题标题";                productInfo.desc = @"描述描述描述描述描述描述";                productInfo.label = @"标签标签标签标签标签标签标签";                productInfo.link = @"www.baidu.com";                [ZCSobot sendProductInfo:productInfo];    
                return YES;                }       
    return NO;
}];


```
	
###35 pod 集成问题* 相似问题：   
	无法搜索到当前版本？      
	搜索到当前版本但是有缓存?      
	如何缓存清理?   
	pod版本如何替换图片?    
	
智齿的pod版本目前只提供带UI版本，普通版本SobotKit，电商版本SobotPlatform；pod版本可能会比官网更新的延迟2天左右；如果使用pod需要更换图片资源，可以放一个同名图片资源在项目中即可，会优先加载项目中的图片。      清理缓存

```// 如果搜索不到最新版本，请运行以下命令更新CocoPods仓库   pod repo update --verbose// 如果无法更新到最新版本，可以删除索引文件，重新尝试rm ~/Library/Caches/CocoaPods/search_index.json//清除pod缓存：//删除代码中的pod 文件夹，pod cache clean SobotKit//再重新构建
pod install

```


###36、APICloud版本何时更新
* 相似问题：   
	APICloud？   
	
APICloud版本需要对方审核，目前审核周期为一周；鉴于APICloud目前版本发布周期比较受限于审核时间，智齿SDK会上传目前较为稳定的版本，所以APICloud版本会比目前官网上的版本更新的慢，如果发现问题或需要使用新的功能，可联系我们技术评估上传。 
###37、什么是离线消息
* 相似问题：   
	离线消息？
	收不到消息?    

智齿SDK现在可实现完全非浸入使用，不咨询时可以完全关闭智齿服务。如果需要不咨询时也能收到客服的消息，如果此时会话未结束就是实时消息，如果此时会话已经结束既定义为离线消息。离线消息比较至少与客服建议过一次连接，SDK会保存上次已建立连接的地址，自动判断尝试连接；由于链接地址24小时未使用会失效，所以长时间没有与客服会话无法通过长连接收到离线消息。

代码相关使用说明：

```//检查当前消息通道是否建立，没有就重新建立， 如果调用 closeIMConnection 后，可调用此方法重新建立链接[[ZCLibClient getZCLibClient] checkIMConnected];  /* 检查当前监听是否被移除，如果移除就重新注册(重新激活 网络监听 ZCNotification_NetworkChange UIApplicationDidBecomeActiveNotification UIApplicationDidEnterBackgroundNotifica)，经常和checkIMConnected一起使用*/[[ZCLibClient getZCLibClient] checkIMObserverWithRegister];/** 移除IM所有监听，可能会影响应用在后台存活时长，如果调用此方法后需要checkIMObserverWithCreate重新激活 网络监听 ZCNotification_NetworkChange UIApplicationDidBecomeActiveNotification UIApplicationDidEnterBackgroundNotification */[[ZCLibClient getZCLibClient] removeIMAllObserver];/** @note 关闭当前消息通道，使其不再接受消息，如果配置推送了，消息会走apns送达 */   [[ZCLibClient getZCLibClient] closeIMConnection];      // ReceivedMessageBlock 未读消息数， obj 当前消息  unRead 未读消息数 [ZCLibClient getZCLibClient].receivedBlock = ^(id obj,int unRead,NSDictionary *object){//        NSLog(@"当前消息数：%d \n %@",unRead,obj);    };
        
```###38、如何离线用户
* 相似问题：      离线用户？    
  结束会话？    
  

根据客户使用场景，有时需要直接结束当前会话以减少排队和人工资源占用，可通过调用接口直接结束当前会话，离线用户。配置如下：

```
// 如果离线用户后，希望收到离线消息，可以参考离线消息配置

/**
 关闭通道，清理内存，退出智齿客户 移除推送
 说明：调用此方法后将不能接收到离线消息，除非再次进入智齿SDK重新激活,
 isClosePush:YES ,是关闭push；NO离线用户，但是可以收到push推送
*/[ZCLibClient  closeAndoutZCServer:NO];


```###39、如何自定义顶部标题 
* 相似问题：    
	自定义标题？    
	不显示名称？   
	   
2.8.0以前版本，支持3种模式的标题自定义，详细见下方注释说明：  

```
ZCLibInitInfo *initInfo = [ZCLibClient getZCLibClient].libInitInfo;
// 0.默认 ,自动判断显示机器人名称和人工名称，
// 1.始终企业名称
// 2.显示自定义标题，需要设置，customTitle
initInfo.titleType = @"0";

initInfo.customTitle = @"我是自定义标题，titleType=2是生效";


```


2.8.0以以后版本，支持5种模式的标题自定义，详细见下方注释说明：  


```
ZCLibInitInfo *initInfo = [ZCLibClient getZCLibClient].libInitInfo;
// 0.默认，自动判断显示机器人头像和人工头像
// 1.始终企业名称和企业logo
// 2.显示自定义标题和图片，需要设置，custom_title、custom_title_url
// 3.仅显示机器人或人工名称，不显示头像
// 4.自动判断，显示机器人、人工的头像和名称
initInfo.title_type = @"0";

// 仅当title_type=@"2"时以下配置有效
initInfo.custom_title = @"我是自定义标题，title_type=2是生效";
initInfo.custom_title_url = @"我是自定义标题，title_type=2是生效";
```###40、关键字转人工
* 相似问题：    
	关键字转人工？
	智能转人工？       

配置关键字转人工，可以省去客户使用时手动点击转人工的操作；智能转人工当机器人无法回复时自动转接人工，可以让客户无感知的情况下切换到人工客服。   

工作台配置：

设置关键词：*”设置->机器人->转人工关键词设置“*
开启关键词功能：*”设置->APP->客服设置（选择APP）->关键词转人工“*
智能转人工:*”设置->APP->客服设置（选择APP）->开启（机器人引导转人工、连续重复提问转人工、客户情绪负向转人工）“*

SDK 端配置均在ZCKitInfo.h中，说明如下：  
**isShowTansfer** ：默认YES，机器人优先模式，是否直接显示转人工按钮(值为NO时，会在机器人无法回答时显示转人工按钮)。     

**unWordsCount** ：机器人优先模式和isShowTansfer配合使用，通过记录机器人未知说辞的次数设置是否直接显示转人工按钮。  

**isOpenActiveUser** ：是否开启关键词转人工,(如输入“转人工”，直接转接人工)，需要隐藏转人工按钮，请参见isShowTansfer和unWordsCount属性。 
 
**activeKeywords**:智能转人工关键字配置，关键字作为key{@"转人工",@"1",@"R":@"1"}   




###41、智能路由
* 相似问题：  
	排队优先级设置？    
	优先接待？    

当用户排队是，可以指定客户优先接待；可以根据客户以往记录智能选择客服接待。详细配置和说明见：*”设置->在线->在线客服分配“* 

SDK端可设置指定客户排队时优先接入；设置如下：

```

ZCLibInitInfo *initInfo = [ZCLibClient getZCLibClient].libInitInfo;

/**
 *
 *  指定客户优先
 *  同PC端 设置-在线客服分配-排队优先设置-指定客户优先   开启传1 默认不设置
 *  开启后 指定客户发起咨询时，如果出现排队，系统将优先接入。
 **/
initInfo.queue_first = 1;

```

