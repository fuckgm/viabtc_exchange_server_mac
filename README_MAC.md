# Mac 下安装编译指南

## 依赖
- zookeeper	
    ```	
    brew install zookeeper
    ```
- kafka  
    要先安装java 1.8即jdk8，以下是先安装adoptopen社区版的jdk8
    ```
    brew cask install adoptopenjdk/openjdk/adoptopenjdk8	
    brew install kafka
    ```
- librdkafka
    ```
    brew install librdkafka
    ```
    源码安装要注释掉
    configure.self里面的
    ```
    # mkl_check "libsasl2" disable
    # mkl_check "zstd" disable
    ```
- libev  
    ```
    brew install libev
    ```
- libmpdec  
    需下载编译安装 [libmpdec](http://www.bytereef.org/mpdecimal/)
- jansson  
    ```			
    brew install jansson
    ```
- libmysqlclient-dev 
    ```
    brew install mysql
    ```
- http_parser
    ```
    brew install  http-parser
    ```
- libcurl  
    需下载编译安装 [libcurl](https://curl.haxx.se/libcurl/)

## 编译

### 编译顺序
    先编译 network,utils  
    然后编译 accesshttp,accessws  
    接着编译 marketprice,matchengine  
    最后编译 alertcenter,readhistory  
### 编译错误
#### network
- EBADFD错误  
    在nw_sock.h添加
    ```
    #ifndef EBADFD
        #define EBADFD EBADF
    #endif
    ```
#### utls
- fatal error: 'openssl/bio.h' file not found  
    ```
    brew install openssl
    echo 'export PATH="/usr/local/opt/openssl/bin:$PATH"' >> ~/.bash_profile
    cd /usr/local/include 
    ln -s ../opt/openssl/include/openssl 
    ```

- fatal error: 'hiredis/hiredis.h' file not found  
    先编译安装depends/hiredis

- fatal error:'endian.h' file not found 和 'byteswap.h' file not found  
    注释掉ut_misc.h以下两行
    ``` c
    #include <endian.h>
    #include <byteswap.h>
    ```
    并添加
    ``` c
    #ifdef __APPLE__
    #include <machine/endian.h>
    #include <libkern/OSByteOrder.h>

    #define htobe16(x) OSSwapHostToBigInt16(x)
    #define htole16(x) OSSwapHostToLittleInt16(x)
    #define be16toh(x) OSSwapBigToHostInt16(x)
    #define le16toh(x) OSSwapLittleToHostInt16(x)

    #define htobe32(x) OSSwapHostToBigInt32(x)
    #define htole32(x) OSSwapHostToLittleInt32(x)
    #define be32toh(x) OSSwapBigToHostInt32(x)
    #define le32toh(x) OSSwapLittleToHostInt32(x)

    #define htobe64(x) OSSwapHostToBigInt64(x)
    #define htole64(x) OSSwapHostToLittleInt64(x)
    #define be64toh(x) OSSwapBigToHostInt64(x)
    #define le64toh(x) OSSwapLittleToHostInt64(x)

    #define __BIG_ENDIAN BIG_ENDIAN
    #define __LITTLE_ENDIAN LITTLE_ENDIAN
    #define __BYTE_ORDER BYTE_ORDER
    #else
    #include
    #include
    #endif
    ```

- use of undeclared identifier 'program_invocation_short_name'  
    mac下没有这个命令  
    在ut_misc.h里加入
    ``` c
    #if defined(__APPLE__) || defined(__FreeBSD__)
    #define appname getprogname()
    #elif defined(_GNU_SOURCE)
    const char *appname = program_invocation_name;
    #else
    const char *appname = argv[0];
    #endif
    ```
    将ut_misc.c里的program_invocation_short_name改成appname
- fatal error: 'mysql/mysql.h' file not found  
    将根目录下的makefile.inc添加以下值
    ```
    LDFLAGS= -L/usr/local/opt/mysql@5.7/lib
    CFLAGS= -I/usr/local/opt/mysql@5.7/include
    ```

- undefined reference to `rd_kafka_conf_set_log_cb'  
    makefile里面将LIBS的 -lrdkafka移到行尾

### accesshttp/accessws
- fatal error: 'error.h' file not found  
    ah_config.h里将
    ```
    # include <error.h>
    ```
    改成
    ```
    #include <mach/error.h>
    ```

- ld: unknown option: -Bstatic  
    去掉makefile里的
    ```
    -Wl,-Bstatic
    -Wl,-Bdynamic
    ```

- Undefined symbols for architecture x86_64:"_error", referenced from:  
    加入以下代码到相应c文件
    ```
    #ifdef __APPLE__
    #  define error printf
    #endif
    ```

- "_clearenv", referenced from:  
    utils/ut_title.h加入
    ```
    #if defined(__FreeBSD__) || defined(__OpenBSD__) || defined(__NetBSD__) || defined(__APPLE__)
    #define clearenv() 0
    #endif
    ```

- library not found for -lssl  
    编译安装openssl到某个位置
    在makefile的 -lssl前加入依赖库
    ``
    -L /Users/bitcocohe/Github/depends/lib
    ```

- mysql库编译无法找到  
    在makefile.inc中的LFAGS加入mysql安装目录
    ```
    -L/usr/local/opt/mysql@5.7/lib
    LFLAGS  := -g -rdynamic -L/usr/local/opt/mysql@5.7/lib
    ```

## 启动
- zookeeper  
    服务启动：
    ```
    brew services start zookeeper
    ```
    临时启动：
    ```
    zkServer start
    ```

- kafka  
    服务启动：
    ```
    brew services start kafka
    ```
    临时启动： 
    ```
    zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties & kafka-server-start /usr/local/etc/kafka/server.properties
    ```

