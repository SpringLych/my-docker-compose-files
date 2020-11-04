# FileBrowser

FileBrowser 是一个使用 Go 语言编写的基于 Web 的文件管理器。



# compose文件配置项说明

重点在于 `environment` ，这里配置的环境变量在创建时就会写入 `database.db` 文件，成为容器创建后的默认配置。

根据官方文档，需要添加前缀 "FB_"进行配置。 可以配置以下变量：

* FB_NOAUTH="true"	启动容器后不需要输入用户和密码
* FB_USERNAME="user"   默认用户名
* FB_PASSWORD="加密后的密码"   默认密码

更多选项可在 [官方文档](https://filebrowser.org/cli/filebrowser) 查看



filebrowser.json 使用的是默认配置

database.db 是一个空文件