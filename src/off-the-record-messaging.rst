Off-The-Record Messaging (OTR)
------------------------------

.. _description-11:

描述
~~~~

:term:`OTR 消息传递` 是用于保护人与人之间即时通讯通信的协议。
它旨在成为现实生活中私密对话的在线等价物。它加密消息，防止窃听者
阅读它们。它还对等方进行相互身份验证，因此他们知道在与谁交谈。
尽管进行了身份验证，它被设计为不可否认的：参与者以后可以向第三方
否认他们彼此说过的任何话。它还被设计为具有完美前向安全性：即使
长期公钥对泄露，也不会危及之前的任何对话。

不可否认性和完美前向安全性特性与其他系统（如 OpenPGP）非常不同。
OpenPGP 有意保证不可否认性。如果你签署软件包、在邮件列表上发言或
签署商业发票，这是一个很好的特性，但 :term:`OTR` 的作者认为这些不是
一对一在线对话的可取特性。此外，OpenPGP 的静态通信模型使得实现
:term:`OTR` 完美前向安全性所需的持续密钥重新协商成为不可能。

:term:`OTR` 通常配置为机会主义，这意味着如果双方都理解协议，它将尝试
保护任何两个对等方之间的通信，而不与不理解的对等方通信。该协议
在许多不同的即时通讯客户端中得到支持，无论是直接支持还是通过插件。
由于它通过即时消息工作，它可以跨许多不同的即时通讯协议使用。

对等方可以通过显式消息（称为 :term:`OTR` 查询消息）来信号他们希望与
对方进行 :term:`OTR` 对话。如果对等方只是愿意进行 :term:`OTR` 但不要求，
他们可以选择不可见地将该信息添加到明文消息中。这是通过巧妙的空白
标签系统完成的：使用一系列空白字符（如空格和制表符）编码该信息。
支持 :term:`OTR` 的客户端可以解释该标签并启动 :term:`OTR` 对话；不支持
:term:`OTR` 的客户端只显示一些额外的空白。

:term:`OTR` 使用我们到目前为止看到的许多基本构件：

- 对称密钥加密（CTR 模式下的 AES）
- :term:`消息认证码 <message authentication code>`（带 SHA-1 的 HMAC）
- Diffie-Hellman 密钥交换

:term:`OTR` 还使用另一种称为 SMP 的机制来检查对等方是否到达了相同的
共享密钥。

.. _key-exchange-1:

密钥交换
~~~~~~~~

在 :term:`OTR` 中，AKE 严重依赖于 Diffie-Hellman 密钥交换，并通过大量
额外的互锁检查进行扩展。Diffie-Hellman 交换本身使用固定的 1536 位
素数和固定的生成器 :math:`g`。

假设有两个参与者，名为 Alice 和 Bob，想要通信并愿意交换敏感数据。
Alice 和 Bob 各自拥有一对长期 DSA 身份验证密钥对，我们分别称为
:math:`(p_A, s_A)` 和 :math:`(p_B, s_B)`。

该协议还依赖于许多其他基本构件：

- 128 位分组密码。在 :term:`OTR` 中，这始终是 AES。在本节中，我们将
  分组密码加密和解密分别称为 :math:`E` 和 :math:`D`。
- 哈希函数 :math:`H`。在 :term:`OTR` 中，这是 SHA1。
- :term:`消息认证码` :math:`M`。在 :term:`OTR` 中，这是 HMAC-SHA1。
- 签名函数 :math:`S`。

提交消息
^^^^^^^^^^^^^

最初 Alice 和 Bob 处于协议状态，等待对等方启动 :term:`OTR` 连接，并
公布他们自己支持 :term:`OTR` 的能力。

假设 Bob 选择与 Alice 启动 :term:`OTR` 对话。他的客户端发送 :term:`OTR`
提交消息，然后转换到等待 Alice 客户端回复的状态。

要发送提交消息，客户端选择一个随机的 128 位值 :math:`r` 和一个随机
的 320 位（或更大）Diffie-Hellman 秘密 :math:`x`。然后它发送
:math:`E(r, g^x)` 和 :math:`H(g^x)` 给对等方。

密钥消息
^^^^^^^^^^^

Alice 的客户端已收到 Bob 客户端启动 :term:`OTR` 会话的通告。她的客户端
用密钥消息回复，这涉及创建新的 Diffie-Hellman 密钥对。她选择一个
320 位（或更大）Diffie-Hellman 秘密 :math:`y` 并发送 :math:`g^y` 给 Bob。

公开签名消息
^^^^^^^^^^^^^^^^^^^^^^^^

