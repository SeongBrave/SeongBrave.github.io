---
layout: post
title: "UICollectionView-方法说明"
date: 2015-06-30
comments: true
categories: IOS
tags: [IOS, UICollectionView]
published: ture
keywords: UICollectionView
description: UICollectionView
---
UICollectionView基本概念



####初始化:

{% highlight ruby %}

//初始化布局类(UICollectionViewLayout的子类)
UICollectionViewFlowLayout *fl = [[UICollectionViewFlowLayout alloc]init];

//初始化collectionView
self.collectionView = [[UICollectionView alloc]initWithFrame:CGRectZero collectionViewLayout:fl];

//设置代理
self.collectionView.delegate = self;
self.collectionView.dataSource = self;

{% endhighlight %}


#####需要实现的协议:

{% highlight ruby %}
UICollectionViewDataSource, UICollectionViewDelegateFlowLayout
PS:UICollectionViewDelegateFlowLayout是UICollectionViewDelegate的子协议

- (void)registerClass:(Class)cellClass forCellWithReuseIdentifier:(NSString *)identifier;
PS:如果是用nib创建的话，使用下面这个函数来注册。
- (void)registerNib:(UINib *)nib forCellWithReuseIdentifier:(NSString *)identifier;

如果需要显示每个section的headerView或footerView，则还需注册相应的UICollectionReusableView的子类到collectionView
elementKind是header或footer的标识符，只有两种可以设置UICollectionElementKindSectionHeader和UICollectionElementKindSectionFooter
- (void)registerClass:(Class)viewClass forSupplementaryViewOfKind:(NSString *)elementKind withReuseIdentifier:(NSString *)identifier;
PS:如果是用nib创建的话，使用下面这个函数来注册。
- (void)registerNib:(UINib *)nib forSupplementaryViewOfKind:(NSString *)kind withReuseIdentifier:(NSString *)identifier;
{% endhighlight %}

#####实现协议的函数:
跟UITableView的DataSource和Delegate很像，大可自行代入理解。
{% highlight ruby %}
//每一组有多少个cell
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section;

//定义并返回每个cell
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath;

//collectionView里有多少个组
- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView;

//定义并返回每个headerView或footerView
- (UICollectionReusableView *)collectionView:(UICollectionView *)collectionView viewForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath;

上面这个方法使用时必须要注意的一点是，如果布局没有为headerView或footerView设置size的话(默认size为CGSizeZero)，则该方法不会被调用。所以如果需要显示header或footer，需要手动设置size。
可以通过设置UICollectionViewFlowLayout的headerReferenceSize和footerReferenceSize属性来全局控制size。或者通过重载以下代理方法来分别设置
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForHeaderInSection:(NSInteger)section;
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForFooterInSection:(NSInteger)section;

{% endhighlight %}

#####Delegate:
跟UITableView的DataSource和Delegate很像，大可自行代入理解。
{% highlight ruby %}
//每一个cell的大小
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath;

//设置每组的cell的边界, 具体看下图
- (UIEdgeInsets)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout insetForSectionAtIndex:(NSInteger)section;
- //cell的最小行间距
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumLineSpacingForSectionAtIndex:(NSInteger)section;

//cell的最小列间距
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumInteritemSpacingForSectionAtIndex:(NSInteger)section;

//cell被选择时被调用
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath;

//cell反选时被调用(多选时才生效)
- (void)collectionView:(UICollectionView *)collectionView didDeselectItemAtIndexPath:(NSIndexPath *)indexPath;
{% endhighlight %}