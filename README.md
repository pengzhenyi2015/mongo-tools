写在前面
===================================
代码仓库基于 MongoDB 官方 mongo-tools 的 100.7.0 版本开发。

官方代码地址：https://github.com/mongodb/mongo-tools.git

改动
----------
在原生 mongodump/mongorestore 的工具上支持**只备份和回档 oplog，而对用户库表不进行备份和回档**。用户库表会通过另外的物理备份流程来完成。

上述功能通过新增的参数来指定。**如果不指定新增参数，则 100% 兼容原生行为。**

具体新增的参数如下。

mongodump:
 - **onlyOplog** - 只备份 oplog，不备份用户的库表。
 - **oplogStart** - 备份 oplog 的起始时间，秒级时间戳。如果备份节点最旧的 opTime 已经大于这个值，备份失败。
 - **oplogEnd** - 备份 oplog 的结束时间，秒级时间戳。如果备份节点最旧的 opTime 已经大于这个值，备份失败；如果最新的 opTime 小于这个值，会等待 1 分钟，让 oplog 追上来，如果还不满足，则备份失败。

mongorestore:
- **onlyOplogReplay** - 只回档 oplog

如何使用
--------
参考示例：

mongodump:

无 SSL：
```
./bin/mongodump --uri=mongodb://xxx:xxx@127.0.0.1:7000/admin?readConcernLevel=majority --onlyOplog --oplogStart=1684390978 --oplogEnd=1684391088 --out=dump1
```

有 SSL：
```
./bin/mongodump --uri=mongodb://xxx:xxx@127.0.0.1:7000/admin?readConcernLevel=majority --onlyOplog --oplogStart=1684390978 --oplogEnd=1684391088 --out=dump1 --ssl --sslCAFile=/xxx/mongodb.pem --tlsInsecure
```

mongorestore:

无 SSL: 
```
./bin/mongorestore --uri=mongodb://xxx:xxx@127.0.0.1:7000/admin?readConcernLevel=majority --oplogReplay --onlyOplogReplay --oplogFile=dump1/oplog.bson --dir=dump1
```

有 SSL: 
```
./bin/mongorestore --uri=mongodb://xxx:xxx@127.0.0.1:7000/admin?readConcernLevel=majority --oplogReplay --onlyOplogReplay --oplogFile=dump1/oplog.bson --dir=dump1 --ssl --sslCAFile=/xxx/mongodb.pem --tlsInsecure
```

如何编译
-----------
1. 安装好 Golang，配置好 GOPATH
2. 创建存放代码的目录，如: mkdir -p $GOPATH/src/github.com/mongodb/
3. cd 到上面新建的目录下，git clone 代码，然后 cd mongo-tools:
4. 然后执行编译，编译完成之后的二进制都会放在 ./bin 目录下
```
~/gocode/src/github.com/mongodb/mongo-tools$./make build
```

MongoDB Tools
===================================

 - **bsondump** - _display BSON files in a human-readable format_
 - **mongoimport** - _Convert data from JSON, TSV or CSV and insert them into a collection_
 - **mongoexport** - _Write an existing collection to CSV or JSON format_
 - **mongodump/mongorestore** - _Dump MongoDB backups to disk in .BSON format, or restore them to a live database_
 - **mongostat** - _Monitor live MongoDB servers, replica sets, or sharded clusters_
 - **mongofiles** - _Read, write, delete, or update files in [GridFS](http://docs.mongodb.org/manual/core/gridfs/)_
 - **mongotop** - _Monitor read/write activity on a mongo server_


Report any bugs, improvements, or new feature requests at https://jira.mongodb.org/browse/TOOLS

Building Tools
---------------

We currently build the tools with Go version 1.15. Other Go versions may work but they are untested.

Using `go get` to directly build the tools will not work. To build them, it's recommended to first clone this repository:

```
git clone https://github.com/mongodb/mongo-tools
cd mongo-tools
```

Then run `./make build` to build all the tools, placing them in the `bin` directory inside the repository.

You can also build a subset of the tools using the `-tools` option. For example, `./make build -tools=mongodump,mongorestore` builds only `mongodump` and `mongorestore`.

To use the build/test scripts in this repository, you **_must_** set GOROOT to your Go root directory. This may depend on how you installed Go.

```
export GOROOT=/usr/local/go
```

Updating Dependencies
---------------
Starting with version 100.3.1, the tools use `go mod` to manage dependencies. All dependencies are listed in the `go.mod` file and are directly vendored in the `vendor` directory.

In order to make changes to dependencies, you first need to change the `go.mod` file. You can manually edit that file to add/update/remove entries, or you can run the following in the repository directory:

```
go mod edit -require=<package>@<version>  # for adding or updating a dependency
go mod edit -droprequire=<package>        # for removing a dependency
```

Then run `go mod vendor -v` to reconstruct the `vendor` directory to match the changed `go.mod` file.

Optionally, run `go mod tidy -v` to ensure that the `go.mod` file matches the `mongo-tools` source code.

Contributing
---------------
See our [Contributor's Guide](CONTRIBUTING.md).

Documentation
---------------
See the MongoDB packages [documentation](https://docs.mongodb.org/database-tools/).

For documentation on older versions of the MongoDB, reference that version of the [MongoDB Server Manual](docs.mongodb.com/manual):

- [MongoDB 4.2 Tools](https://docs.mongodb.org/v4.2/reference/program)
- [MongoDB 4.0 Tools](https://docs.mongodb.org/v4.0/reference/program)
- [MongoDB 3.6 Tools](https://docs.mongodb.org/v3.6/reference/program)

Adding New Platforms Support
---------------
See our [Adding New Platform Support Guide](PLATFORMSUPPORT.md).

Vendoring the Change into Server Repo
---------------
See our [Vendor the Change into Server Repo](SERVERVENDORING.md).
