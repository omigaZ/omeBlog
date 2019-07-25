---
title: 四种语言的MD5实现
date: 2019-07-25 10:52:42
tags: md5
---
事情起源是这样的：最近在和公司同部门的同事对接一个接口，做签名验证的时候总是验证不通过。虽然已经严格按照文档来做了，而且也和这个同事沟通过很多次。
但是签名依然不通过...嘛~这个也是常有的事。毕竟跟百度小哥对接的时候也是这样，毕竟我们是“宇宙最好的语言--PHP”，毕竟.....  毕你个鬼啊啊啊啊啊啊！！！！

我还就不相信了。


开始排查，参数排序check、排除空值参数check、post参数整体作为一个key为‘body’的参数和其他参数进行签名生成check...  卧槽！我没问题啊。这个时候
就可以开始怀疑别人了，hiahiahia...

和对接的小伙伴一起讨论，把生成签名组合好的参数以及结果发给他（生成很简单就是个md5），发给他。他那边跑了一下。哦哟~  不一样哦。

先发一下我的数据

php代码

$str = "body={"buildingAge":[{"flag":"3"}],"city":"310000","distance":3,"latitude":31.23,"longitude":121.47,"occupancyRate":[{"flag":"3"}],"parkingNum":[{"flag":"3"}],"propertyRent":[{"flag":"3"}]}&clientid=ka_sale&timestamp=1563960381b2d8666a-dcf2-4e92-a044-5a1c68689ac3";

$sign = md5($str);

不能再简洁了，var_dump($sign) 输出 : 32ff9c45347c62876a9cd6f23c86125b  小哥把字符串拿去输出的结果是 2c78236a22f03bd8230f76f495275919

好的问题就在这了，那么就是怎么解决的事情了。我问小哥用的是什么语言，第一次小哥回复是java。包名import org.springframework.util.DigestUtils;函数是DigestUtils.md5DigestAsHex(String) 那么我也去试试（本人不才，会用极其简单的springboot）代码如下：

//测试java的MD5
    @GetMapping(value = "/testmd5")
    public String testMd5 () throws UnsupportedEncodingException {
        String jsonStr = "body={\"buildingAge\":[{\"flag\":\"3\"}],\"city\":\"310100\",\"distance\":3,\"latitude\":31.23,\"longitude\":121.47,\"occupancyRate\":[{\"flag\":\"3\"}],\"parkingNum\":[{\"flag\":\"3\"}],\"propertyRent\":[{\"flag\":\"3\"}]}&clientid=ka_sale&timestamp=1563960381b2d8666a-dcf2-4e92-a044-5a1c68689ac3";
        byte[] b=jsonStr.getBytes("utf-8");
        return DigestUtils.md5DigestAsHex(b);
    }
    
输出结果 32ff9c45347c62876a9cd6f23c86125b  哎哎哎  这不是可以么，瞬间陷入无限的自我怀疑中...


转天早上出现了转机，当我把java的结果发给他时。小哥说我们的网关用的是nginx+lua  我感觉希望来了

lua之前完全没有用过，那么就度娘一下呗。还好windows环境搭建比较简单，下载一个exe就好了。附上链接

+++++++++++++++++++++

Window 系统上安装 Lua
window下你可以使用一个叫"SciTE"的IDE环境来执行lua程序，下载地址为：

本站下载地址：LuaForWindows_v5.1.4-46.exe
Github 下载地址：https://github.com/rjpcomputing/luaforwindows/releases
Google Code下载地址 : https://code.google.com/p/luaforwindows/downloads/list
双击安装后即可在该环境下编写 Lua 程序并运行。

你也可以使用 Lua 官方推荐的方法使用 LuaDist：http://luadist.org/

+++++++++++++++++++++++++来源于https://www.runoob.com/lua/lua-environment.html侵删

我下载的是github的  里面有个exe执行程序  安装结束后给了个编译器--真是方便的语言