现在 Alice 已发送她的 Diffie-Hellman 公钥，Bob 可以完成他的一部分
Diffie-Hellman 协议。Alice 还不能继续，因为她还没看到 Bob 的公钥。

当我们讨论 Diffie-Hellman 时，我们注意到它不*身份验证*对等方。
Bob 可以计算一个秘密，但不知道他在与 Alice 交谈。与 TLS 和使用
Diffie-Hellman 的其他系统一样，这个问题通过身份验证密钥交换解决。

验证 Alice 的公钥是有效值后，Bob 计算共享秘密 :math:`s = (g^y)^x`。
使用密钥派生函数，他从 :math:`s` 派生多个密钥：两个 AES 密钥
:math:`c, c^\prime` 和四个 MAC 密钥
:math:`m_1, m_1^\prime, m_2, m_2^\prime`。

他为当前 Diffie-Hellman 密钥对 :math:`(x, g^x)` 选择一个标识号
:math:`i_B`。一旦 Alice 和 Bob 生成新的密钥对，这将在后面的 :term:`OTR` 协议中变得重要。

Bob 计算：

.. math::

   M_B = M_{m_1}(g^x, g^y, p_B, i_B)

.. math::

   X_B = (p_B, i_B, S(p_B, M_B))

他发送给 Alice :math:`r, E_c(X_B), M_{m_2}(E_c(X_B))`。

签名消息
^^^^^^^^^^^^^^^^^

Alice 现在可以确认她直接与 Bob 交谈，因为 Bob 用他的长期 DSA 密钥
签名的交换认证器 :math:`M_B`。

Alice 现在也可以计算共享秘密：Bob 发送给她 :math:`r`，这之前用于加密
Bob 的 Diffie-Hellman 公钥。然后她自己计算 :math:`H(g^x)`，与 Bob
发送的进行比较。通过完成她这边的 Diffie-Hellman 交换
(:math:`s = (g^x)^y`)，她派生出相同的密钥：
:math:`c, c^\prime, m_1, m_1^\prime, m_2, m_2^\prime`。使用 :math:`m_2`，她
可以验证 :math:`M_{m_2}(E_c(X_B))`。一旦该消息被验证，她可以使用她
计算出的 :math:`c` 安全地解密它。

然后她也可以计算 :math:`M_B = M_{m_1}(g^x, g^y, p_B, i_B)`，并验证它
与 Bob 发送的相同。通过验证 Bob 公钥上签名的部分 :math:`S(p_B, M_B)`，
她现在已经明确地将当前交互与 Bob 的长期身份验证密钥联系起来。

然后她计算 Bob 计算的相同值，以将他的长期密钥与短期握手联系起来，
这样 Bob 也可以验证她。她为当前 DH 密钥对 :math:`(y, g^y)` 选择一个
标识号 :math:`i_A`，计算 :math:`M_A = M_{m_1^\prime}(g^y, g^x, p_A, i_A)`
和 :math:`X_A = p_A, i_A, S(p_A, M_A)`。最后，她发送给 Bob
:math:`E_{c^\prime}(X_A), M_{m_2^\prime}(E_c(X_B))`。

身份验证 Alice
^^^^^^^^^^^^^^^^^^^^

现在 Bob 也可以验证 Alice，同样通过镜像步骤。首先，他验证
:math:`M_{m_2^\prime}(E_c(X_B))`。这让他可以检查 Alice 看到了相同的
:math:`X_B` 他发送的。

一旦他解密 :math:`E_{c^\prime}(X_A)`，他就有了访问 :math:`X_A` 的权限，
这是 Alice 的长期公钥信息。然后他可以计算
:math:`M_A = M_{m_1^\prime}(g^y, g^x, p_A, i_A)` 与 Alice 发送的版本进行比较。
最后，他用 Alice 的公钥验证 :math:`S(p_A, M_A)`。

我们完成了什么？
^^^^^^^^^^^^^^^^^^^^^^^^^^

如果所有检查都成功，Alice 和 Bob 已经完成了经过身份验证的 Diffie-Hellman
交换，并拥有只有他们两人知道的共享秘密。

现在你已经看到了经过身份验证握手的双方，你可以理解为什么从 Diffie-Hellman
秘密派生出如此多的不同密钥。带素数标记（:math:`\prime`）的密钥用于
第二个对等方（响应通告的对等方，在我们的例子中是 Alice）发起的消息；
不带素数的密钥用于发起对等方（在我们的例子中是 Bob）。

数据交换
~~~~~~~~~

TODO: Explain (https://otr.cypherpunks.ca/Protocol-v3-4.0.0.html), #33
