---
title: Vue和移动端App的交互
date: 2019-04-25 16:28:35
tags: ["Vue", "Webapp"]
categories: ["前端"]
---

### 一. Vue 调用 移动端

**Vue** 调用 移动端比较方便, 只需要注册全局函数就能成功调用移动端提供的方法函数

以下为个人所做项目中的一些简单调用方法

----------
	
	let utils = { //注册的全局函数
	  versions: function () { // 判断Android 和 IOS
	    var u = navigator.userAgent, app = navigator.appVersion;
	    return {
	      ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/),
	      android: u.indexOf('Android') > -1 || u.indexOf('Linux') > -1,
	      iPhone: u.indexOf('iPhone') > -1 || u.indexOf('Mac') > -1,
	      iPad: u.indexOf('iPad') > -1,
	    };
	  }(),
	  isMobile: function () {
	    return utils.versions.ios || utils.versions.iPhone || utils.versions.iPad || utils.versions.android;
	  },
	  
	  getToken: function () {
	    if (utils.versions.android) {
	      return jsObj.readFile(); // 调用Android端方法
	    } else if (utils.versions.ios || utils.versions.iPhone || utils.versions.iPad) {
	      return window.prompt(); // 调用IOS端方法
	    }
	    return "";
	  },
	  getEmpCode: function () {
	    if (utils.versions.android) {
	      return jsObj.getEmpCode();
	    } else if (utils.versions.ios || utils.versions.iPhone || utils.versions.iPad) {
	      return window.prompt("getUser");
	    }
	    return "";
	  },
	}

	export default {
	  utils,
	}

----------

### 二. 移动端 调用 Vue

移动端想调用 **Vue**中的方法 需要Vue将被调用的方法挂载到 **Windows** 对象下面


----------

	mounted: function () {
      let _this = this
      window['recordComplete'] = (time, fileId) => {
        // console.log("recordComplete方法回调, 参数: tapeTime: "+time+"---fileId: "+fileId)
        // _this.show1 = true
        _this.recordComplete(time, fileId)
      }
    },
	methods: {
		recordComplete: function() {
			...
		}
	}
	
----------