闲话不说，看着教程print了一下hello world 确认能用之后立马搜了一下lua的md5 实现起来也很简单

附上代码：

require "md5"

a='body={"buildingAge":[{"flag":"3"}],"city":"310000","distance":3,"latitude":31.23,"longitude":121.47,"occupancyRate":[{"flag":"3"}],"parkingNum":[{"flag":"3"}],"propertyRent":[{"flag":"3"}]}&clientid=ka_sale&timestamp=1563960381b2d8666a-dcf2-4e92-a044-5a1c68689ac3'

b=md5.sumhexa(a)

print(b)


ok，结果果然是 2c78236a22f03bd8230f76f495275919 但是为什么，why  不理解啊  大家哪里不一样了....


网上度娘+咨询大佬无果之后我发现了一个问题  这个问题是这样的  java字符串不能使用单引号 只能使用双引号 所以字符串长这样 "body={\"buildingAge\":[{\"flag\":\"3\"}],\"city\":\"310100\",\"distance\":3,\"latitude\":31.23,\"longitude\":121.47,\"occupancyRate\":[{\"flag\":\"3\"}],\"parkingNum\":[{\"flag\":\"3\"}],\"propertyRent\":[{\"flag\":\"3\"}]}&clientid=ka_sale&timestamp=1563960381b2d8666a-dcf2-4e92-a044-5a1c68689ac3"

反斜杠！！  再看看小哥发给我他使用的字符串，使用单引号连接起来的！！  这个会不会是突破口呢？？ 管他呢  试一试

lua代码修改：

require "md5"

c="body={\"buildingAge\":[{\"flag\":\"3\"}],\"city\":\"310100\",\"distance\":3,\"latitude\":31.23,\"longitude\":121.47,\"occupancyRate\":[{\"flag\":\"3\"}],\"parkingNum\":[{\"flag\":\"3\"}],\"propertyRent\":[{\"flag\":\"3\"}]}&clientid=ka_sale&timestamp=1563960381b2d8666a-dcf2-4e92-a044-5a1c68689ac3"

d=md5.sumhexa(c)

print(d)

怀着激动的心情运行，结果:32ff9c45347c62876a9cd6f23c86125b  我靠对了！！！我是天才！！！


至于为毛会这样目前还在继续调查中，个人推测是反斜杠被md5了

到此为止问题解决

PS：由于最近在学习go  所以心血来潮找了下go的实现

go代码：

package controller

import (
	"crypto/md5"
	"encoding/hex"
	"fmt"
	"github.com/gin-gonic/gin"
	"io"
	"net/http"
)

func TestMd5Api(c *gin.Context)  {
	str := "body={\"buildingAge\":[{\"flag\":\"3\"}],\"city\":\"310100\",\"distance\":3,\"latitude\":31.23,\"longitude\":121.47,\"occupancyRate\":[{\"flag\":\"3\"}],\"parkingNum\":[{\"flag\":\"3\"}],\"propertyRent\":[{\"flag\":\"3\"}]}&clientid=ka_sale&timestamp=1563960381b2d8666a-dcf2-4e92-a044-5a1c68689ac3"
	md5Str1 := md5V(str)
	md5Str2 := md5V2(str)
	md5Str3 := md5V3(str)
	c.JSON(http.StatusOK, gin.H{
		"code":0,
		"data":md5Str1+">>>>>>"+md5Str2+">>>>>>"+md5Str3,
		"errInfo":"",
	})
}

func md5V(str string) string  {
	h := md5.New()
	h.Write([]byte(str))
	return hex.EncodeToString(h.Sum(nil))
}

func md5V2(str string) string {
	data := []byte(str)
	has := md5.Sum(data)
	md5str := fmt.Sprintf("%x", has)
	return md5str
}


func md5V3(str string) string {
	w := md5.New()
	io.WriteString(w, str)
	md5str := fmt.Sprintf("%x", w.Sum(nil))
	return md5str
}

结果就是都一样，看来只有lua比较特殊啊。以上！--Omega

 
