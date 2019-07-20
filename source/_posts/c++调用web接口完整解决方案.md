---
title: c++调用web接口完整解决方案
date: 2018-12-19 20:21:11
tags:
- c++
---



在很多其他语言里，调用web接口是非常方便的，但是c++有点麻烦，本篇文章就以调用腾讯ai开放平台的接口为例，介绍一下用到的库以及函数。



## [C++ Requests: cpr](https://github.com/whoshuu/cpr)

这个库一个curl的包装，可以很方便的发起请求和处理错误。这是一段非常简单的GET请求的代码：

```c++
#include <cpr/cpr.h>

int main(int argc, char** argv) {
    auto r = cpr::Get(cpr::Url{"https://api.github.com/repos/whoshuu/cpr/contributors"},
                      cpr::Authentication{"user", "pass"},
                      cpr::Parameters{{"anon", "true"}, {"key", "value"}});
    r.status_code;                  // 200
    r.header["content-type"];       // application/json; charset=utf-8
    r.text;                         // JSON text string
}
```

在向Parameters里添加参数时，cpr会自动进行url编码，腾讯ai开放平台要求的编码方式和cpr自带的不一致，所以可以使用我修改的下面这段代码进行编码，然后拼接参数后直接赋值给Parameters.content.

``` c++
std::string url_encode(const std::string& value)
{
    std::ostringstream escaped;
    escaped.fill('0');
    escaped << std::setiosflags(std::ios::uppercase) << std::hex;
    for (auto i = value.cbegin(), n = value.cend(); i != n; ++i)
    {
        char c = (*i);
        if (std::isalnum(static_cast<unsigned char>(c)) || c == '-' || c == '_' || c == '.' || c == '~')
        {
            escaped << c;
            continue;
        }
        else if (c == ' ')
        {
            escaped << '+';
            continue;
        }
        escaped << '%' << std::setw(2) << std::int32_t((unsigned char)c);
    }
    return escaped.str();
}
```



## [解析响应： rapidjson](https://github.com/Tencent/rapidjson)

大部分接口的响应格式都是json，可以使用rapidjson，有中文文档读起来没啥障碍。



## [MD5加密： openssl](https://github.com/openssl/openssl)

接口鉴权会用到，实测可用的代码:

``` c++
unsigned char md5[17] = { 0 };
char smd5[33] = { 0 };
MD5(reinterpret_cast<const unsigned char*>(content.c_str()), content.size(), md5);
for (int i = 0; i < 16; i++)
    std::sprintf(smd5 + i * 2, "%02X", static_cast<int>(md5[i]));
//smd5即为结果
```



## [日志：glog](https://github.com/google/glog)

最后这个跟调用web接口没啥关系，不过在程序里打印日志对调试web接口作用还是很大的。