.. _key-exchange:

密钥交换
--------

.. _description-3:

描述
~~~~

:term:`密钥交换` 协议试图解决一个乍看似乎不可能的问题。Alice 和 Bob，
他们从未见过面，必须就一个秘密值达成一致。他们用来通信的渠道是不安全
的：我们假设他们通过渠道发送的所有内容都被窃听了。

我们在这里将演示这样一个协议。Alice 和 Bob 最终将拥有一个共享秘密，
仅通过不安全渠道通信。尽管 Eve 拥有 Alice 和 Bob 相互发送的所有信息，
她也无法使用任何这些信息来找出他们的共享秘密。
secret.

That protocol is called Diffie-Hellman, named after Whitfield Diffie and
Martin Hellman, the two cryptographic pioneers who discovered it. They
suggested calling the protocol Diffie-Hellman-Merkle :term:`key exchange`, to
honor the contributions of Ralph Merkle. While his contributions
certainly deserve honoring, that term hasn't really caught on. For the
benefit of the reader we'll use the more common term.

Practical implementations of Diffie-Hellman rely on mathematical
problems that are believed to be very complex to solve in the “wrong”
direction, but easy to compute in the “right” direction. Understanding
the mathematical implementation isn't necessary to understand the
principle behind the protocol. Most people also find it a lot easier to
understand without the mathematical complexity. So, we'll explain
Diffie-Hellman in the abstract first, without any mathematical
constructs. Afterwards, we'll look at two practical implementations.

Abstract Diffie-Hellman
~~~~~~~~~~~~~~~~~~~~~~~

In order to describe Diffie-Hellman, we'll use an analogy based on
mixing colors. We can mix colors according to the following rules:

-  It's very easy to mix two colors into a third color.
-  Mixing two or more colors in different order results in the same
   color.
-  Mixing colors is *one-way*. It's impossible to determine if, let
   alone which, multiple colors were used to produce a given color. Even
   if you know it was mixed, and even if you know some of the colors
   used to produce it, you have no idea what the remaining color(s)
   were.

We'll demonstrate that with a mixing function like this one, we can
produce a secret color only known by Alice and Bob. Later, we'll simply
have to describe the concrete implementation of those functions to get a
concrete :term:`key exchange` scheme.

To illustrate why this remains secure in the face of eavesdroppers,
we'll walk through an entire exchange with Eve, the eavesdropper, in the
middle. Eve is listening to all of the messages sent across the network.
We'll keep track of everything she knows and what she can compute, and
end up seeing *why* Eve can't compute Alice and Bob's shared secret.

To start the protocol, Alice and Bob have to agree on a base color. They
can communicate that across the network: it's okay if Eve intercepts the
message and finds out what the color is. Typically, this base color is a
fixed part of the protocol; Alice and Bob don't need to communicate it.
After this step, Alice, Bob and Eve all have the same information: the
base color.

.. figure:: ./Illustrations/DiffieHellman/alice-bob-eve.svg
   :align: center

Alice and Bob both pick a random color, and they mix it with the base
color.

.. figure:: ./Illustrations/DiffieHellman/alice-bob-secret.svg
   :align: center

At the end of this step, Alice and Bob know their respective secret
color, the mix of the secret color and the base color, and the base
color itself. Everyone, including Eve, knows the base color.

.. figure:: ./Illustrations/DiffieHellman/alice-bob-eve-secret.svg
   :align: center

