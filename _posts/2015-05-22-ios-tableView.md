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