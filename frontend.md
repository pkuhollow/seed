# 前端篇

前端的主要任务是部署网页和CDN。[webhole](https://github.com/pkuhollow/webhole)主要的资源文件都使用`jsDelivr`免费高速的`GitHub`资源文件CDN.而且基于React框架的webhole将几乎所有功能放在了js文件里，因此我们甚至只需要在web服务器内放置几个html文件即可。

## 修改名称

目前webhole的修改并不十分方便

1. 搜索修改“未名” “北大” “北京大学” “pku” “pkuhollow”等字眼
2. 修改资源图片，如favicon

## 本地调试

在本地调试的时候，请尽量断开网络连接。最好在后端服务运行起来之后进行调试。

```bash
git clone https://github.com/pkuhollow/webhole
cd webhole
git submodule update --init --recursive

# Edit environment configs
vim .env

# start dev server
npm install
npm start
```

## 生产环境

我们写好了GitHub Actions文件，只需要将修改推到master分支上，GitHub就会自动构建新的版本，并推送到`gh-pages-master`分支。版本号位于`package.json`里面。修改后会变成新的版本号。

**记得修改GitHub actions文件里面的仓库地址**

在主机上进行`git pull`操作后就可以获得构建之后的新前端。



**祝您好运，愿树洞之神与您常在。**