Then, Alice and Bob both send their mixed colors over the network. Eve
sees both mixed colors, but she can't figure out what either of Alice
and Bob's *secret* colors are. Even though she knows the base, she can't
“un-mix” the colors sent over the network. [#]_

.. [#]
   虽然使用黑白近似颜色混合看起来很容易，但请记住这只是插图的
   失败：我们的假设是这是困难的。

.. figure:: ./Illustrations/DiffieHellman/mixed-secret.svg
   :align: center

.. [#]
   While this might seem like an easy operation with black-and-white
   approximations of color mixing, keep in mind that this is just a
   failure of the illustration: our assumption was that this was hard.


At the end of this step, Alice and Bob know the base, their respective
secrets, their respective mixed colors, and each other's mixed colors.
Eve knows the base color and both mixed colors.

.. figure:: ./Illustrations/DiffieHellman/alice-bob-eve-mixed.svg
   :align: center


Once Alice and Bob receive each other's mixed color, they add their own
secret color to it. Since the order of the mixing doesn't matter,
they'll both end up with the same secret.

.. figure:: ./Illustrations/DiffieHellman/alice-bob-shared-mixed.svg
   :align: center

Eve can't perform that computation. She could finish the computation
with either Alice or Bob's secret color, since she has both mixed
colors, but she has neither of those secret colors. She can also try to
mix the two mixed colors, which would have both Alice and Bob's secret
colors mixed into them. However, that would have the base color in it
twice, resulting in a different color than the shared secret color that
Alice and Bob computed, which only has the base color in it once.

基于离散对数的 Diffie-Hellman
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

本节描述基于离散对数问题的 Diffie-Hellman 算法的实际实现。它旨在
提供一些数学背景，需要模运算来理解。如果您不熟悉模运算，您可以
跳过本章，或先阅读 :ref:`数学背景附录<modular-arithmetic>`。

离散对数 Diffie-Hellman 基于计算以下等式中的 :math:`y` 很容易
（至少对计算机而言）的想法：

.. math::

   y \equiv g^x \pmod{p}

然而，给定 :math:`y`、:math:`g` 和 :math:`p` 计算 :math:`x` 被认为
是非常困难的。这称为离散对数问题，因为没有模运算的类似操作称为对数。

这只是我们前面讨论的抽象 Diffie-Hellman 过程的具体实现。共同基础
颜色是一个大素数 :math:`p` 和基数 :math:`g`。"颜色混合"操作是上面
给出的等式，其中 :math:`x` 是输入值，:math:`y` 是结果混合值。

当 Alice 或 Bob 选择他们的随机数 :math:`r_A` 和 :math:`r_B` 时，
他们将其与基础颜色混合以产生混合数字 :math:`m_A` 和 :math:`m_B`：

.. math::

   m_A \equiv g^{r_A} \pmod{p}

.. math::

   m_B \equiv g^{r_B} \pmod{p}

这些数字通过网络发送，Eve 可以看到它们。离散对数问题的前提是这样
做是安全的，因为在 :math:`m \equiv g^r \pmod{p}` 中找出 :math:`r`
据说是非常困难的。

一旦 Alice 和 Bob 拥有彼此的混合数字，他们就会添加自己的秘密数字。
例如，Bob 将计算：

.. math::

   s \equiv (g^{r_A})^{r_B} \pmod{p}

虽然 Alice 的计算看起来不同，但他们得到相同的结果，因为
:math:`(g^{r_A})^{r_B} \equiv (g^{r_B})^{r_A} \pmod{p}`。这是
共享秘密。
the shared secret.

Because Eve doesn't have :math:`r_A` or :math:`r_B`, she can not perform
the equivalent computation: she only has the base number :math:`g` and
mixed numbers :math:`m_A \equiv g^{r_A} \pmod{p}` and
:math:`m_B \equiv g^{r_B} \pmod{p}` , which are useless to her. She
needs either :math:`r_A` or :math:`r_B` (or both) to make the
computation Alice and Bob do.

TODO: 谈谈主动 MITM 攻击，其中攻击者选择平滑值以产生弱秘密？

椭圆曲线 Diffie-Hellman
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

本节描述基于椭圆曲线离散对数问题的 Diffie-Hellman 算法的实际实现。
它旨在提供一些数学背景，需要（非常基本的）理解椭圆曲线密码学背后的
数学。如果您不熟悉椭圆曲线，您可以跳过本章，或先阅读:ref:`数学背景
附录<elliptic-curves>`。

椭圆曲线 Diffie-Hellman 变体的好处之一是所需密钥大小比基于离散对数
问题的变体小得多。这是因为攻击椭圆曲线离散对数问题的最快算法比
非椭圆变体具有更大的渐近复杂性。例如，用于离散对数的数域筛，
一种攻击基于离散对数的 Diffie-Hellman 的最先进算法，具有时间复杂度：

.. math::

   L\left[1/3,\sqrt[3]{64/9}\right]

这比数字数多于多项式（但少于指数）。另一方面，可用于破解椭圆曲线
离散对数问题的最快算法都具有复杂性：

.. math::

   L\left[1, 1/2\right] = O(\sqrt{n})

相对而言，这意味着使用两种的最先进算法，解决椭圆曲线问题比解决
常规离散对数问题困难得多。另一方面是，对于等效的安全级别，椭圆曲线
算法需要小得多的密钥大小 :

.. [#]
   这些数字实际上是针对 RSA 问题与等效椭圆曲线问题的，但它们
   的安全级别足够接近以给您一个概念。

.. [#]
   These figures are actually for the RSA problem versus the equivalent
   elliptic curve problem, but their security levels are sufficiently
   close to give you an idea.

====================== ===================== =======================
Security level in bits Discrete log key bits Elliptic curve key bits
====================== ===================== =======================
56                     512                   112
80                     1024                  160
112                    2048                  224
128                    3072                  256
256                    15360                 512
====================== ===================== =======================

.. _remaining-problems-3:

剩余问题
~~~~~~~~

使用 Diffie-Hellman，我们可以通过不安全的互联网就共享秘密达成一致，
免受窃听者攻击。然而，虽然攻击者可能无法仅通过窃听获取秘密，但主动
攻击者仍然可以破坏系统。如果这样的攻击者，通常称为 Mallory，位于
Alice 和 Bob 之间，她仍然可以执行两次 Diffie-Hellman 协议：一次
与 Alice，其中 Mallory 假装是 Bob，一次与 Bob，其中 Mallory 假装是
Alice。

.. figure:: ./Illustrations/DiffieHellman/MITM.svg
   :align: center

这里有两个共享秘密：一个在 Alice 和 Mallory 之间，另一个在 Mallory
和 Bob 之间。然后攻击者（Mallory）可以简单地把她从一个人那里收到的
所有消息发给另一个人，她可以查看明文消息，删除消息，也可以以任何她
选择的方式修改它们。

更糟糕的是，即使两个参与者之一以某种方式意识到正在发生什么，他们也
没有办法让另一方相信他们。毕竟：Mallory 与无知的受害者执行了成功的
Diffie-Hellman 交换，她有所有正确的共享秘密。Bob 与 Alice 没有共享
秘密，只有与 Mallory；他无法证明他是合法参与者。对 Alice 来说，
Bob 只是选择了一些随机数字。没有任何方法可以将 Bob 持有的任何密钥
与 Alice 持有的任何密钥联系起来。

这样的攻击称为 MITM 攻击，因为攻击者（Mallory）位于两个对等方
（Alice 和 Bob）之间。鉴于我们通常用于发送消息的网络基础设施由许多
不同的运营商运行，这种攻击场景非常现实，安全的密码系统必须 somehow
解决它们。

虽然 Diffie-Hellman 协议成功在两个对等方之间产生了共享秘密，但显然
仍然有一些拼图缺失才能构建安全的密码系统。我们需要工具来帮助我们对
Bob 验证 Alice 反之亦然，我们需要工具来帮助保证消息完整性，允许
接收者验证收到的消息实际上是发送者打算发送的消息。
