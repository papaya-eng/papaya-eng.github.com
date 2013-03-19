---
layout: post
title: "About mysql foreign key checks"
description: ""
category: 
tags: [mysql]
author: <a href="https://github.com/socrateslee">Chun Li</a>
---
{% include JB/setup %}


Mysql will ensure the foreign key constraint check on InnoDB if the variable _foreign\_key\_checks_ is enabled. The check itself wil casue issues in some cases(like some tables are sharded among different machines), the pose will show you some tricks to tweak this setting.

The first thing, you may use

    set foreign_key_checks=0;
    
to disable the check. But by default, the scope of the variable is SESSION, not GLOBAL, and the settings is not supported in _my.cnf_. Of course you may want to write a

    set global foreign_key_checks=0;

You need to be careful bacause _global_ clause for _foreign\_key\_checks_ is only available on Mysql 5.5 or future version, using it with Mysql 5.1 will cause an error.

There is an trick for adjust _init\_connection_ variable [from this post](http://oksoft.antville.org/stories/2073847/), to make sure the setting is disabled when every session established. You may write

    set global init_connect='set foreign_key_checks = 0';
    
or put

    init_connect='set foreign_key_checks = 0';

in _my.cfg_ file.

Likely you may put

    init_file=PATH_TO_FILE.sql

in _my.cnf_, and the content of _PATH\_TO\_FILE.sql_ is

    set global foreign_key_checks=0;
   
Then every time _mysql_ starts, the _PATH\_TO\_FILE.sql_ will be executed, and _foreign\_key\_checks_ is disabled in _global_ scope.