Sphinx 语法教程
==================

1.段落::

    使用用空行分隔的一段文本。

2.用 *星号* 斜体::

    *文本*

3.用 **双星号** 加粗::

    **文本**


4.代码示例::

     public int add(int a,int b){
        return a + b;
     }

5.引用

Lorem ipsum [REC]_ dolor sit amet.

.. [REC] 参考文献, 书,URL 等.

6.图片

.. image:: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201810%2F26%2F20181026191400_cnmqy.thumb.700_0.jpeg&refer=http%3A%2F%2Fb-ssl.duitang.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1629651575&t=0cfe06e3ab90b146a9e1110160f749c0


7.列表

* 无序列表(*)
    * 子项
    * 子项

1. 数字有序列表
    1. 子项
    2. 子项

#. #代表另一种有序列表
    #. 子项
    #. 子项

8.行模块

| 这些行
| 在源文件里
| 被分隔的一模一样.

9.表格

+------------------------+------------+----------+----------+
| Header row, column 1   | Header 2   | Header 3 | Header 4 |
| (header rows optional) |            |          |          |
+========================+============+==========+==========+
| body row 1, column 1   | column 2   | column 3 | column 4 |
+------------------------+------------+----------+----------+
| body row 2             | ...        | ...      |          |
+------------------------+------------+----------+----------+

10.简单表格

=====  =====  =======
A      B      A and B
=====  =====  =======
False  False  False
True   False  False
False  True   False
True   True   True
=====  =====  =======

11.连接::

`百度 <http://baidu.com/>`_ , 百度的首页

`百度 <http://baidu.com/>`_ , 百度的首页

12.章节

=================
这是章节的标题
=================

13.尾注

Lorem ipsum [#f1]_ dolor sit amet ... [#f2]_

.. rubric:: Footnotes

.. [#f1] 第一条尾注的文本.
.. [#f2] 第二条尾注的文本.



