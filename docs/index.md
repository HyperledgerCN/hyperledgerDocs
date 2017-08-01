***欢迎阅读！！！***


***欢迎贡献！！！***

## 简介

**Hyperledger国际化工作组**是Hyperledger中国工作组(TWGC)下属的一个小组，主要负责相关文档的中文编写和翻译，以及组织讨论、教育培训活动等。

目前小组有成员100余名，活跃贡献者20余名，已完成文章30余篇。

## 如何贡献

以前我们用[Hyperledger Wiki](https://wiki.hyperledger.org/groups/twgc/team_ie)管理文章，但Wiki读写操作有诸多不便，为此我们将文档转移到[github上](https://github.com/ChainNova/hyperledgerDocs)，以使大家更方便地阅读和编辑贡献资源。

***贡献内容包括但不限于：文档翻译、知识总结、经验教训、好文链接、奇思妙想...***

***如果您不想作如下操作，可将直接内容邮件（见页面最下方）发送给工作组，我们为您发布***

### 加入组织

1. 加入微信群

    目前微信群已超一百人，只能通过邀请方式加入。您可以请认识的小伙伴拉你入群，也可以联系管理员（见页面最下方）。

2. 加入wiki

    Hyperledger Wiki是官方的信息渠道，所以请将您的信息加入其中。[点击进入](https://wiki.hyperledger.org/groups/twgc/team_ie)，登陆，然后编辑`Volunteers`表格，将自己的信息写入并保存。

### 贡献资源

目前以[github](https://github.com)管理文档，以[github pages](https://pages.github.com/)展示文档，以[MkDocs](http://www.mkdocs.org/)构建文档。其中文档都是以Markdown编写。

#### 准备

1. github账号
2. [安装git](https://git-scm.com/book/zh/v1/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
3. 安装[MkDocs](http://www.mkdocs.org/)（可参照[中文文档](http://markdown-docs-zh.readthedocs.io/zh_CN/latest/)）

#### 本地编辑&预览

1. 下载源码

        git clone https://github.com/ChainNova/hyperledgerDocs.git

2. 编辑预览

    **注意：**文档开头固定以下格式：


        | 原文 | 作者 | 审核修正 |
        | --- | --- | —--- |
        | [原文](<原文路径>) | <如果你是作者，请在此留名> | <如果你是修改者，请在此留名，可以多个> |
    
    图片放到`hyperledgerDocs/docs/img`里，文档中以`img/xx.png`引用。

    * 修改已有文档：进入`hyperledgerDocs/docs`目录，编辑对应文件。

    * 添加新文档：进入`hyperledgerDocs/docs`目录，添加新的`Markdown文件`并编辑内容；然后编辑`mkdocs.yml`，将新加文档按如下格式添加到配置文件中。

            pages:
                - 欢迎: index.md
                - 词汇表: glossary.md
                - 快速入门: getting_started.md
                - 协议规范: protocol-spec_zh.md
                - Fabric教程:
                    - 构建第一个fabric网络: build_network_zh.md
                    - 编写第一个应用: write_first_app_zh.md
                    - Chaincode: chaincode_zh.md
    
    * 本地预览：在`hyperledgerDocs`目录下执行
    
            mkdocs serve
        
        然后浏览器打开`http://127.0.0.1:8000/`找到相应页面。

#### 线上提交&部署

本地预览无误后，即可提交到线上供大家阅读。

在`hyperledgerDocs`目录下执行
    
    ./build.sh

如无报错，浏览器打开`https://chainnova.github.io/hyperledgerDocs/`查看修改结果。

**线上确认成功后，千万不要忘了将本地修改的源文件提交到github仓库：**

    git add .
    git commit -m "your message"
    git push