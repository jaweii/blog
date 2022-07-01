title: Proto接口文档生成及自动化
author: 朱嘉伟
thumbnail: /img/poster/protobuf.png
tags: []
categories:
  - WEB向
date: 2021-08-17 21:57:00
---
### 序

我们公司后台用的go，接口都是用proto写的，后端又没有提供接口文档，都是让我们前端自己看 proto 文件的定义和注释。而我以往的经验都是看专门的接口文档站点来调试开发，习惯了看文档以后，初来公司时让我看 proto 代码调试接口，感觉是很不习惯，体验不好。

后来查了一下，发现有工具([protoc-gen-doc](https://github.com/pseudomuto/protoc-gen-doc))可以把 proto 文件的注释转换成文档，于是跟公司后端提了建议，看他们能不能引入这个来给我们前端提供一个接口文档的功能。

提了几次，并无下文，于是自己动手，调配protoc-gen-doc的命令、写文档网站、调配CI/CD后，有了最终成品：<http://fe-x.gitlab.guanmai.cn/gm_api_doc>（内部用）

![](/img/gm_api_doc/gm_api_doc.png)

------------------------------

### 生成文档

protoc-gen-doc可以把后端工程目录中的所有proto文件转换成指定模板的格式。

首先把我们的要生成的目标对象，后端仓库 ceres 克隆下来，文件夹结构大概长这样：
```

├── account
│   ├── account.conf
│   ├── client
│   │   ├── client.go
│   │   └── client_method.go
│   ├── cmd
│   │   ├── account
│   │   │   └── main.go
│   │   └── accountcli
│   │       └── main.go
│   ├── crypto
│   │   └── crypto.go
│   ├── meta
│   │   └── meta.go
│   ├── proto
│   │   ├── account.md5
│   │   ├── account.pb
│   │   ├── account.pb.go
│   │   ├── account.pb.validate.go
│   │   └── account.proto
│   └── server
│       ├── account_test.go
│       ├── database_sharding.go
│       ├── impl.go
│       ├── impl_test.go
│       ├── profile_test.go
│       ├── server.go
│       ├── server_method.go
│       ├── server_test.go
│       └── testing.go
├── analytics
├── enterprise
├── eshop
├── ......
```

ceres目录下有account、analytics、enterprise、eshop等子目录，划分了不同类的接口，且都有一个proto文件夹，里面有对应一个同名 proto 文件，包含了接口的数据结构和注释。

protoc-gen-doc可以把proto文件转换成Markdown、html、DocBook、Json等格式。

要在本地使用proto-doc-gen命令，我们需要安装好Go语言，按照protoc-gen-doc的文档说的安装:

go get -u github.com/pseudomuto/protoc-gen-doc/cmd/protoc-gen-doc

Mac上还要设置几个环境变量:

```
export GOPATH=$HOME/Documents/goworkspace
export PATH=$PATH:$HOME/go/bin
export PATH=$PATH:/usr/local/go/bin
```

现在假设我们有个简单的proto文件，其中有这么一段结构描述:
```
/**
 * Represents the booking of a vehicle.
 *
 * Vehicles are some cool shit. But drive carefully!
 */
message Booking {
  int32 vehicle_id     = 1; /// ID of booked vehicle.
  int32 customer_id    = 2; /// Customer that booked the vehicle.
  BookingStatus status = 3; /// Status of the booking.

  /** Has booking confirmation been sent? */
  bool confirmation_sent = 4;

  /** Has payment been received? */
  bool payment_received = 5;

  string color_preference = 6 [deprecated=true]; // Color preference of the customer.
}
```

然后使用下面的命令，就会在ceres/docs目录下生成index.html：

```
cd ~/Desktop/gm_projects/ceres
protoc --doc_out=./docs --doc_opt=html,index.html **/*.proto --proto_path=/Users/jawei/Desktop/gm_projects/ceres
```

[转换成html后](https://rawgit.com/pseudomuto/protoc-gen-doc/master/examples/doc/example.html)：

![](/img/gm_api_doc/Scalar%20Value%20Types.png)

如果要生成JSON格式的，---doc_opt指定了生成的模板，把html替换成json即可:

```
protoc --doc_out=./docs --doc_opt=json,index.json **/*.proto --proto_path=/Users/jawei/Desktop/gm_projects/ceres
```

[转换成JSON后](https://github.com/pseudomuto/protoc-gen-doc/blob/master/examples/doc/example.json):

```
{
          "name": "Booking",
          "longName": "Booking",
          "fullName": "com.example.Booking",
          "description": "Represents the booking of a vehicle.\n\nVehicles are some cool shit. But drive carefully!",
          "hasExtensions": false,
          "hasFields": true,
          "hasOneofs": false,
          "extensions": [],
          "fields": [
            {
              "name": "vehicle_id",
              "description": "ID of booked vehicle.",
              "label": "",
              "type": "int32",
              "longType": "int32",
              "fullType": "int32",
              "ismap": false,
              "isoneof": false,
              "oneofdecl": "",
              "defaultValue": ""
            },
            {
              "name": "customer_id",
              "description": "Customer that booked the vehicle.",
              "label": "",
              "type": "int32",
              "longType": "int32",
              "fullType": "int32",
              "ismap": false,
              "isoneof": false,
              "oneofdecl": "",
              "defaultValue": ""
            },
            {
              "name": "status",
              "description": "Status of the booking.",
              "label": "",
              "type": "BookingStatus",
              "longType": "BookingStatus",
              "fullType": "com.example.BookingStatus",
              "ismap": false,
              "isoneof": false,
              "oneofdecl": "",
              "defaultValue": ""
            },
            {
              "name": "confirmation_sent",
              "description": "Has booking confirmation been sent?",
              "label": "",
              "type": "bool",
              "longType": "bool",
              "fullType": "bool",
              "ismap": false,
              "isoneof": false,
              "oneofdecl": "",
              "defaultValue": ""
            },
            {
              "name": "payment_received",
              "description": "Has payment been received?",
              "label": "",
              "type": "bool",
              "longType": "bool",
              "fullType": "bool",
              "ismap": false,
              "isoneof": false,
              "oneofdecl": "",
              "defaultValue": ""
            },
            {
              "name": "color_preference",
              "description": "Color preference of the customer.",
              "label": "",
              "type": "string",
              "longType": "string",
              "fullType": "string",
              "ismap": false,
              "isoneof": false,
              "oneofdecl": "",
              "defaultValue": "",
              "options": {
                "deprecated": true
              }
            }
          ]
        }

```

![](/img/gm_api_doc/m%20index.json.png)

这里由于自带的html模板过于简陋，我选择生成JSON格式的Doc，然后自己写一个文档站点来加载JSON文件并渲染。

------------------------

### 文档站开发

文档站我使用UmiJS + Ant Design开发，如图一所示，界面采用三栏设计，左中右分别是proto列表、接口请求和响应、代码参考。数据从上面生成的JSON文件中读取。

源码已经上传到了GitLab仓库https://code.guanmai.cn/fe-x/gm_api_doc。

为了让文档站仓库支持在线访问，我们还需要为其配置Page功能。

在仓库根目录创建一个`.gitlab-ci.yml`的文件，定义一个名字为Pages的特殊Job，并在script中完成clone、build、deploy：

```
stages:
  - deploy

pages:
  image: node:14.15.1
  stage: deploy
  script:
    - node -v
    - yarn -v
    - yarn config set registry http://registry.npm.taobao.org/
    - yarn config set registry https://registry.npmjs.org/
    - git clone https://code.guanmai.cn/jawei/gm_api_doc.git gm_api_doc
    - cd gm_api_doc
    - yarn
    - yarn build
    - cd ..
    - mkdir .public
    - cp -r gm_api_doc/dist/* .public
    - mv .public public
  artifacts:
    paths:
      - public
  only:
    - master
```

这样当master分支有更新后，就会在容器中自动执行上面定义的脚本。脚本中最后把build后的dist文件夹复制到了public目录，并且存在artifacts中，于是我们就可以通过仓库的Settings/pages里给的页面地址来在线访问了:

![](/img/gm_api_doc/Pasted%20Graphic%208.png)

Page的配置可参考：<https://docs.gitlab.com/ee/user/project/pages/getting_started/pages_from_scratch.html>

### 全分支覆盖

在日常工作中，后端同事每天都会更新ceres仓库，接口内容是一直在更新的。

在实际开发中，我们会有多个分支，功能上的更新、Bug的修复，都会开新分支，我们的ceres仓库有多达437个分支。

并且会存在这种情况，比如产品要新增一个修改密码的需求，后端得提供修改密码的接口，就会在master上创个新的分支feature/change_password开发接口，开发完成后进入联调阶段，前端使用的也是新分支的测试环境来联调，这时候新分支上有了修改密码的接口，而master上没有，这时候如果光看master分支的文档是找不到接口的，完成联调后再完成测试阶段，才会合并新分支到master。

所以，文档站的内容实时更新不能只是master上的，所有分支都要支持。所以我们的文档站还得有切换分支的功能。

![](/img/gm_api_doc/Pasted%20Graphic%2010.png)

有同学可能会问，上图中的分支名列表是哪里来的鸭？

当文档站页面打开后，它会先从腾讯云加载一个branch.txt的文本，里面包含了所有的分支名，并渲染在上图的下拉框中。

当用户选择一个分支后，它会再加载一个对应分支的json文件，这个文件就是前面用proto-gen-doc生成的。

一共437个分支，我们要一个一个分支切，再用proto-gen-doc生成。

我写了个shell来完成这个任务，把下面代码保存为generate.sh，和ceres仓库位于同一目录，再执行`sh generate.sh`：

```
#!/bin/bash
echo "generating..."

mkdir doc
branches=$(cd ceres && git branch -r)
for item in $branches;
do
  cd ceres;
  branch=$(cut -c 8- <<< $item);
  git checkout -b $branch origin/$branch;
  git checkout $branch;
  git reset --hard
  sleep 0.5
  echo "checkout $branch";
  cd ..;
  protoc --doc_out=doc --doc_opt=json,${item//\//____}.json --proto_path=ceres/ $(cd ceres && ls */proto/*.proto);
  sleep 0.5
done

echo "done"

```

Shell执行完成后，会生成一个doc目录包含了所有分支的JSON数据，每个json文件大概在4M左右，加起来有2.01G。

将其上传到腾讯云，供文档站读取。

### 自动更新

上面这些完成后，文档的开发就完成了，文档的数据内容也完善了，一个成熟的文档网站将要诞生了。

现在还缺一个文档数据自动更新的功能，因为ceres仓库是一直在更新的，我们不能用前朝的接口请求今朝的服务器。

要实现自动更新，我们还是要用到CI/CD。

同样的，在后端项目的仓库中，创建一个.gitlab-ci.yml文件，配置如下：

```
stages:
  - build
  - deploy

build-structure:
  stage: build
  image:
    name: pseudomuto/protoc-gen-doc
    # disable entrypoint so we can use git clone etc.
    entrypoint: [""]

  before_script:
    - export
    ## 更换镜像源
    - echo '' > /etc/apt/sources.list && echo 'deb http://mirrors.tuna.tsinghua.edu.cn/debian/ jessie main contrib non-free' >> /etc/apt/sources.list && echo 'deb http://mirrors.tuna.tsinghua.edu.cn/debian/ jessie-updates main contrib non-free' >> /etc/apt/sources.list && echo 'deb http://mirrors.tuna.tsinghua.edu.cn/debian-security jessie/updates main contrib non-free' >> /etc/apt/sources.list

  script:
    - echo "$GM_TENCENT_COS_SECRET_ID"
    - apt-get update && apt-get -q -y install git
    - git clone https://code.guanmai.cn/back_end/ceres.git ceres
    - mkdir doc
    - cd ceres
    - git checkout $CI_COMMIT_BRANCH
    - echo $(git branch -r) > ../doc/branch.txt
    - echo "$(git rev-parse --short HEAD)" > ../doc/short_id.txt
    - cd ..
    - protoc --doc_out=doc --doc_opt=json,data.json --proto_path=ceres/ $(cd ceres && ls */proto/*.proto)

  artifacts:
    paths:
      - doc/

upload:
  stage: deploy
  image: guanmaidocker/node-with-coscmd:14-buster
  script:
    - name="origin____${CI_COMMIT_BRANCH//\//____}"
    - coscmd config -a $GM_TENCENT_COS_SECRET_ID -s $GM_TENCENT_COS_SECRET_KEY -b $GM_TENCENT_COS_GUANMAI_PROTO_NAME -r ap-guangzhou
    - coscmd upload doc/data.json ci_test/$name.json
    - coscmd upload doc/branch.txt ci_test/branch.txt
    - coscmd upload doc/short_id.txt ci_test/${name}_id.txt
```

这个CI中定义了build和deploy两个stage，任何分支的更新都会触发此CI。

在CI的build阶段，使用了proto-gen-doc提供的docker镜像，ceres项目会被clone下来，然后用echo命令写出git仓库的远程分支列表以及当前分支的commit id到txt文件中，再用proto-gen-doc生成当前分支的JSON数据保存在doc目录。

在deploy阶段，使用了腾讯的coscmd镜像，将刚才生成的远程分支列表branch.txt文本、接口文档JSON文件、当前分支的commit id文本上传到腾讯云。

保存后，以后这个仓库某个分支有新的push，都会触发CI重新生成对应分支的接口数据并上传到腾讯云，文档站刷新一下，拿到的就是最新的接口数据内容。

至此，一个成熟的，会自己更新的接口文档完成。

---------------

### 流程图

![](/img/gm_api_doc/Pasted%20Graphic%2013.png)

参考链接：
Proto转文档：<https://github.com/pseudomuto/protoc-gen-doc>
GitLab CI/CD文档: <https://docs.gitlab.com/ee/ci/>
GitLab 静态页面部署: <https://docs.gitlab.com/ee/user/project/pages/>
腾讯云对象存储COSCMD工具: <https://cloud.tencent.com/document/product/436/10976>
Shell教程:  <https://www.runoob.com/linux/linux-shell.html>