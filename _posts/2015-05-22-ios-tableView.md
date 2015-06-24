---
layout: post
title: IOS TableView
category: 技术
tag: [IOS, TableView]
description:  
---

### didSelectRowAtIndexPath not call

- delegate
- tableview的section不能是none.

### 设置点击时的页面响应UI

	- (void)tableView:(UITableView *)tableView didHighlightRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(6_0)
	- (void)tableView:(UITableView *)tableView didUnhighlightRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(6_0)

### headView 背景色

    UIView *bgView = [[UIView alloc] initWithFrame:self.frame];
    bgView.backgroundColor = kColorBackground;
    self.backgroundView = bgView;

###内存释放

	重用cell引用的图片不会立刻释放，想让他释放，手动执行cell.image = nil;
	
	[p.alAssetURL absoluteString] 这个方法很慢；