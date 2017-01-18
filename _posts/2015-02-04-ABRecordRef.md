---
layout: post
title: "ABRecordRef获得通讯录中联系人"
date: 2015-02-04
comments: true
categories: iOS
tags: [iOS, ABRecordRef]
published: ture
keywords: ABRecordRef
description: ABRecordRef获得通讯录中联系人
---
ABRecordRef获得通讯录中联系人



###创建保存联系人得model

PersonModel

>PersonModel.h

{% highlight ruby %}
/**
 *  联系人姓名
 */
@property(nonatomic , strong)NSString *m_personName;

/**
 *  电话号
 */
@property(nonatomic , strong)NSString *m_phoneNum;

/**
 *  保存联系人多个电话号
 */
@property(nonatomic , strong)NSArray *m_phoneList;
{% endhighlight %}

Member的父类是`MTLModel`同时还遵守`<MTLJSONSerializing>`协议，查看这个协议你会发现里面有个必须实现的方法：
```
+ (NSDictionary *)JSONKeyPathsByPropertyKey;
```
这个方法就是前面提到的用于字段转换的，下面实现这个方法

>PersonModel.m
暂时为做处理


然后是通过联系人实例化 数组

最后用一句代码来得到你想要的模型


{% highlight ruby %}

-(NSMutableArray *)GetContactListData
{
 
 
  NSMutableArray *contactList = [[NSMutableArray alloc]init];
    ABAddressBookRef addressBook;
    if ([[UIDevice currentDevice].systemVersion floatValue] >= 6.0)    {
        addressBook = ABAddressBookCreateWithOptions(NULL, NULL);
        //等待同意后向下执行
        dispatch_semaphore_t sema = dispatch_semaphore_create(0);        ABAddressBookRequestAccessWithCompletion(addressBook, ^(bool granted, CFErrorRef error)                                                 {                                                     dispatch_semaphore_signal(sema);                                                 });
        dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    }else{
        addressBook = ABAddressBookCreate();
    }
     CFArrayRef results = ABAddressBookCopyArrayOfAllPeople(addressBook);
    for(int i = 0; i < CFArrayGetCount(results); i++)
    {
        PersonModel *model = [[PersonModel alloc]init];
        NSMutableDictionary *dict =[[NSMutableDictionary alloc]init];
        
        ABRecordRef person = CFArrayGetValueAtIndex(results, i);
        //读取firstname
        NSString *personName = ( NSString*)CFBridgingRelease(ABRecordCopyValue(person, kABPersonFirstNameProperty));
        //读取lastname
        NSString *lastname = (__bridge NSString*)ABRecordCopyValue(person, kABPersonLastNameProperty);
        //读取middlename
        NSString *middlename = (__bridge NSString*)ABRecordCopyValue(person, kABPersonMiddleNameProperty);
   
         model.m_personName = [NSString stringWithFormat:@"%@%@%@",lastname==nil?@"":lastname,personName==nil?@"":personName,middlename==nil?@"":middlename];
         NSMutableArray *telList = [[NSMutableArray alloc]init];
        //读取电话多值
        ABMultiValueRef phone = ABRecordCopyValue(person, kABPersonPhoneProperty);
        for (int k = 0; k<ABMultiValueGetCount(phone); k++)
        {
         //获取电话Label
            NSString * personPhoneLabel = (__bridge NSString*)ABAddressBookCopyLocalizedLabel(ABMultiValueCopyLabelAtIndex(phone, k));
            //获取該Label下的电话值
            NSString * personPhone = (__bridge NSString*)ABMultiValueCopyValueAtIndex(phone, k);

            [telList addObject:@{@"personPhoneLabel":personPhoneLabel,@"personPhone":personPhone}];
            model.m_phoneList = telList;
        }
        if (model.m_phoneList.count >0) {
            model.m_phoneNum =model.m_phoneList[0][@"personPhone"];
        }
        
        dict[@"phones"] =telList ;
        
        
        [contactList addObject:model];

    }
    
    CFRelease(results);
    CFRelease(addressBook);

    return contactList;
    
}
{% endhighlight %}

##代码说明

{% highlight ruby %}
 NSMutableArray *contactList = [[NSMutableArray alloc]init];
    ABAddressBookRef addressBook;
    if ([[UIDevice currentDevice].systemVersion floatValue] >= 6.0)    {
        addressBook = ABAddressBookCreateWithOptions(NULL, NULL);
        //等待同意后向下执行
        dispatch_semaphore_t sema = dispatch_semaphore_create(0);         ABAddressBookRequestAccessWithCompletion(addressBook, ^(bool granted, CFErrorRef error)                                                 {                                                     dispatch_semaphore_signal(sema);                                                 });
        dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    }else{
        addressBook = ABAddressBookCreate();
        {% endhighlight %}
    以上代码是实例化一个ABAddressBookRef 对象
