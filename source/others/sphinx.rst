1.Sphinx 文档基础语法教程
============================
Sphinx 是一种文档工具，它可以令人轻松的撰写出清晰且优美的文档, 由 Georg Brandl 在BSD 许可证下开发. 新版的Python文档 就是由Sphinx生成的， 并且它已成为Python项目首选的文档工具,同时它对 C/C++ 项目也有很好的支持; 并计划对其它开发语言添加特殊支持. 本站当然也是使用 Sphinx 生成的，它采用reStructuredText! Sphinx还在继续开发. 下面列出了其良好特性,这些特性在Python官方文档中均有体现:

#. 丰富的输出格式: 支持 HTML (包括 Windows 帮助文档), LaTeX (可以打印PDF版本), manual pages（man 文档）, 纯文本
#. 完备的交叉引用: 语义化的标签,并可以自动化链接函数,类,引文,术语及相似的片段信息
#. 明晰的分层结构: 可以轻松的定义文档树,并自动化链接同级/父级/下级文章
#. 美观的自动索引: 可自动生成美观的模块索引
#. 精确的语法高亮: 基于 Pygments 自动生成语法高亮
#. 开放的扩展: 支持代码块的自动测试,并包含Python模块的自述文档(API docs)等

Sphinx 使用 reStructuredText 作为标记语言, 可以享有 Docutils 为reStructuredText提供的分析，转换等多种工具.




1.段落::

    使用用空行分隔的一段文本。

2.斜体::

    *星号包裹的文本代表斜体*

*星号包裹的文本代表斜体*

3.加粗::

    **双星号包裹的问题会加粗**

**双星号包裹的问题会加粗**

4.代码高亮::

     public int add(int a,int b){
        return a + b;
     }

5.引用

Lorem ipsum [REC]_ dolor sit amet.

.. [REC] 参考文献, 书,URL 等.

6.图片

.. image:: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201810%2F26%2F20181026191400_cnmqy.thumb.700_0.jpeg&refer=http%3A%2F%2Fb-ssl.duitang.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629651575&t=0cfe06e3ab90b146a9e1110160f749c0


7.列表::

    * 无序列表(*)
        * 子项
        * 子项

    1. 数字有序列表
        1. 子项
        2. 子项

    #. #代表另一种有序列表
        #. 子项
        #. 子项

8.行模块::

    | 这些行
    | 在源文件里
    | 被分隔的一模一样.

9.表格::

    +------------------------+------------+----------+----------+
    | Header row, column 1   | Header 2   | Header 3 | Header 4 |
    | (header rows optional) |            |          |          |
    +========================+============+==========+==========+
    | body row 1, column 1   | column 2   | column 3 | column 4 |
    +------------------------+------------+----------+----------+
    | body row 2             | ...        | ...      |          |
    +------------------------+------------+----------+----------+

+------------------------+------------+----------+----------+
| Header row, column 1   | Header 2   | Header 3 | Header 4 |
| (header rows optional) |            |          |          |
+========================+============+==========+==========+
| body row 1, column 1   | column 2   | column 3 | column 4 |
+------------------------+------------+----------+----------+
| body row 2             | ...        | ...      |          |
+------------------------+------------+----------+----------+

10.简单表格::

    =====  =====  =======
    A      B      A and B
    =====  =====  =======
    False  False  False
    True   False  False
    False  True   False
    True   True   True
    =====  =====  =======

=====  =====  =======
A      B      A and B
=====  =====  =======
False  False  False
True   False  False
False  True   False
True   True   True
=====  =====  =======

11.连接::

    `百度 <http://baidu.com/>`_ 百度的首页

`百度 <http://baidu.com/>`_ 百度的首页

12.章节::

    =================
    这是章节的标题
    =================

=================
这是章节的标题
=================

13.尾注::

    Lorem ipsum [#f1]_ dolor sit amet ... [#f2]_

    .. rubric:: Footnotes

    .. [#f1] 第一条尾注的文本.
    .. [#f2] 第二条尾注的文本.


Lorem ipsum [#f1]_ dolor sit amet ... [#f2]_

.. rubric:: Footnotes

.. [#f1] 第一条尾注的文本.
.. [#f2] 第二条尾注的文本.