
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| 无 | 于林生 |  |

## Zanata

[Zanata](http://zanata.org/)是一个基于网络的翻译平台，用于翻译者、内容创作者和开发人员来管理本地化项目。

为了更好的管理、推广、翻译Hyperledger国际化文档，工作组尝试使用zanata翻译平台。

* [Zanata官方快速使用指南](http://docs.zanata.org/en/release/user-guide/translator-guide/)
* [Zanata官方文档](http://docs.zanata.org/en/release/)

## 用Zanata翻译Hyperledger文档

1. 注册用户并登陆
将您的Zanata ID添加至项目组[wiki页面](https://wiki.hyperledger.org/groups/twgc/team_ie)。我们会将您加入Zanata翻译工作组中。若您没有被及时加入该组，欢迎联系Linsheng Yu或Jiannan Guo。
2. 查找并单击进入项目`hyperledger-fabric-docs`
![](img/zanata-1.jpeg)
3. 单击选择版本号
![](img/zanata-2.jpg)
4. 单击选择语言
![](img/zanata-3.jpg)
5. 单击选择要翻译的文件
![](img/zanata-4.jpg)
6. 在右侧编辑翻译

    在右侧输入译文，编辑完后会自动保存；下侧是翻译提示内容，可直接`copy`；也可以点击`使用新版`体验新版翻译页面。

    **注意：翻译中的固定术语尽量按照翻译提示统一命名，以免混乱！！！**
![](img/zanata-5.jpg)

    新版翻译页面
![](img/zanata-6.jpg)

**7. 原文更新及译后文档的展示操作，之后补充说明**


## 问题

目前编者对Zanata并不够熟悉，还有很多问题没有解决，请大家多多指教。

1. Zanata文档与github的关联，如何自动更新原文，如何将疑问自动push到github仓库。
2. 新用户如何能直接翻译文档，而无需管理者事先将译者用户加入到项目中。
3. Zanata不支持fabric官方文档的`rst`格式，目前是将之转换为`html`格式，导致Zanata上的原文包含很多重复信息（如目录、网页标题等）。
4. 