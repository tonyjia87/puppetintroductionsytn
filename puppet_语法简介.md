
#Puppet 语法简介
@@TOC@@


> ##1.   写在前面
> &emsp;这个简介的目的是为了更好地帮助大家了解对puppet入门，因为我在看 PRO puppet 第一 ，第二版中都话了很多时间在语法上专研，主要是英语太差，字面的翻译感觉忽懂忽不痛。打算写一个简明的语法介绍，让朋友们花上3到5个番茄时间能对它的语法有感觉。
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

		resourec { “type”：
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












