---
layout: "post"
title: “加快已有Rspec项目速度的尝试”
date: "2017-10-26 15:04"
---

### 背景
对于已有项目，尤其是老项目，想要大刀阔斧的优化很难，只能从一些细节入手。主要是将已有可迅速修复的问题迅速修复和在以后的开发过程中引入一些新的策略。刚接手了一个项目，测试用例整个跑下来耗时很长，在这里大致列举一下以后改进的思路。


#### 用Mock和stub
像API请求类，sidekiq相关等用mock的方式能节省很长时间。这里值得提出的一点是rspec-sidekiq这个Gem，如果配置了`clear_all_enqueued_jobs `这一项，会在全局before(:each)里执行清空队列任务操作，而我们项目里大部分的测试是不需要这个操作的。

#### 谨慎使用callback函数
	  config.before(:each) do
	    Mongoid.purge!
	  end

上面这段代码在每一个测试用例中都进行一次purge操作，而实际上我们有很多场景并不需要用到mongoid，上面代码可以通过细化场景来优化：
	config.before(:each, :mongo_case) do
	  Mongoid.purge!
	end
	
	scenario "some case need mongoid", :mongo_case do
	  # spec logic
	end

这种方式是把所有需要额外操作的用例通过自定义的meta-data标示出来，要求之前的所有的代码进行更改，将需要额外操作的进行添加meta-data。如果项目实在太过巨大或者时间关系，同样可以在以后的代码里添加\_不\_需要额外操作的标签，然后从全局callback中将其忽略掉就可以了。如下：
	 before do
	  unless example.metadata[:no_mongo]
	    Mongoid.purge!
	  end
	end
	
	'' scenario "some case no need  for mongoid", :no_mongo do
	''   # spec logic
	'' end

### 使用`[build _stubbed] `
[它][1]相当于Factory Girl中的mock用法，避免了一次数据库插入。场景使用有限，因为它没有真正的执行数据库插入操作。

### 不要总是`require rails_helper `
比如lib中的文件。优化之后的单个测试用例速度提升明显：
![][image-1]






[1]:	(https://robots.thoughtbot.com/use-factory-girls-build-stubbed-for-a-faster-test)

[image-1]:	http://7xj6xc.com1.z0.glb.clouddn.com/WechatIMG3.jpeg