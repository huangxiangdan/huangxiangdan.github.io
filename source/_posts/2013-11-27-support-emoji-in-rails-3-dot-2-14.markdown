---
layout: post
title: "support emoji in rails 3.2.14"
date: 2013-11-27 09:29:23 +0800
comments: true
categories: rails
---
本文主要参考一片日文的文章[Rails 3.2でiOS5の絵文字を扱う](http://qiita.com/ikm/items/7ac0c32c5264eac2b8bb)，实践并修改完成。让你的应用可以支持emoji，需要达到以下几点要求

1. 数据库支持emoji(utf8mb4)
2. rails的sql adapter可以支持以utf8mb4的方式访问数据库
3. 传输过程中，utf8mb4的信息不会丢失

#让MySQL数据库支持emoji(utf8mb4)的存储
MySQL 5.5.3以上已经支持了utf8mb4的字符集，所以如果你的MySQL是5.5.3以上，只需要在my.conf文件中按照如下的配置就可以支持utf8mb4的字符集了。
	
	[client]
	default-character-set = utf8mb4
	
	[mysqld]
	collation-server = utf8mb4_unicode_ci
	character-set-server = utf8mb4
在修改完my.conf配置之后，重启mysql，检查字符集是否已经更改，除了``character_set_system``和``character_set_filesystem``之外，其他的字符集都需要变成utf8mb4类型。
	
	mysql> show variables like 'char%';
	+--------------------------+------------------------------------------------------+
	| Variable_name            | Value                                                |
	+--------------------------+------------------------------------------------------+
	| character_set_client     | utf8mb4                                              |
	| character_set_connection | utf8mb4                                              |
	| character_set_database   | utf8mb4                                              |
	| character_set_filesystem | binary                                               |
	| character_set_results    | utf8mb4                                              |
	| character_set_server     | utf8mb4                                              |
	| character_set_system     | utf8                                                 |
	| character_sets_dir       | /usr/local/Cellar/mysql/5.5.10/share/mysql/charsets/ |
	+--------------------------+------------------------------------------------------+
	
为了支持之前用utf8创建的表同样支持emoji，你还需要将之前的表修改未utf8mb4字符集

	alter table posts convert to character set utf8mb4 collate utf8mb4_unicode_ci;
	
#rails支持emoji(utf8mb4)
rails对utf8bm4的支持主要通过sql adapter实现，mysql2的0.3.13版本已经支持了utf8mb4 encoding。主要在配置(database.yml)的时候加上以下配置即可

	charset: utf8mb4
	encoding: utf8mb4
	collation: utf8mb4_unicode_ci
	
在做完上述工作之后你的rails其实已经支持了utf8mb4的存储与读取，但为了与之前的代码兼容，你还需要做下面的事情
##处理MySQL index 767个字节的限制
因为mysql对index是有767个字节的限制的，在我们默认使用utf8编码的时候，如果我们定义如下的结构和index，是不会有问题的
	
	add_column :users, :name, :string
	add_index :users, :name
	
因为utf8最多占3个字节，而rails中创建string类型的字段时，默认用的是varchar(255)这样的数据库结构，所以255\*3=765，没有超出767的限制。但我们把字符集改成了utf8mb4，所以一个字最多可以占用4个字节，那255\*4就会超出767的字符限制了，所以我们没有办法使用上述的方式来创建字段和index了。

所以我们应该限定name的长度

	add_column :users, :name, :string, :limit => 100
	add_index :users, :name

但对于之前的系统，如果我们全部修改string类型的index,会是一个很大的工作量（主要是在正式服务器上的耗费时间会很长），所以我们可以有个折中的方案，只修改index的长度限制。

	add_index :entries, [:user_id, :url],
      unique: true, length: { url: (191-4) }
      
所以你需要找遍所有的migration文件，通通加上长度的限制，这样才能避免一个新同学可以通过`rake db:migrate`。

除此之外，你还需要修改rails系统的schema_migrations_table表的结构，打开activerecord（`bundle open activerecord`），修改lib/active_record/connection_adapters/abstract/schema_statements.rb的419行，换成下面的语句。

	schema_migrations_table.column :version, :string, :null => false, :limit => 15
	
或者你可以像我一样，打个补丁。将以下内容写入`config/initializers/active_record_schema_migrations_version.rb`文件

	require 'active_support/core_ext/array/wrap'
    require 'active_support/deprecation/reporting'

    module ActiveRecord
      module ConnectionAdapters # :nodoc:
        module SchemaStatements
                # Should not be called normally, but this operation is non-destructive.
          # The migrations module handles this automatically.
          def initialize_schema_migrations_table
            sm_table = ActiveRecord::Migrator.schema_migrations_table_name

            unless table_exists?(sm_table)
              create_table(sm_table, :id => false) do |schema_migrations_table|
                schema_migrations_table.column :version, :string, :null => false, :limit => 15
              end
              add_index sm_table, :version, :unique => true,
                :name => "#{Base.table_name_prefix}unique_schema_migrations#{Base.table_name_suffix}"

              # Backwards-compatibility: if we find schema_info, assume we've
              # migrated up to that point:
              si_table = Base.table_name_prefix + 'schema_info' + Base.table_name_suffix

              if table_exists?(si_table)
                ActiveSupport::Deprecation.warn "Usage of the schema table `#{si_table}` is deprecated. Please switch to using `schema_migrations` table"

                old_version = select_value("SELECT version FROM #{quote_table_name(si_table)}").to_i
                assume_migrated_upto_version(old_version)
                drop_table(si_table)
              end
            end
          end
        end
      end
    end

	
#修改JSON gem保障utf8mb4信息不丢失
据[Emoji and Rails JSON output issue](http://blog.sosedoff.com/2012/04/26/emoji-and-rails-json-output-issue/)文章描述，在未对json库做补丁之前，JSON会丢失掉一些信息。（我测试的结果是，未打补丁之前，确实emoji无法正确显示）
	
	>> JSON({:a => "\360\237\230\204"}.to_json)
	=> {"a"=>"\357\230\204"}
	
所以，针对这个问题的解决方案就是，把以下内容写入`config/initializers/active_support_encoding.rb`文件

	module ActiveSupport::JSON::Encoding
		class << self
    		def escape(string)
      			if string.respond_to?(:force_encoding)
        			string = string.encode(::Encoding::UTF_8, :undef => :replace).force_encoding(::Encoding::BINARY)
      			end
      			json = string.gsub(escape_regex) { |s| ESCAPED_CHARS[s] }
      			json = %("#{json}")
      			json.force_encoding(::Encoding::UTF_8) if json.respond_to?(:force_encoding)
      			json
    		end
    	end
    end
这样做完之后，你就可以支持emoji在json格式中的传输了。

[Rails 3.2でiOS5の絵文字を扱う](http://qiita.com/ikm/items/7ac0c32c5264eac2b8bb)还给了另外一个解决办法，

	module ActiveSupport
		module JSON
		    module Encoding
		      class << self
		        def escape_with_json_gem(string)
		          ::JSON.generate([string])[1..-2]
		        end
		        alias_method_chain :escape, :json_gem
		      end
		    end
		end
	end
但据我测试，如果在不使用resque的时候，但在使用`resque`的时候，就会出现死循环的问题了。导致`resque`无法启动。如果你也使用了`resque`，你可以在escape_with_json_gem里面加入puts信息，来验证我说的话。很有可能是resque对JSON库的generate做了一些改动（我看了resque的代码，没有发现相关的细节，但resque确实动过JSON gem），导致escape与generate循环调用了。因为这个原因，我们使用这种办法来做JSON的补丁。


#添加测试保证对utf8mb4的支持
为了确保我们上述的工作已经很好地支持了emoji，你最好可以在你的测试中加入类似的测试用例，来确保这一点。

	  test "post emoji comment create" do
    	post = FactoryGirl.create(:post)
    	post :create, :format => :json, :comment => {:content => "😔"}
	    assert_response_result(true)
	    #测试json是否正确
	    assert_equal "😔", @response_json["content"]
	    #测试数据库是否正确保存与读取
	    assert_equal "😔", Comments.first.content
	  end
	  
###参考资料

* [Rails 3.2でiOS5の絵文字を扱う](http://qiita.com/ikm/items/7ac0c32c5264eac2b8bb)
* [Emoji and Rails JSON output issue](http://blog.sosedoff.com/2012/04/26/emoji-and-rails-json-output-issue/)