 Crypto 101: 中文翻译版
======================

.. image:: https://github.com/grill-glitch/book/actions/workflows/ci.yml/badge.svg?branch=zh_CN-translation
   :target: https://github.com/grill-glitch/book/actions/workflows/ci.yml?branch=zh_CN-translation

这是 `Crypto 101`_ 的中文翻译版本。Crypto 101 是由 lvh_ 编写的密码学入门教材。

.. _`Crypto 101`: https://www.crypto101.io/
.. _lvh: https://twitter.com/lvh

原始英文版本: https://github.com/crypto101/book

许可证
======

详见 `LICENSE <LICENSE>`_ 文件。

翻译状态
=========

本项目目前已完成中文翻译，涵盖以下内容：

- 全书主体章节已翻译（部分章节仍有 TODO 标记，待补充）
- 支持格式：HTML（网站）、PDF（打印阅读）
- EPUB 格式因公式渲染问题暂不支持

构建
====

在仓库根目录运行 ``make book`` 可将源文件转换为所有支持的格式。

依赖
----

由于依赖较多，强烈建议使用 Docker：

.. code-block:: sh

   docker build -t crypto101 docker/
   docker run --rm -it -v "$(realpath .)":/repo -u "$(id -u)" crypto101 ./make-lang zh_CN html latexpdf

语言代码必须是有效的 `Sphinx 语言代码
<https://www.sphinx-doc.org/en/master/usage/configuration.html#confval-language>`_，
例如 ``en``、``fr``、``ko`` 或 ``zh_CN``。

您可以在各自的 Dockerfile 中查找 `ubuntu <docker/Dockerfile.ubuntu>`_ 和
`fedora <docker/Dockerfile.fedora>`_ 的依赖安装说明。

中文版发布
==========

- GitHub Release: https://github.com/grill-glitch/book/releases/tag/zh_CN-20260321-v4
- 在线浏览: https://crypto.notarobot.ggff.net

贡献
====

欢迎提交 Issue 和 Pull Request 帮助改进翻译或修复问题。
