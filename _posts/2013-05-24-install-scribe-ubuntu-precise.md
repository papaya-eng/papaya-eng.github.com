---
layout: post
title: "Install scribe in Ubuntu 12.04"
description: ""
category: 
tags: [scribe]
author: <a href="https://github.com/elprup">elprup</a>
---
{% include JB/setup %}

1. if apt-get complains about pgp auth problem, just add pgp key.

        wget --no-check-certificate https://ftp-master.debian.org/keys/archive-key-6.0.asc
        && apt-key add archive-key-6.0.asc
        sudo gpg --keyserver hkp://subkeys.pgp.net --recv-keys 16126D3A3E5C1192
        sudo gpg --export --armor 16126D3A3E5C1192 | sudo apt-key add -
     
2.  install supporting libs.

        sudo apt-get install libboost-all-dev libboost-test-dev libboost-program-options-dev libevent-dev automake libtool flex bison pkg-config g++ libssl-dev git-core make
         
3. install thrift, facebook303

        wget http://archive.apache.org/dist/thrift/0.9.0/thrift-0.9.0.tar.gz
        tar xzvf thrift-0.9.0.tar.gz
        cd thrift-0.9.0
        ./configure
        make
        sudo make install
        cd contrib/fb303/
        ./bootstrap.sh
        ./configure
        make
        sudo make install
         
4. install scribe
     
        git clone git://github.com/facebook/scribe.git
        cd scribe/
        ./bootstrap.sh
        ./configure  CPPFLAGS="-DHAVE_INTTYPES_H -DHAVE_NETINET_IN_H"
        make
It should work here, but my computer always says undefined reference boost::system, so refer to some link[https://github.com/facebook/scribe/issues/57], just link by yourself.

        cd ./src
        g++ -Wall -O3 -o scribed store.o store_queue.o conf.o file.o conn_pool.o scribe_server.o network_dynamic_config.o dynamic_bucket_updater.o env_default.o -L/usr/local/lib -L/usr/local/lib -L/usr/local/lib -lfb303 -lthrift -lthriftnb -levent -lpthread libscribe.a libdynamicbucketupdater.a -L/usr/lib -lboost_system-mt -lboost_filesystem-mt
        cd ..
        sudo make install
        
GIST  LINK[https://gist.github.com/elprup/5642303]
