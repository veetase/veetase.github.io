---
comments: true
date: 2015-08-02 15:54:11
layout: post
title: 重新认识javascript原型链
tags:
- javascript
---
----------

刚接触javascript的时候，只是临时用来在网页上写点小程序，看上两眼就开始用，认为javacript连类都没有，那平时见到的String，Array之流是什么鬼？没有类如何敢声称自己是一门面向对象语言呢？最近有幸拜读了大师Nicholas C. Zakes的《Principles of Object-Oriented Programming in JavaScript》才彻底解开了心中的疑惑。继承作为面向对象最重要的特点之一，本文就试图从解释javascript如何用原型链实现继承来剖析javascript。

##问题一

既然是面向对象语言，那么javascript中的对象是什么呢？在javascript中，对象指由若干个键值对组成的集合，每个键就是对象的property，值就是property的值。如果值是function，那这个function被称为这个对象的方法。

##问题二
知道了什么是对象，如何产生一个对象呢？
产生一个对象的方式有两种，第一种是字面量的方式。以下用字面量的方式创建一个鸭子对象。

{% highlight javascript linenos %}
	var  duck = {
		name: "小黄鸭", 
		color: "Yellow",
		speak: function(){
			console.log("gagaga");
		}
	}
{% endhighlight %}

字面量的方式完全吻合对象的定义，就是一系列的键值对。但是这种方式虽然简单直白，却不能复用，每次都要这么定义一通。若在其它语言里，先定义一个类，每次只要new＋类名，就会得到这个类的实例对象，javascript中虽然没有类，却有类似的东西。名曰：constructor。可以把constructor看成是其它变成语言的初始化函数，它本身就是个function，专门用来生产对象，跟其它普通function不同的是首字母大写（便于区分），其它并没有什么不同。有了constructor就能用它来创建长的一毛一样的对象了。平时我们见到的String，Function就是constructor啦。以下为用constructor的方式创建多个鸭子对象。

{% highlight javascript linenos %}
	function Duck(name, color){
		this.name = name;
		this.color = color;
	}

	duck1 = Duck.new("pony", "yellow");
	duck2 = Duck.new("Tom", "Red");
{% endhighlight %}

以上先定义了一个constructor，然后用这个constructor分别创建了两个鸭子对象。

##问题三
那么如何实现对象的继承呢。
有了前面的constructor，习惯了其它面向对象的用法，如果是constructor去继承另一个constructor想必很好理解，然而事实并不是这样，对象是通过prototype来继承的。可以把constructor想象成鸭子它妈，一个鸭妈妈可以生产出多个小鸭子，这些小鸭子都有自己的小胳膊小腿儿（ownproperty），但是这个鸭妈妈比较懒，只负责生孩子，不负责教育。孩子不能从鸭妈妈这里继承到其它东西，于是鸭爸爸站了出来，这位伟大的鸭爸爸名曰：prototype。

在用constructor创建对象的时候，constructor会把自己的prototype property给每一个对象作为它的prototype property。这相当于每次生孩子的时候，这位懒妈妈就对这个孩子说，那个是你亲爹，以后要学费继承财产什么的找你爹，别找我。（请忽略创建出来的对象跟constructor共用一个prototype这个在现实社会有违伦理的事情）。

跟妈给的东西（own property）不同，每个妈给的胳膊腿儿，每个小鸭子都有自己单独的拥有的。继承自爹的property却是公用的，比如说语言，他们都是用的他爹教给他们的同一种语言，鸭语。他们的房子也是他们爹的房子给他们公用。所以有了prototype，能节省很多不必要资源。

## 三者之间的关系
现在对象也有了, 继承问题也理清了，但总体他们三者又是什么关系呢，孩子将来跟谁姓呀？从上面大家也可以看出来，鸭妈妈这么懒，除了吃喝生孩子什么事都不干，显然这是母系社会，小鸭子将来注定要跟娘姓。也就是说每个对象的instanceof是constructor的名字。最悲催的是，鸭妈妈只负责生，连孩子姓什么，也是鸭爸爸告诉他们的，男人就是命苦啊。
如下图：
![Relation]({{ site.url }}/images/relation.jpg)

												鸣谢：Nicholas C. Zakes，闻端