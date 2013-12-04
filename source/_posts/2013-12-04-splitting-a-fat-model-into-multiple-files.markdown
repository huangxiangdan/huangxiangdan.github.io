---
layout: post
title: "splitting a fat model into multiple files"
date: 2013-12-04 18:07:10 +0800
comments: true
categories: rails
---
##起因
Rails的应用内，总有一些超级复杂的model，比如User。而正因是**胖model,瘦controller**这样的指导思想，会导致一些model臃肿不堪，有些model甚至会超过1000行，而一旦超过1000行，一个类的代码就变得难以维护了。所以一段时间内，我们为了不让我们的model太臃肿，我们只能不往User里面写代码，而把一些代码写在其他的model里面。比如写成如下的方式
	
	#exam.rb
	class Exam < ActiveRecord::Base
		def add_user(user)
		end
	end
但这样的方式`Exam.add_user(user)`明显没有`user.join_exam(exam)`直观。

	#user.rb
	class User < ActiveRecord::Base
		def join_exam(exam)
		end
	end
所以为了兼顾直观与保持model可维护性更高，我查找了一些资料，尝试为我们的model瘦身。

##解决思路

一个广为流传的办法是[Use concerns to keep your models manageable](https://gist.github.com/dhh/1014971)，这是dhh（Creator of Ruby on Rails）大神推荐的方式，已被列入Rails4的规范之中了([Rails4重大设计决策：“胖”Model用ActiveSupport::Concern瘦身](http://ruby-china.org/topics/7709))。

但使用concern的根本目的不是为了解决单个model的肥胖问题，而是为了让一些model通用的代码片段存在于一个合适的位置，比如commentable，这样但凡有用到comment的model，只要`include commentable`就可以实现评论相关的功能，这样的实现方式既简单又直观。

如果为model瘦身，主要的问题是，如何分离你的业务逻辑，因为单纯的将代码发在几个小文件中是没有什么意义的（为什么没有意义？因为会增加你查找的难度和理解的难度），这也是为什么[7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)说
	
	“Any application with an app/concerns directory is concerning.”
的一个重要原因。

所以综合以上思路，最终实践得出了三个可以简化User model的办法

* 为model抽象一些功能性concern，比如Authenticatable, Genderable。
* 为model分离一些业务相关模块。比如user_post, 比如 user_event
* 将特殊的功能模块抽象成class

##实践
以下实践均以User model为例
###为model抽象一些功能性concern
这样做符合concern本身的意图，比如将用户验证的相关操作和相关scope放在Authenticatable的module，这样User就拥有了可拔插的能力，如果想去掉相关的功能，只需要取消include就可以了。

所以，这部分的代码类似于
	
	#app/models/concerns/authenticatable.rb
	# -*- encoding : utf-8 -*-
	module Authenticatable
	  extend ActiveSupport::Concern
	  included do
	  	def self.basic_auth(account, password)
	  	end
	  end
	  
	  def has_password?
	  end
	end
###为model分离一些业务相关模块
这样做是不太符合concern本身的意图的，但借用concern来表达我们的业务逻辑，也没有什么问题。但这样做最大的问题是，这部分的concern不应该和传统的concern混在一起，这样不同model的业务逻辑容易混乱（至少不好查找）。

所以，我们将这部分代码放在了`app/models/concerns/user_concern/`中，代码类似如下
	
	#app/models/concerns/user_concern/post.rb
	# -*- encoding : utf-8 -*-
    module UserConcern
      module Post
        extend ActiveSupport::Concern

        included do
          has_many :posts, :dependent => :destroy
          has_many :other_posts, :dependent => :destroy
        end
      end
    end
###将特殊的功能模块抽象成class
这种办法最受[7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)推崇，所以，可以借鉴一些这边文章提到的一些方法来抽象我们的代码。这部分代码类似于
	
	#app/models/lib/edit_limit.rb
	# -*- encoding : utf-8 -*-
    class EditLimit
      NO_MODIFY_LIMIT_USERS_KEY = "no_modify_limit_users"

      MODIFY_LIMIT = 5
      MODIFY_LIMIT_KEY = "editable_times_%d"

      def initialize(user)
        @user = user
      end

      def self.no_modify_limit_users
        Rails.cache.fetch NO_MODIFY_LIMIT_USERS_KEY do
          Set.new
        end
      end

      def self.add_no_modify_limit_user(user)
        no_limit_user_ids = no_modify_limit_users

        no_limit_user_ids << user.id

        Rails.cache.write NO_MODIFY_LIMIT_USERS_KEY, no_limit_user_ids
        no_limit_user_ids
      end
    end
### 
最终，我们在user.rb的引用如下所示
	
	#app/models/user.rb
    class User < ActiveRecord::Base
      include Authenticatable
      include UserConcern::Post
    end
##总结
因为Rails本身的设计，对于一些不太复杂的逻辑与应用，实在无需煞费苦心去寻求瘦身与简化。因为符合Rails本身的设计思想在很大程度上就已经很好的分离了model，view，controller（MVC）三者的职责。而一些复杂的应用，在不考虑更好的可读性的前提下（比如将许多逻辑放置在关联表中），也不会形成太过肥胖的model。所以，只有一些人像我这样对可读性有些洁癖，又想让model保持简单的人，才需要类似的解决办法。

##参考资料
* [Use concerns to keep your models manageable](https://gist.github.com/dhh/1014971)
* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [Put chubby models on a diet with concerns](https://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns)