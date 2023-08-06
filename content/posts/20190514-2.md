---
title: "在Gitlab CI中检测Golang单元测试覆盖率"
date: 2019-05-14T00:00:00+08:00
draft: false
---

我今天尝试给Gitlab上的golang工程，在CI过程中加入单元测试覆盖率的显示，在此记录一下。

首先，要在一台机器上安装gitlab runner，并根据project settings中的token对runner进行注册。这个过程在网上有很多，不再赘述。

gitlab ci的使用是非常方便的，只需要在项目根目录下设置.gitlab-ci.yml文件即可。在提交commit时和提交MR时，将自动触发在这个yml文件中定义的CI过程。

直接给出我当前使用的yml文件内容：

```
stages:
  - test
before_script:
  - GOPROXY=https://goproxy.io go mod download
  
lint:
  stage: test
  script:
    - diff -u <(echo -n) <(gofmt -d .)
    - go vet -composites=false $(go list ./... | grep -v /vendor/)
    - go test -race $(go list ./... | grep -v /vendor/) -v -coverpkg=all
  tags:
    - xxx
```

如果gitlab runner服务器是在本地或是在国内的云服务器环境里，那怎么能正常访问互联网下载go依赖就是个问题。幸好，从golang 1.11开始，go mod功能能使用代理，只需要在相关明星前设置好代理，即可正常在国内下载go依赖。也就是GOPROXY=https://goproxy.io go mod download这句话。

如果是用go get的方式下载依赖，在正常访问互联网这方面会有些麻烦，可能需要设置http代理。proxychains对go get工具是不生效的。所以还是推荐用go mod + 代理的方式下载go依赖。主要是golang.org, gopkg.in等域名被墙。

中间的三行命令，其中第一行是gofmt检测用的，第二行是govet检测，主要就是看代码格式有没有什么低级的问题。

第三行，就是运行单元测试并显示代码覆盖率。注意，最后的-coverpkg=all参数，如果想要显示总的覆盖率，就要使用此参数。比如，如果使用-cover，就只能显示各个package各自的覆盖率。

另外注意，这个yml文件里的tags，需要和注册runner时指定的tag一致，这样才能正常触发runner运行。

除此之外，如果想在MR界面中显示代码覆盖率，还需要设置gitlab project settings的Test coverage parsing。就用提示文本中给出的coverage: \d+.\d+% of statements即可。

这样一来，在提交MR时，就可以显示出单元测试的总覆盖率了。