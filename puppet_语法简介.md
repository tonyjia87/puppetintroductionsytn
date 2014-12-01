
#Puppet 语法简介
@@TOC@@


> ##1.   写在前面
> &emsp;这个简介的目的是为了更好地帮助大家了解对puppet入门，因为我在看 PRO puppet 第一 ，第二版和puppet官方指南都话了很多时间在语法上专研，主要是英语太差，字面的翻译感觉忽懂忽不痛。打算写一个简明的语法介绍，让朋友们花上3到5个番茄时间能对它的语法有感觉。
> 

##2.   资源的认识

&emsp;第一个puppet命令 ` puppet resource service` ,运行后你可以发现你现在所有服务状况。

		ervice { 'SuSEfirewall2_init':
  		  ensure => 'stopped',
          enable => 'false',
        }
		service { 'SuSEfirewall2_setup':
  		  ensure => 'stopped',
  		  enable => 'false',
		}
		service { 'aaeventd':
  		  ensure => 'running',
  		  enable => 'false',
		}
		... ...
&emsp;这个命令中 `resource` 是 `puppet` 的子命令，`service`是puppet中的一个资源。点击 [资源](https://docs.puppetlabs.com/references/latest/type.html "Type Reference" )你可以先预览puppet内置资源，我在这里就不浪费章节介绍了。

&emsp;在这里我简单介绍资源，假如你要建立一个http服务器，你可能会建立一个 `www` 账号作为系统用户给apache用，会用`iptables`放开通信端口，会`yum` 安装相关软件，会 `setsebool `运行软件运行，会`vi`配置你的conf文件，会`service`运行你得服务，多种操作帮你建立你需要的服务器，puppet希望将**操作** 变成一个**资源**，你将会有各种资源控制你手上的服务器，通俗的说资源就是干某件简单事情的类。

&emsp;一个资源在使用上十分简单，注意四个部分：

		resourec { “title”：
			attributs	=> value,
			}
			
使用这样表达形式就像给一个命令写描述，在puppet行话中叫 `decribe` ** 描述**，具体案例可参考如下 ：

		user { "suse"：
			ensure 		=> present,
			uid    		=> "1000",
			gid    		=> 'video',
			shell  		=> '/bin/zsh',
			home   		=> '/home/suse',
			managehome	=> true,
			}

而这样写好的资源，要应用起来，在puppet行话中叫 `declaration` **宣告**。来一个宣告的例子让你了解下 ：

		class ssh {
  			class { '::ssh::install':} ->
			class { '::ssh::config':} ->
			class { '::ssh::service':} ->
			Class['ssh']
			}
最后一个`Class` ，就是宣告。`class ssh {` 后面3个class都是描述，描述的是一个执行过程。

&emsp; 我个人的理解：拿一个你需要的**资源** ，**描述**你的需求，完成**宣告**工作。 ***<font color="red"> “it is the heart of the Puppet language”</font>***。 注意 不是所有资源的属性使用一样的值，甚至type的定义也各不相同。虽然puppet 语言不是很难，但是写模块前一定要看一遍[官方资源参考手册](https://docs.puppetlabs.com/references/latest/type.html "Type Reference" ),当你在主机上操作时对于需要的资源不属性，可以使用`puppet describe ` 子命令参考。

&emsp;前面我使用` puppet resource service`看到了服务器的服务状况，说明puppet其实可以用命令的形式查询。如果要需改也可以使用命令的形式，格式如下：

	puppet resource <TYPE> [ATTRIBUTE=VALUE ...]

假如我写一个计划任务，我可以通过puppet的命令完成，如：

	puppet resource cron puppet-agent ensure=present user=root minute=30 command='/opt/puppet/bin/puppet agent --onetime --no-daemonize --splay --splaylimit 60'


##3.   清单
&emsp;puppet模块中 用`.pp`做后缀的文件被叫做清单，行话`manifests`。puppet只认此扩展名文件，所以在模块中，如果你没有指定加载某个.pp文件，puppet会自动帮你加载。
&emsp;清单内就是代码，支持逻辑顺序，条件状态，资源集合，函数等功能。你可以用 `puppet apply` 跟上你得.pp文件，就可以在本地应用你得代码。如果你要对你的.pp做语法检查，请使用`puppet parser validate`跟上.pp文件。如果对模块进行语法检查，可以在模块根目录下 执行`puppet parser validate *`.下面来一次体验。


	#/tmp/fileEX.pp
	file {"/tmp/test1":
        ensure => file,
        content => "Hi",
	},

	file {"/tmp/test2":
        ensure => directory,
        mode => 0644,
	},

	file {"/tmp/test3":
        ensure => link,
        target => "/tmp/testlink",
	},
	user {"tony":
        ensure => absent,	
	},
	notify {"notify you.":
	},

使用`puppet parser validate`检查发现有报错：

	Error: Could not parse for environment production: Syntax error at ',' at /tmp/fileEX.pp:4

将pp文件中 `},` 改为`}` ,在检查一遍没有问题了，我们使用 `puppet apply /tmp/fileEX.pp `本地应用一次。得到如下的结果

	Notice: /Stage[main]/Main/File[/tmp/test3]/ensure: created
	Notice: /Stage[main]/Main/File[/tmp/test1]/ensure: defined content as '{md5}c1a5298f939e87e8f962a5edfc206918'
	Notice: /Stage[main]/Main/File[/tmp/test2]/ensure: created
	Notice: notify you.
	Notice: /Stage[main]/Main/Notify[notify you.]/message: defined 'message' as 'notify you.'

检查一下我们的结果：

	 # ls -l /tmp/  | grep test
	-rw-r--r-- 1 root root    2 Nov  3 09:27 test1
	drwxr-xr-x 2 root root 4096 Nov  3 09:27 test2
	lrwxrwxrwx 1 root root   13 Nov  3 09:27 test3 -> /tmp/testlink

test2目录权限是0644，但没有用递归，如果需要可以使用 `recurse => true`.

&emsp;还没有结束奥，请在运行一次`puppet apply /tmp/fileEX.pp`,屏幕显示：

	Notice: notify you.
	Notice: /Stage[main]/Main/Notify[notify you.]/message: defined 'message' as 'notify you.'


&emsp;通过上面的例子你是否有feel？ 其实`ensure => file`就是做了创建文件，`directory` 和 `link`也是相应地创建了目录和链接，当我们第二次运行 `puppet apply /tmp/fileEX.pp` 后只有 “notify you” ，***是因为puppet不会将已经创建的文件再重新创建一次***，这样的机制不是仅仅针对file资源，除了`notify`这类显示告知的资源外，其他资源都会在puppet内部做一次判断，所以你写清单的时候不需要像 BASH那样先做一堆`test`的活。

&emsp;不要浪费这个例子，之前描述中有一股User['tony']的事件是没有生效，它的作用是删除tony这个用户，因为没有这个用户所以puppet就不会让他生效。那么我们只需要创建tony用户，再运行`fileEX.pp` 就可以生效，让我们看看：

	# useradd tony
	# id tony
	uid=1000(tony) gid=100(users) groups=16(dialout),33(video),100(users)
	# puppet apply /tmp/fileEX.pp

	Notice: /Stage[main]/Main/User[tony]/ensure: removed
	Notice: notify you.
	Notice: /Stage[main]/Main/Notify[notify you.]/message: defined 'message' as 'notify you.'

	# id tony
	id: tony: No such user

&emsp;puppet语法中存在一个`namevar`的概念，在上一个例子中你就应该看到了。
	
	file { "/tmp/test1" :

我前面说资源的时候给出了一个格式：
	
			resourec { “title”：
			attributs	=> value,
			}

其实根据格式应该是写成：

	file { "file of resource title" :
	path => "/tmp/test1" 
	ensure => file,
	content => "Hi",
	}

	user { "user of resource title" :
	name => "tony"
	ensure => absent,
	}
每个资源都会有namevar的机制，你在参考资源使用上会看到这个特别的变量。

	notify { 'resource title':
  	  name     => # (namevar) An arbitrary tag for your own reference; the...
  	  message  => # The message to be sent to the...
  	  withpath => # Whether to show the full object path. Defaults...
		# ...plus any applicable metaparameters
	}



##顺序
&emsp;fileEX.pp创建了我要的文件，那么现在我需要删除他们，代码如下：
	
	file { "/tmp/test1" :
        ensure => absent,
	}

	file { "/tmp/test2" :
        ensure => absent,
        force => true,
        recurse => true,
	}

	file { "/tmp/test3" :
        ensure => absent,
	}
	file { "/tmp/testlink" :
        ensure => absent,
	}

	notify { "notify you." :}

	notify { "Has deleted file test1." :
        require => File["/tmp/test1"],
	}

	notify { "Has deleted directory test2." :
        require => File["/tmp/test2"],
	}

	Notify["Has deleted file test1."] -> Notify["Has deleted directory test2."] -> Notify["notify you."] -> File["/tmp/test3"] -> File["/tmp/testlink"]   
用`puppet apply`运行我的代码后：

	Notice: /Stage[main]/Main/File[/tmp/test1]/ensure: removed
	Notice: /Stage[main]/Main/File[/tmp/test2]/ensure: removed
	Notice: Has deleted file test1.
	Notice: /Stage[main]/Main/Notify[Has deleted file test1.]/message: defined 'message' as 'Has deleted file test1.'
	Notice: Has deleted directory test2.
	Notice: /Stage[main]/Main/Notify[Has deleted directory test2.]/message: defined 'message' as 'Has deleted directory test2.'
	Notice: notify you.
	Notice: /Stage[main]/Main/Notify[notify you.]/message: defined 'message' as 'notify you.'
	Notice: /Stage[main]/Main/File[/tmp/test3]/ensure: removed

如果我将最后一条表达式删除，显示的结果也不一样。它将会显示：

	Notice: /Stage[main]/Main/File[/tmp/test3]/ensure: removed
	Notice: /Stage[main]/Main/File[/tmp/test1]/ensure: removed
	Notice: /Stage[main]/Main/File[/tmp/test2]/ensure: removed
	Notice: Has deleted directory test1.
	Notice: /Stage[main]/Main/Notify[Has deleted directory test1.]/message: defined 'message' as 'Has deleted directory test1.'
	Notice: notify you.
	Notice: /Stage[main]/Main/Notify[notify you.]/message: defined 'message' as 'notify you.'
	Notice: Has deleted file test1.
	Notice: /Stage[main]/Main/Notify[Has deleted file test1.]/message: defined 'message' as 'Has deleted file test1.'

&emsp;最后一条表达式的功能就是 先删除test1文件，在删除test2目录，最后删除链接。而没有表达式我的代码执行顺序是不能控制的，"`->`"符号的作用就是让你的执行顺序变得更加清晰。代码中的 `Notify` 和 `File`这样的将资源首字母大写，其作用是为了应用，除了应用以外还具有默认使用的意思。"`require`"属于
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              [metaparameters](https://docs.puppetlabs.com/references/latest/metaparameter.html "") 
，往往是做好一个事情后触发性做其他事情，相当于**依赖关系**的功能，与`require`相对的是`before`。另外还有一种触发性的关键字是 `notify`和`subscribe`，他们主要用在 `service`,`exec`和`mount`的联系上，具有刷新（refreshed）的含义。比如我将配置下发好后我要重启服务 ：

	file { "/etc/apache2/apache.conf":
		ensure => file,
		mode => 0640,
		source => 'puppet:///modules/apache2/httpd.conf',
	}

	service { 'apache2':
		hasstatus => false,
		hasrestart => false,
		status => '/etc/init.d/apache2 status',
		restart => '/etc/init.d/apache2 reload',
    	start => '/etc/init.d/apache2 start',
    	stop  => '/etc/init.d/apache2 stop',
    	ensure => running,
    	enable => true,
  		}

##变量 

&emsp;这篇是介绍基础知识，不设计扩展功能，如 从数据库去取变量的值，这类功能会在高级的语法介绍中出现。

&emsp;首先介绍在同一个.pp文件内变量的使用：

	$var1 = "it is a variable"
 	file { "/tmp/varfile" :
		ensure => file,
		content => $var1 ,
	} 

	 notify {$var1 :}
	
执行后 ：
	
	Notice: /Stage[main]/Main/File[/tmp/varfile]/ensure: defined content as '{md5}5c2f8eb9caf6acc0d00e7882f7215749'
	Notice: it is a variable
	Notice: /Stage[main]/Main/Notify[it is a variable]/message: defined 'message' as 'it is a variable'

	#cat varfile 
	it is a variable

&emsp;模块内部变量的使用，先看下模块**结构**：

	E:\PUPPET-MODULES\apache
	│  LICENSE
	│  README.md
	│
	└─manifests
        confdir.pp
        config.pp
        data.pp
        datadir.pp
        defaults.pp
        init.pp
        install.pp
        module.pp
        params.pp
        service.pp

&emsp;简单介绍下模块，`E:\PUPPET-MODULES\apache` 中的apache是模块名，而我们在 [forge.puppetlabs.com/](https://forge.puppetlabs.com "forge")会看到 “nanliu/staging” 或者 “pro-mysql”等模块名称，都是不能直接拿来使用的。因为puppet将上传给大众参考的模块依照` “提供者”-“模块名” `的格式要求。另外模中的pp文件块必须在 `manifests`目录内，而且必须有` init.pp	`文件。模块内多个pp文件需要传变量名需要哪些必要条件呢？

&emsp;首先举一个例子

用命令 `puppet module generate chinese-apache`创建模块，你也可以自己`touch`文件组合一个模块。

	# puppet module generate chinese-apache
	We need to create a metadata.json file for this module.  Please answer the
	following questions; if the question is not applicable to this module, feel free
	to leave it blank.

	Puppet uses Semantic Versioning (semver.org) to version modules.
	What version is this module?  [0.1.0]
	--> 

	Who wrote this module?  [chinese]
	--> 

	What license does this module code fall under?  [Apache 2.0]
	--> 

	How would you describe this module in a single sentence?
	--> 

	Where is this module's source code repository?
	--> 

	Where can others go to learn more about this module?
	--> 

	Where can others go to file issues about this module?
	--> 

	----------------------------------------
	{
  	"name": "chinese-apache",
  	"version": "0.1.0",
  	"author": "chinese",
  	"summary": null,
  	"license": "Apache 2.0",
  	"source": "",
  	"issues_url": null,
  	"project_page": null,
  	"dependencies": [
    	{
      	"version_range": ">= 1.0.0",
      	"name": "puppetlabs-stdlib"
    	}
  	]
	}
	----------------------------------------

	About to generate this metadata; continue? [n/Y]
	--> Y

	Notice: Generating module at /tmp/chinese-apache...
	Notice: Populating ERB templates...
	Finished; module generated in chinese-apache.
	chinese-apache/tests
	chinese-apache/tests/init.pp
	chinese-apache/spec
	chinese-apache/spec/classes
	chinese-apache/spec/classes/init_spec.rb
	chinese-apache/spec/spec_helper.rb
	chinese-apache/manifests
	chinese-apache/manifests/init.pp
	chinese-apache/README.md
	chinese-apache/Rakefile
	chinese-apache/metadata.json

注意下一步：

	 # mv chinese-apache/ /etc/puppet/modules/apache

这样模块“apache”就创建好了。

下面在 `apache/manifests` 内 创建defaults.pp

	#vi defults.pp
	class apache::defaults {
        $version = 'Version_0.1'
	}

init.pp的内容

	class apache {
		include apache::defaults
		notify { $::apache::defaults::version :}
	}

服务器端运行 `puppet master --verbose --no-daemonize --noop` , 客户端运行 `puppet agent -t --server=ppmaster`，你可以看到 ：
	Notice: /Stage[main]/Apache/Notify[Version_0.1]/message: defined 'message' as 'Version_0.1'

这下可有体验到变量的使用吗？ ***$::模块名::pp文件::变量名*** 。

体验下facter值的应用。
	
	# cat defaults.pp 
	class apache::defaults {
		$version = 'Version_0.1'
		$motd = "This VM`s IP address is ${ipaddress},running ${operatingsystem} !!! "
	}
	

	# cat init.pp 
	class apache {
		include apache::defaults
		$notice = $::apache::defaults::motd	
		notify { $::apache::defaults::version :}
		notify { $notice :}
	}

运行后可以看到 

	defined 'message' as 'This VM`s IP address is 192.168.2.11,running SLES !!! '

对于facter的值，pp文件在引用主机自己fact值的时候，并且在引号内 使用 `${fact}` 格式。不再引号内使用 `$::fact`格式。 


##判断条件

这里介绍不会很全面，因为puppet的条件语句 `if` 和 `unless`是相反的意义，一样的结构。所以只介绍`if`。判断语句`case`和linux 的case一样用法。`selector`是一个小型的判断语句，这会介绍。

if语句结构：

	if condition {
  	  block of code
	}
	elsif condition {
  	  block of code
	}
	else {
  	  block of code
	}

if语句中状态其实是靠boolean判断，简单说就是true和false的结果。不同的对象有不同的定义，条件语句中不同于linux会做一个计算后，将0和1做结果去判断。因为puppet是声明语言，只做描述，如你要做什么！你要怎么去做！，不会思考。如果你要思考就要用facter去判断，或者lib库中加入自己的判断函数。

* **字符串（strings）** ：空的字符串就是false，其他就是true。原生态的puppet对字符串“false”也是任务true，如果你有需要可以使用stdlib中的`str2bool`做转换。
* **数值（numbers）**：原生态的puppet对所有数值都是true，包括零。如果你有需要对零作为false可以使用stdlib的`num2bool`。
* **undef**： 数据类型是undef就是flase，undef就没有定义值，类似python的pass。
* **数组和哈希（也叫字典）** ：数组和哈希都是true，即使内容是空的。


以上提到的lib库，可以参考[stdlib](https://forge.puppetlabs.com/puppetlabs/stdlib "puppetlabs/stdlib") 。由于和puppet语法没有关系，会在后期开发模块上介绍。例外条件中如果需要判断，与或非等功能可以参考[expressions](https://docs.puppetlabs.com/puppet/latest/reference/lang_expressions.html "lang_expressions")。值得一提的是，[“转换”](https://docs.puppetlabs.com/puppet/latest/reference/lang_datatypes.html#automatic-conversion-to-boolean  "conversion")，对常用的类型转换进行总结。

case 判断支持正则表达式

	case $operatingsystem {
      'Solaris':          { include role::solaris } # apply the solaris class
      'RedHat', 'CentOS': { include role::redhat  } # apply the redhat class
      /^(Debian|Ubuntu)$/:{ include role::debian  } # apply the debian class
      default:            { include role::generic } # apply the generic class
    }


Selectors 和case有一些类似

    $apache = $operatingsystem ? {
      centos                => 'httpd',
      redhat                => 'httpd',
      /(?i)(ubuntu|debian)/ => 'apache2',
      default               => undef,
    }


Selectors 和case 一般我会用在params.pp文件下，在不同系统不同软件版本上做判断。

##模块
前面讲到的`Class['ssh']` 其实就应用ssh类的一种手段，但是应用类还可以使用`include`完成。
