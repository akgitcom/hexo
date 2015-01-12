title: rails 第三天
date: 2015-01-12 09:11:06
tags: [rails,关联查询,多对多]
---
##多对多关联：
在Rails中多对多关联通过在关联表对应的类中声明has_and_belongs_to_many来实现。
在数据库中，多对多关联使用中间表来实现，表中包括关联表的主键，Active Record假定这个中间表的名字是由关联表的名字根据字母的顺序串联起来得到的。
<!--more-->
例如，关联表为categories和products，中间表的名字就是categories_products。
![关联表为categories和products](http://7u2icj.com1.z0.glb.clouddn.com/github-Rails57_1.gif)

> 注意我们的关联表没有id列，有两个原因:
> 首先，不需要一个唯一的标识来识别两个外键之间的连接，我们定义表的语句像下面这样：


```sql
create table categories_products (
category_id int not null,
product_id int not null,
constraint fk_cp_category foreign key (category_id) references categories(id),
constraint fk_cp_product foreign key (product_id) references products(id),
primary key (category_id, product_id)
);
```

> 第二个原因在中间表中不包括一个**id**列，**Active Record**在访问某个行时会自动包含所有的列。如果包含了一个id列，那么这个**id**列就会复写掉在关联表中的**id**列。

##he has_and_belongs_to_many()声明:
> **has_and_belongs_to_many**在很多方面很像**has_many**，**has_and_belongs_to_many**创建了本质上是一个集合的属性，该属性支持和**has_many**相同的方法。
> 也许我们使用Rails来写一个社区站点，在这里用户可以阅读文章。这里有很多的用户和文章，而且任何一个用户都可以阅读多个文章，为了跟踪，我们希望知道谁读了哪些文章，每篇文章有谁阅读过，我们也希望知道用户最后一次在什么时间阅读了哪篇文章，我们会这样设计表：

![](http://7u2icj.com1.z0.glb.clouddn.com/github-Rails57_2.gif)

我们这样设置两个Model类互相关联：
```ruby
class Article < ActiveRecord::Base
	has_and_belongs_to_many :users
# ...
end

class User < ActiveRecord::Base
	has_and_belongs_to_many :articles
# ...
end
```
这样我们就可以列出所有阅读过文章123的用户和名为pragdave的用户阅读的所有文章：
```ruby
# Who has read article 123?
article = Article.find(123)
readers = article.users
# What has Dave read?
dave = User.find_by_name("pragdave")
articles_that_dave_read = dave.articles
```
当我们的程序通知某个人阅读了某篇文章的时候，将user记录和article记录建立关联，我们调用下面的方法：

```ruby
class User < ActiveRecord::Base
	has_and_belongs_to_many :articles
	def read_article(article)
		articles.push_with_attributes(article, :read_at => Time.now)
	end
# ...
end
```

方法push_with_attributes( )和<<方法的作用一样，都是给两个Model之间设置连接，而且还赋值给中间表记录什么人在什么时间阅读了文章。

> 注：如果该方法难以理解，可以想象一下C#中使用反射给某个对象的字段赋值，我们需要提供对象，对象的字段名，字段对应的值来进行操作。作为一种的关联方法，**hasand_belongs_to_many**支持一系列声明来复写**Active Record**的默认设置：**:class_name**, **:foreign_key**和**:conditions**，和其他的has方法一样(**:foreign_key**设置中间表中的外键的名字)。进一步说，**has_and_belongs_to_many**支持复写中间表的名字，外键列的名字，**find**，**insert**，**delete**中使用的SQL，详细请参考Rdoc。

原文出处:[Ruby on rails开发从头来（五十七）- ActiveRecord基础（多对多关联关系）](http://www.cnblogs.com/dahuzizyd/archive/2008/04/24/ruby_rails_57.html)
参考文章:
[Ruby on Rails，一对多关联（One-to-Many）](http://blog.csdn.net/abbuggy/article/details/8274717)
[http://guides.ruby-china.org/association_basics.html](http://guides.ruby-china.org/association_basics.html)
[https://gist.github.com/nightire/5124880](https://gist.github.com/nightire/5124880)