---
layout: post
title: "Autolayout实现UITableView的Cell高度"
date: 2015-07-1
comments: true
categories: IOS
tags: [IOS, Autolayout，UITableViewCell]
published: ture
keywords: UITableViewCell
description: UITableViewCell
---
UITableView核心概念:

######不管你是在哪个iOS版本上做开发，以下步骤中的前两个步骤都是必须的:

#####1、设置好布局约束条件
      
       在UITableViewCell子类中，添加布局约束，使得cell子视图的边缘固定(pin)到cell的contentView的边缘（最重要的是要有顶部和底部的边距约束条件）。注意：不要将子视图的边距约束固定到cell本身上了，只能固定到cell的contentView上！ 确保每个子视图垂直方向上的内容压缩阻力(compression resistance)和吸附性约束(hugging constraints)没有被你添加的更高优先级的约束条件覆盖，让这些子视图的固有内容尺寸(intrinsic content size)来驱动contentView的高度。（没看懂？点这里。）
      记住，要点是让cell的子视图与contentView之间产生垂直的连结，让它们能够对contentView“施加压力”，使contentView扩展以适合它们的尺寸。下面用一个cell和一些子视图作为示例，展示了你的一些(不是全部！)布局约束应该看起来是什么样的：
  ![1](/images/Autolayout/a.jpg)
  
    可以设想，随着更多的文本被添加到上例中“Multi-line body”那个label上，它需要垂直地增高以适合文本，这将有效地迫使cell的高度增加。（当然，你需要正确地设置约束条件，以使其正常的工作！）
    如何设置正确的约束条件，绝对是使用Autolayout实现动态行高时最难最重要的部分。如果弄错了，它就可能无法正常工作——所以，不要着急，慢慢来！我建议你用代码来设置布局约束，这样你就完全知道每个布局约束被加到了什么地方，出问题时也更容易调试。特别如果使用一些优秀开源库，可以让用代码设置约束和用Interface Builder设置约束一样简单直观，并且功能还更强大。这里也有一个由我设计和维护的专用库：
  [PureLayout](https://github.com/smileyborg/PureLayout)
  
 * 如果你用代码来设置布局约束，你应该在UITableViewCell子类的updateConstraints方法里面一次性完成。注意，updateConstraints可能不止被调用一次，因此要避免重复添加相同的布局约束。在updateConstraints中，可以将添加布局约束的代码包在一个if条件语句中(比如用一个叫didSetupConstraints的布尔属性，运行一次添加布局约束的代码后就将其设置为YES)，以确保不重复添加相同的布局约束。另外，更新已有布局约束的代码（比如调整布局约束的constant属性），也应该将它们放置在updateConstraints 中，但是要在didSetupConstraints条件语句的外面，这样才可以确保每次调用的时候都会被执行。
 
 
#####2、 Cell使用具有唯一性的重用标示符
  在cell里面，为每一组特定的约束条件，使用一个特定的cell重用标示符。换句话说，如果cell有多种不同的布局，每一种布局应当有其对应的重用标示符。（当cell有多种不同数量的子视图的时候，或者子视图以一种独特的方式布局的时候，这些情况下你就需要使用一个新的重用标示符。）

例如，要在一个cell中显示一条email消息，可能会有4种独特的布局：第一种，只有主题的消息；第二种，带主题和正文的消息；第三种，带主题和图片附件的消息；第四种，带主题、正文和图片附件的消息。每一种布局都需要完全不同的布局约束才能实现。因此，一旦cell被初始化并且布局约束被加到其中任意一种类型的cell上后，cell应当得到一个唯一的重用标示符来指定该cell类型。这就意味着，当你dequeue重用一个cell的时候，该类型cell的布局约束已经添加好了，可以直接使用。

注意，由于固有内容尺寸的不同，具有相同布局约束的cell仍然可能具有不同的高度！不要混淆了布局（不同的约束）和由不同内容尺寸而计算出（通过相同的布局约束来计算）的不同视图frame这两个概念，它们根本是完全不同的两个东西。
 
 * 不要将拥有不同布局约束条件的cell丢到同一个重用池当中（也就是使用相同的重用标示符），然后又在每次dequeue过后将旧的约束移除，又从头开始重新添加约束。自动布局引擎内部并没有被设计来可以处理大规模的约束更改，你会看到大量的性能问题。 
 
####iOS8 - Self-Sizing Cells


#####3、 启用估算行高

 在iOS8上，苹果将许多在iOS8之前比较难实现的东西都内置实现了。为了让cell实现self-sizing的机制，必须先将tableView的rowHeight属性设置为常量UITableViewAutomaticDimension。然后，只需将tableView的estimatedRowHeight属性设置为非零值即可开启行高估算功能，例如：

 {% highlight ruby %}
 self.tableView.rowHeight = UITableViewAutomaticDimension;
 self.tableView.estimatedRowHeight = 44.0; // 设置为一个接近“平均”行高的值
 
 {% endhighlight %}
 
 这样做就为tableView上还没有显示在屏幕上的cell提供了一个临时的估算的行高。然后，当cell即将滚入屏幕范围内的时候，会计算出实际的高度。为了确定每一行的实际高度，tableView会自动让每个cell基于其contentView的已知固定宽度（tableView的宽度，减去其他额外的，像section index或accessoryView这些宽度）和被加到contentView及其子视图上的自动布局约束规则来计算contentView的高度。一旦真正的行高被计算出来后，旧的估算的行高会被更新为这个真实的行高（并且其他任何需要对tableView的contentSize或contentOffset的更改都自动替你完成了）。

一般来说，行高估算值不需要太精确——它只是被用来修正tableView中滚动条的大小的，当你在屏幕上滑动cell的时候，即便估算值不准确，tableView还是能很好地调节滚动条。将tableView的estimatedRowHeight属性设置成（在viewDidLoad或类似的方法中）一个接近于“平均”行高的常量值即可。只有行高变化很极端的时候（比如相差一个数量级），才会在滚动时产生滚动条“跳跃”的现象。这个时候，你才应当实现tableView:estimatedHeightForRowAtIndexPath:方法，为每一行返回一个更精确的估算值。

####iOS7支持（需要自己实现cell尺寸自适应功能）

#####3、 完成一个完整的布局过程 & 计算cell的高度
 首先，为每一个cell都初始化一个离屏(offscreen)实例，为每个重用标示符实例化一个与之对应的cell实例，这些cell完全用于高度计算。（离屏表示cell的引用被存储在view controller的一个属性或实例变量之中，并且这个cell绝对不会被用作tableView:cellForRowAtIndexPath:方法的返回值以实际呈现在屏幕上。）接着，这个cell的内容（例如，文本、图片等等）还必须和会被显示在table view中的内容完全一致。

然后，强制cell立即更新子视图的布局，再用cell的contentView调用systemLayoutSizeFittingSize:方法计算出cell所需的高度是多少。使用UILayoutFittingCompressedSize参数可以得到适合cell中所有内容所需的最小尺寸。然后其高度就可以作为tableView:heightForRowAtIndexPath:方法的返回值。
#####4、 使用估算的行高
如果你的table view超过了几十行，你会发现自动布局约束的解决方式在第一次加载table view的时候会迅速地卡住主线程。因为，在第一次加载过程中，每一行都会调用tableView:heightForRowAtIndexPath:方法（为了计算滚动条的尺寸）。

iOS7中，你可以（也绝对应该）使用table view的estimatedRowHeight属性。这样会为还不在屏幕范围内的cell提供一个临时估算的行高值。然后，当这些cell即将要滚入屏幕范围内的时候，真实的行高值会被计算出来（通过tableView:heightForRowAtIndexPath:方法），估算的行高就会被替换掉。

一般来说，行高估算值不需要太精确——它只是被用来修正tableView中滚动条的大小的，当你在屏幕上滑动cell的时候，即便估算值不准确，tableView还是能很好地调节滚动条。将tableView的estimatedRowHeight属性设置成（在viewDidLoad或类似的方法中）一个接近于“平均”行高的常量值即可。只有行高变化很极端的时候（比如相差一个数量级），才会在滚动时产生滚动条“跳跃”的现象。这个时候，你才应当实现tableView:estimatedHeightForRowAtIndexPath:方法，为每一行返回一个更精确的估算值。
#####5、 缓存行高（如果需要）
如果上面提到的你都做了，但是tableView:heightForRowAtIndexPath:的性能仍然慢的不可接受。非常不幸，你需要给行高做一些缓存（这是苹果的工程师们给出的改进建议）。大体的思路是，第一次计算时让自动布局引擎解析约束条件，然后将计算出的行高缓存起来，以后所有对该cell的高度的请求都返回缓存值。当然，关键还要确保任何会导致cell高度变化的情况发生时你都清除了缓存的行高——这通常发生在cell的内容变化时或其他重大事件发生时（比如用户调节了动态类型文本大小(Dynamic Type text size)的滑动条）。
######iOS7示例代码（包含详细的注释）
 
{% highlight ruby %}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{    
    // 判断indexPath对应cell的重用标示符，    
    // 取决于特定的布局需求（可能只有一个，也或者有多个）    
    NSString *reuseIdentifier = ...;    
     
    // 取出重用标示符对应的cell。    
    // 注意，如果重用池(reuse pool)里面没有可用的cell，这个方法会初始化并返回一个全新的cell，    
    // 因此不管怎样，此行代码过后，你会可以得到一个布局约束已经完全准备好，可以直接使用的cell。    
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:reuseIdentifier];    
     
    // 用indexPath对应的数据内容来配置cell，例如：    
    // cell.textLabel.text = someTextForThisCell;    
    // ...    
     
    // 确保cell的布局约束被设置好了，因为它可能刚刚才被创建好。    
    // 使用下面两行代码，前提是假设你已经在cell的updateConstraints方法中设置好了约束：
    [cell setNeedsUpdateConstraints];    
    [cell updateConstraintsIfNeeded];    
     
    // 如果你使用的是多行的UILabel，不要忘了，preferredMaxLayoutWidth需要设置正确。 
    // 如果你没有在cell的-[layoutSubviews]方法中设置，就在这里设置。    
    // 例如：    
    // cell.multiLineLabel.preferredMaxLayoutWidth = CGRectGetWidth(tableView.bounds);    
    return cell;
}
 
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{    
    // 判断indexPath对应cell的重用标示符，    
    NSString *reuseIdentifier = ...;    
     
    // 从cell字典中取出重用标示符对应的cell。如果没有，就创建一个新的然后存储在字典里面。
    // 警告：不要调用table view的dequeueReusableCellWithIdentifier:方法，因为这会导致cell被创建了但是又未曾被tableView:cellForRowAtIndexPath:方法返回，会造成内存泄露！    
    UITableViewCell *cell = [self.offscreenCells objectForKey:reuseIdentifier];    
    if (!cell) {        
        cell = [[YourTableViewCellClass alloc] init];        
        [self.offscreenCells setObject:cell forKey:reuseIdentifier];    
    }    
     
    // 用indexPath对应的数据内容来配置cell，例如：    
    // cell.textLabel.text = someTextForThisCell;    
    // ...    
     
    // 确保cell的布局约束被设置好了，因为它可能刚刚才被创建好。    
    // 使用下面两行代码，前提是假设你已经在cell的updateConstraints方法中设置好了约束：
    [cell setNeedsUpdateConstraints];    
    [cell updateConstraintsIfNeeded];    
     
    // 将cell的宽度设置为和tableView的宽度一样宽。     
    // 这点很重要。    
    // 如果cell的高度取决于table view的宽度（例如，多行的UILabel通过单词换行等方式），
    // 那么这使得对于不同宽度的table view，我们都可以基于其宽度而得到cell的正确高度。  
    // 但是，我们不需要在-[tableView:cellForRowAtIndexPath]方法中做相同的处理，    
    // 因为，cell被用到table view中时，这是自动完成的。    
    // 也要注意，一些情况下，cell的最终宽度可能不等于table view的宽度。    
    // 例如当table view的右边显示了section index的时候，必须要减去这个宽度。    
    cell.bounds = CGRectMake(0.0f, 0.0f, CGRectGetWidth(tableView.bounds), CGRectGetHeight(cell.bounds));    
     
    // 触发cell的布局过程，会基于布局约束计算所有视图的frame。    
    // （注意，你必须要在cell的-[layoutSubviews]方法中给多行的UILabel设置好preferredMaxLayoutWidth值；    
    // 或者在下面2行代码前手动设置！）    
    [cell setNeedsLayout];    
    [cell layoutIfNeeded];    
     
    // 得到cell的contentView需要的真实高度    
    CGFloat height = [cell.contentView systemLayoutSizeFittingSize:UILayoutFittingCompressedSize].height;    
     
    // 要为cell的分割线加上额外的1pt高度。因为分隔线是被加在cell底边和contentView底边之间的。    
    height += 1.0f;    
     
    return height;
}
 
// 注意：除非行高极端变化并且你已经明显的觉察到了滚动时滚动条的“跳跃”现象，你才需要实现此方法；否则，直接用tableView的estimatedRowHeight属性即可。
- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath{    
    // 以必需的最小计算量，返回一个实际高度数量级之内的估算行高。    
    // 例如：    
    //     
    if ([self isTallCellAtIndexPath:indexPath]) {        
        return 350.0f;    
    } else {        
        return 40.0f;    
    }
}

{% endhighlight %}

###项目demo

  * [IOS8的示例代码](https://github.com/SeongBrave/TableViewCellWithAutoLayoutiOS8) - iOS8以上才支持
  * [iOS7的示例代码](https://github.com/SeongBrave/TableViewCellWithAutoLayout) - iOS7+
  * [其他参考博客](http://blog.sunnyxx.com/2015/05/17/cell-height-calculation/) 

