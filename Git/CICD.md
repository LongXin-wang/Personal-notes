- [概念](#概念)

# 概念

CICD的全称是持续集成（Continuous Integration）和持续交付（Continuous Delivery）。这是一套软件开发实践，旨在通过自动化软件构建、测试和部署流程，实现快速、高质量的软件交付。其主要流程包括：

持续集成（CI）：开发人员将代码频繁地集成到共享存储库中，并通过自动化构建和测试流程来验证代码的变更。持续集成完成了构建、单元测试和集成测试这些自动化流程。

持续交付（CD）：通过自动化的部署流程，将经过验证的代码变更部署到生产环境中，以便快速、可靠地交付新功能和更新。

整个流程的核心在于自动化，包括自动化构建、自动化测试、自动化部署


![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121635694.png)

# DevOps (Development Operations 开发运维)

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121629822.png)


# 实践

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121631624.png)

逻辑就是...gitlab发送合并请求到CI，CI拉取做一系列处理，然后审批完成之后走后置命令(docker build,docker push)；

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403122035578.png)

CI走完，然后质测平台会有脚本，CI走完之后，看是否有CD任务需要处理，有的话，就会走CD任务调用丹炉那边的部署
