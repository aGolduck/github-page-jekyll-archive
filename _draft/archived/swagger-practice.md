---
title: swagger 在 node 项目中的实践
tags: [swagger, openapi, node]
---
Swagger, 现在也称 openapi 定义了规范的 RESTFUL 接口文档标准，围绕这个标准产生了一个很丰富的生态圈。看看如何在 node 项目中应用这些工具。

https://apis.guru/awesome-openapi3/

首先最基础的是 swagger 的文档功能。如何展示 swagger 文档？swagger ui 已经不错，界面较为美观，而且能够交互性地操作，产生对应的 curl 命令，很大程度上可以取代 postman 等工具。 目前的缺点是接口数量多了之后应当如何组织。可以直接将 swagger 的文件扔给能够处理的工具，直接显示，这种不够灵活。另外一种就是自行组装好 openapi 的对象，再调用工具函数显示出来。

关于文档如何书写的问题。api 文档往往会急骤膨胀。目前用到的策略是用 @swagger jsdoc 书写具体的接口。其它的 components 可以由多个 yaml 文件书写，swagger-jsdoc 能够将各个文件的内容组装起来。

写文档的时候记得尽可能使用 $ref, allOf 等， 提高文档书写的可复用性。

可以利用 swagger 对出参入参进行基本的类型和是否必须进行校验。可以用 oas-chow-chow. 进行校验的 openapi 必须严格遵守规范，最好使用 oas-validator 先进行校验。

还有一些问题需要解决。

错误响应怎么处理？

大量的 api 应该怎么组织。
