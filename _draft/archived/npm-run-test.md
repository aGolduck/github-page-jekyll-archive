我只是想加个 npm run test...

原来的 Jenkinfile 的 lint 和 test 只是一个 echo，github 上发起 PR checks 直接通过。先是试着加一个 lint 过程。但是默认的 centos 默认的 node 是 6. 以为需要在所有的生产服解决版本的问题，就卡住了。开始弄觉得 nvm 的方案实在是太 low 了。然后尝试阿里的 nodeinstall，有二进制包的问题，根本没法解决。然后想通了，lint 和 test 并不需要在生产服上做，搞复杂了。但是 jenkins 服务器的 node 也很久，没办法直接在 stage 里面直接用 nvm. 并不是一个正常的 shell 环境，也就用不了 nvm 了。直接装了 node 12, 给自己挖了坑 版本太新了，有一些包没有适配好。另外一个后面尝试的过程中 node 10 和 node 8 切换不过去，装的一直是 10. yum clean all 才终于搞定了。遇到个问题，同一特性分支同时向 master 和 develop 发起 pr 时，对应的 jenkins 串了。还以为是 github 的 jenkins 插件有问题，有专门处理 PR 的插件，一通乱找。还找到一个 hudson post build 的，发现和我们的项目配置对不上，才明白 有 jenkinsfile 的不能那样弄。jenkins 项目是有 freestyle, pipeline 等的区别。然后发现串的是同一特性分支发起的 PR。不同的特性分支没有问题，这是个伪 BUG。正常情况下同一个特性分支不会发起两个 PR。jenkins 的配置没有问题。
将测试阶段设置成只在 pr 下进行。剩下的就是 jenkins 所在主机的配置问题了。碰到 sharp 包需要 zlib 的版本比系统提供的高。原本以为是 node 的版本的问题，折腾来，折腾去。网上搜，找到了办法用设置环境变量的方法解决。

其实有些包如果安装过程有些问题的话，可以自己 fork 一个的。要有自己修改源码的意识。
