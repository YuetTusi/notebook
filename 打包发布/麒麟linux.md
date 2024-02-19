## 银河麒麟


### 发布前

接下项目将所有包安装正确

将node_modules目录所有文件赋最大权限

```txt
sudo chmod 777 -R ./node_modules
```

安装fpm

```txt
sudo apt update
sudo apt install ruby -y
sudo gem install fpm
export USE_SYSTEM_FPM=true
```


使用npm全局安装node-pre-gyp：

```txt
sudo npm install node-pre-gyp -g
```

使用华为镜像代替国外源：

```txt
export ELECTRON_BUILDER_BINARIES_MIRROR=https://mirrors.huaweicloud.com/electron-builder-binaries
```

如果打包过程中报node内存不足，可全局安装`increase-memory-limit`包，并在项目目录中执行：

```bash
sudo increase-memory-limit
```

