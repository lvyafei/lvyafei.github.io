# lvyafei.github.io
my blog sites source


## 0.设置taobao镜像

```
# npm install -g cnpm --registry=https://registry.npm.taobao.org
# npm config set registry https://registry.npm.taobao.org
```

## 1.安装hexo服务

```
# npm install -g hexo --save
# npm install -g hexo-cli --save
```

## 2.初始化hexo项目

```
# hexo init test-blog
```

## 3.构建

```
# hexo g
```

## 4.运行

```
# hexo s
```

## 5.部署

```
# hexo d
```

## 6.本地下载项目依赖npm包

```
# npm install
```

## 7.集成travis-ci

地址：https://travis-ci.com/lvyafei

需要在项目中添加".travis.yml"文件，配置CICD策略

项目中新增插件,需要在.travis.yml文件中的"before_install:"节点下添加内容，如添加hexo-pdf插件

```
before_install:
  - npm install -g hexo-cli
  - npm install -g hexo-pdf
```