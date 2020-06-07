---
title: "Golang项目包依赖管理工具——govendor"
date:  2020-04-24T20:22:03
categories: golang
tags:
- golang
- vendor
---

本文主要介绍Golang项目包依赖管理工具—govendor的基本概念、常见命令以及注意事项。

## Govendor介绍

### 基本概念

Govendor 是Golang 包依赖管理工具。在项目使用Govendor管理依赖包时，当执行 `go build` 或 `go run` 命令时，会按照以下顺序去查找包：

- 当前包下的 vendor 目录
- 向上级目录查找，直到找到 src 下的 vendor 目录
- 在 GOROOT 目录下查找
- 在 GOPATH 下面查找依赖包

Govendor具有以下特性：

- 支持从项目源码中分析出依赖的包，并从 `$GOPATH` 复制到项目的 `vendor` 目录下
- 支持包的指定版本，并用 `vendor/vendor.json` 进行包和版本管理
- 支持用 `govendor add/update` 命令从 `$GOPATH` 中复制依赖包
- 可直接用 `govendor fetch` 添加或更新依赖包
- 可用 `govendor migrate` 从其他 `vendor` 包管理工具中一键迁移到 `govendor`

### 常见命令

#### 项目常见命令介绍

在使用govendor时，当前项目需要在`$GOPATH`路径下。并且Go版本=1.5，需要设置GO15VENDOREXPERIMENT=1。在使用Govendor之前，需要安装Govendor。

```sh
go get -u github.com/kardianos/govendor
```

为了便于项目使用Govendor。建议将 `$GOPATH/bin` 添加到 PATH 中。

```sh
export PATH="$GOPATH/bin:$PATH"
```

1. govendor初始化

   ```
   govendor init
   ```

2. 从远程仓库添加依赖包

   ```
   govendor fetch github.com/tjfoc/gmsm/
   ```

3. 从`$GOPATH`目录下添加依赖包

   ```
   govendor add github.com/tjfoc/gmsm/
   ```

4. 从远程仓库添加包并下载到`$GOPATH`目录

   ```
   govendor get github.com/tjfoc/gmsm/
   ```

5. 基于`vendor.json` 中记录的依赖包信息，拉取更新

   ```
   govendor sync
   ```

####  govendor 子命令介绍

可以使用go vendor --help查看子命令

```sh
Sub-Commands

	init     Create the "vendor" folder and the "vendor.json" file.
	list     List and filter existing dependencies and packages.
	add      Add packages from $GOPATH.
	update   Update packages from $GOPATH.
	remove   Remove packages from the vendor folder.
	status   Lists any packages missing, out-of-date, or modified locally.
	fetch    Add new or update vendor folder packages from remote repository.
	sync     Pull packages into vendor folder from remote repository with revisions
  	             from vendor.json file.
	migrate  Move packages from a legacy tool to the vendor folder with metadata.
	get      Like "go get" but copies dependencies into a "vendor" folder.
	license  List discovered licenses for the given status or import paths.
	shell    Run a "shell" to make multiple sub-commands more efficient for large
	             projects.
```

| 子命令  | 功能                                                         |
| :-----: | ------------------------------------------------------------ |
|  init   | 创建 `vendor` 目录和 `vendor.json` 文件                      |
|  list   | 列出&过滤依赖包及其状态                                      |
|   add   | 从 `$GOPATH` 复制包到项目 `vendor` 目录                      |
| update  | 从 `$GOPATH` 更新依赖包到项目 `vendor` 目录                  |
| remove  | 从 `vendor` 目录移除依赖的包                                 |
| status  | 列出所有缺失、过期和修改过的包                               |
|  fetch  | 从远程仓库添加或更新包到项目 `vendor` 目录(不会存储到 `$GOPATH`) |
|  sync   | 根据 `vendor.json` 拉取相匹配的包到 `vendor` 目录            |
| migrate | 从其他基于 `vendor` 实现的包管理工具中一键迁移               |
|   get   | 与 `go get` 类似，将包下载到 `$GOPATH`，再将依赖包复制到 `vendor` 目录 |
| license | 列出所有依赖包的 LICENSE                                     |
|  shell  | 可一次性运行多个 `govendor` 命令                             |

#### govendor 命令参数

```
Status Types

	+local    (l) packages in your project
	+external (e) referenced packages in GOPATH but not in current project
	+vendor   (v) packages in the vendor folder
	+std      (s) packages in the standard library

	+excluded (x) external packages explicitly excluded from vendoring
	+unused   (u) packages in the vendor folder, but unused
	+missing  (m) referenced packages but not found

	+program  (p) package is a main package

	+outside  +external +missing
	+all      +all packages
```

|   状态    | 缩写 | 含义                                                 |
| :-------: | :--: | ---------------------------------------------------- |
|  +local   |  l   | 本地包，即项目内部编写的包                           |
| +external |  e   | 外部包，即在 `GOPATH` 中、却不在项目 `vendor` 目录   |
|  +vendor  |  v   | 已在 `vendor` 目录下的包                             |
|   +std    |  s   | 标准库里的包                                         |
| +excluded |  x   | 明确被排除的外部包                                   |
|  +unused  |  u   | 未使用的包，即在 `vendor` 目录下，但项目中并未引用到 |
| +missing  |  m   | 被引用了但却找不到的包                               |
| +program  |  p   | 主程序包，即可被编译为执行文件的包                   |
| +outside  |      | 相当于状态为 `+external +missing`                    |
|   +all    |      | 所有包                                               |

支持状态参数的子命令有：`list`、`add`、`update`、`remove`、`fetch`

### 注意事项

+ 如果引用包目录下不存在go文件，使用fetch、get等命令会报错
+ 如果`$GOPATH`目录下存在引用包，使用fetch命令会出现无法下载包的情况