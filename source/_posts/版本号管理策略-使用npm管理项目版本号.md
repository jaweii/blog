title: 版本号管理策略&&使用npm管理项目版本号
author: 朱嘉伟
tags:
  - 原创
  - 教程
  - 版本
categories:
  - WEB向
date: 2018-03-08 17:45:00
---
# 命令

在Node.js项目中的前后端项目中，版本号管理使用的是NPM的命令——别跟我说，你是手动改package来更新版本号的。

在命令行敲入`npm version ?`就可以看到可以使用的命令：

```
npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease | from-git]
```

major：主版本号

minor：次版本号

patch：补丁号

premajor：预备主版本

prepatch：预备次版本

prerelease：预发布版本

我的package.jsond的**当前version为6.0.0**，依次输入下面的命令，package的version会变更为提升后的版本号：

```dos
C:\Users\Administrator\Desktop\work\stage-view (master) (stage-view@6.0.0)
λ npm version preminor
v6.1.0-0

C:\Users\Administrator\Desktop\work\stage-view (master) (stage-view@6.1.0-0)
λ npm version minor
v6.1.0

C:\Users\Administrator\Desktop\work\stage-view (master) (stage-view@6.1.0)
λ npm version prepatch
v6.1.1-0

C:\Users\Administrator\Desktop\work\stage-view (master) (stage-view@6.1.1-0)
λ npm version patch
v6.1.1

C:\Users\Administrator\Desktop\work\stage-view (master) (stage-view@6.1.1)
λ npm version prerelease
v6.1.2-0

C:\Users\Administrator\Desktop\work\stage-view (master) (stage-view@6.1.2-0)
λ npm version premajor
v7.0.0-0

C:\Users\Administrator\Desktop\work\stage-view (master) (stage-view@7.0.0-0)
λ npm version major
v7.0.0
```

如上所示，敲入`npm version preminor`，项目的version就从6.0.0变成了6.1.0-0。

对了，项目的git status必须是clear，才能使用上述命令。

如果你的项目中包含git，它还会自动给你提交更新到git，`git commit -m "X.Y.Z"`，所以还可以在npm version NEWVERSION 后面加上-m参数来指定自定义的commit message。比如：

```
npm version patch -m "Upgrade to %s for reasons"
```

message中的s%将会被替换为版本号。

# 版本号策略

版本号格式：主版本号**.**次版本号**.**修订号；

主版本号：当你做了不兼容的 API 修改；

次版本号：当你做了向后兼容的功能性新增；

修订号：当你做了向后兼容的问题修正；

处于开发阶段的项目版本号以0.Y.Z形式表示，此阶段正在开发基础功能、公众API；

版本号只能增加，禁止下降，代码的修改必须以新版本形式更新；

如何判断发布 1.0.0 版本的时机？ 当你的软件被用于正式环境，它应该已经达到了 1.0.0 版。如果你已经有个稳定的 API 被使用者依赖，也会是 1.0.0 版。如果你很担心向下兼容的问题，也应该算是 1.0.0 版了。

万一不小心把一个不兼容的改版当成了次版本号发行了该怎么办？一旦发现自己破坏了语义化版本控制的规范，就要修正这个问题，并发行一个新的次版本号来更正这个问题并且恢复向下兼容。即使是这种情况，也不能去修改已发行的版本。


# 编程式

在项目代码中有时候需要判断当前版本，可以通过读取package文件获取当前版本：

```
import {version} from './package.json'
```

要判断两个版本号字符串的大小，可以使用插件 compare-versions :

```
compareVersions('10.1.8', '10.0.4'); // 1
compareVersions('10.0.1', '10.0.1'); // 0
```  



# 自动更新版本号

在项目目录的`.git/hooks/`目录中新建文件: `post-commit`——是的，没有后缀名。
然后粘贴以下代码并保存文件：
```
#!/bin/sh

COMMIT_MSG="$(git log --pretty=format:"%s" -1 head)"

echo "$COMMIT_MSG" | grep  -q  "^[0-9]"
if [ $? -ne 0 ];then
  echo $(npm version patch)
fi

```
上面代码会在每次`git commit` 执行后被运行，它检查commit的message是不是版本号，如果不是，它就会执行`npm version patch`更新版本号。

---


参考链接：

https://docs.npmjs.com/cli/version

https://semver.org/lang/zh-CN/