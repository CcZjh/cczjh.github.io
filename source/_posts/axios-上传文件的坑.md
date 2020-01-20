---
title: axios 上传文件的坑
date: 2019-04-12 14:38:27
tags: ["Vue", "Axios"]
categories: ["前端"]
---

最新做一个用Vue写的webApp, 里面需要上传头像文件, 发现通过**Axios**发送**formData**参数, **Java**后台报错:
![](error01.png)

于是乎就麻烦度娘, 可是网上找的写法千篇一律, 写法不同但作用相同, 一直没发现问题到底出在哪, 然后就以为是后台的问题, 从springBoot框架自带的MultipartFile上传组件到最原始的servlet写法都试过了, 还是一样的问题, 所以专一找Vue的问题, 最后终于在茫茫文章的看到了曙光.

Vue在进行跨域请求时需要设置**withCredentials：true**,表示为是否需要使用凭证, 默认为false
完整代码如下: 

----------

	  // 上传头像到服务器
      uploadHeadToServer: function (file) {
        let formData = new FormData()
        formData.append("file", file)
        let headers = {headers: {"Content-Type": "multipart/form-data"}}

        const instance=this.$axios.create({
          withCredentials: false
        })

        instance.post('/api/uploadHead', formData, headers).then(response=>{
            console.log(response.data);
          })
       },

----------

OK 这样一写, 后台不报原来的错的, 但是又出现了新的错误:
![](error02.png)

那就顺带记录一下吧, 毕竟小白, 好记性不如记下来

这是springBoot本身的问题

springBoot文档说明表示，每个文件的配置最大为1Mb，单次请求的文件的总数不能大于10Mb。要更改这个默认值需要在配置文件（如application.properties）中加入两个配置

----------

	multipart.maxFileSize = 10Mb
	multipart.maxRequestSize=100Mb

----------
Spring Boot2.0之后的版本配置修改为:

----------

	spring.servlet.multipart.max-file-size = 10MB  
	spring.servlet.multipart.max-request-size=100MB

----------



