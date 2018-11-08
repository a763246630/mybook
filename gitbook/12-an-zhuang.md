# 本地安装

* gitbook依赖nodejs,首先安装nodejs.

* nodejs下载地址

* [https://nodejs.org/dist/v10.13.0/node-v10.13.0-x64.msi](https://nodejs.org/dist/v10.13.0/node-v10.13.0-x64.msi)

```
node -v //查看node版本号,node是否安装成功.
```

* gitbook 安装

```
npm install -g gitbook-cli
```

* 查看gitbook是否安装成功

```
gitbook -v
```

本地reloadlive热加载插件如果出现bug，删除node_modules\gitbook-plugin-livereload文件后重新执行

```
gitbook install
```

```
gitbook build
```

```
gitbook serve
```

