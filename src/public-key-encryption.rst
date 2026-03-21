Public-key encryption
---------------------

.. _description-4:

Description
~~~~~~~~~~~

So far, we have only done :term:`secret-key encryption`. Suppose that you could
have a cryptosystem that didn't involve a single secret key, but instead
had a key pair: one public key, which you freely distribute, and a
private one, which you keep to yourself.

People can encrypt information intended for you by using your public
key. The information is then impossible to decipher without your private
key. This is called :term:`public-key encryption`.

For a long time, people thought this was impossible. However, starting
in the 1970s, such algorithms started appearing. The first publicly
available encryption scheme was produced by three cryptographers from
MIT: Ron Rivest, Adi Shamir and Leonard Adleman. The algorithm they
published is still the most common one today, and carries the first
letters of their last names: RSA.

:term:`public-key algorithm`\s aren't limited to encryption. In fact, you've
already seen a :term:`public-key algorithm` in this book that isn't directly
used for encryption. There are actually three related classes of
:term:`public-key algorithm`\s:

#. :term:`Key exchange <key exchange>` algorithms, such as Diffie-Hellman, which allow you to
   agree on a shared secret across an insecure medium.
#. Encryption algorithms, such as the ones we'll discuss in this
   chapter, which allow people to encrypt without having to agree on a
   shared secret.
#. Signature algorithms, which we'll discuss in a later chapter, which
   allow you to sign any piece of information using your private key in
   a way that allows anyone else to easily verify it using your public
   key.

Why not use public-key encryption for everything?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At face value, it seems that :term:`public-key encryption` algorithms obsolete
all our previous :term:`secret-key encryption` algorithms. We could just use
public key encryption for everything, avoiding all the added complexity
of having to do :term:`key agreement` for our symmetric algorithms. However,
when we look at practical cryptosystems, we see that they're almost
always *hybrid* cryptosystems: while :term:`public-key algorithm`\s play a very
important role, the bulk of the encryption and authentication work is
done by secret-key algorithms.

By far the most important reason for this is performance. Compared to
our speedy :term:`stream cipher`\s (native or otherwise), :term:`public-key encryption`
mechanisms are extremely slow. For example, with a 2048-bit (256 bytes)
RSA key, encryption takes 0.29 megacycles, and decryption takes a whopping
11.12 megacycles. :cite:`cryptopp:bench` To put this into perspective,
symmetric key algorithms work within an order of magnitude of 10 or so
cycles per byte in either direction. This means it will take a symmetric
key algorithm approximately 3 kilocycles in order to decrypt 256 bytes,
which is about 4000 times faster than the asymmetric version. The state
of the art in secure symmetric ciphers is even faster: AES-GCM with
hardware acceleration or Salsa20/ChaCha20 only need about 2 to 4 cycles
per byte, further widening the performance gap.

There are a few other problems with most practical cryptosystems. For
example, RSA can't encrypt anything larger than its modulus, which
generally doesn't exceed 4096 bits, far smaller than the largest
messages we'd like to send. Still, the most important reason is the
speed argument given above.

RSA
~~~

正如我们前面提到的，RSA 是最早的实用 :term:`public-key encryption` 方案之一。
直到今天，它仍然是最常见的方案。

加密和解密
^^^^^^^^^^^^

RSA 加密和解密依赖于模算术。在继续之前，您可能想要复习一下
:ref:`模算术入门 <modular-arithmetic>`。

本节描述 RSA 背后的简化数学问题，通常称为"教科书 RSA"。单独来看，
这不会产生安全的加密方案。我们将在后面一节看到建立在它之上的安全构造
称为 OAEP。

为了生成密钥，您选择两个大素数 :math:`p` 和 :math:`q`。这些数字必须
随机选择，并且保密。您将它们相乘产生模数 :math:`N`，这是公开的。
然后，您选择一个*加密指数* :math:`e`，这也是公开的。通常，这个值是
3 或 65537。因为这些数字的二进制展开中只有很少的"1"，您可以更高效地
计算幂运算。组合起来，:math:`(N, e)` 是公钥。任何人都可以使用公钥
将消息 :math:`M` 加密成密文 :math:`C`：

.. math::

   C \equiv M^e \pmod{N}

下一个问题是解密。事实证明，存在一个值 :math:`d`，*解密指数*，它
可以将 :math:`C` 还原为 :math:`M`。假设您知道 :math:`p` 和 :math:`q`，
这个值相当容易计算（我们知道）。使用 :math:`d`，您可以如下解密消息：

.. math::

   M \equiv C^d \pmod{N}

RSA 的安全性依赖于不知道秘密指数 :math:`d` 就无法执行解密操作，
并且从公钥 :math:`(N, e)` 计算秘密指数 :math:`d` 非常困难（实际上
不可能）。我们将在下一节看到攻击 RSA 的方法。

Breaking RSA
^^^^^^^^^^^^

与许多密码系统一样，RSA 依赖于特定数学问题的假定难度。对于 RSA，
这是 RSA 问题，具体是：给定方程中的密文 :math:`C` 和公钥 :math:`(N, e)` 时
找到明文消息 :math:`M`：

.. math::

   C \equiv M^e \pmod{N}

我们知道的最简单方法是将 :math:`N` 分解成 :math:`p \cdot q`。给定
:math:`p` 和 :math:`q`，攻击者只需重复密钥所有者在密钥生成期间所做的
过程以计算私有指数 :math:`d`。

幸运的是，我们没有算法能在合理时间内分解如此大的数字。不幸的是，
我们也没有证明它不存在。更不幸的是，存在一个称为 Shor 算法的理论
算法，它*将能够*在量子计算机上以合理时间分解这样的数字。现在，
量子计算机远未达到实用，但确实看起来如果未来有人设法建造足够大的
量子计算机，RSA 将变得无效。

在本节中，我们只考虑了通过分解模数攻击纯抽象数学 RSA 问题的私有
密钥恢复攻击。在下一节，我们将看到依赖*实现*中缺陷的 RSA 各种现实
攻击，而不是上面陈述的数学问题。

Implementation pitfalls
^^^^^^^^^^^^^^^^^^^^^^^

现在，没有已知的针对 RSA 的实际完整破坏。这并不意味着使用 RSA 的
系统不被 routinely 破坏。就像大多数被破坏的密码系统一样，有很多
声音组件不适当应用导致无用系统的情况。有关 RSA 实现可能出错的事情
的更完整概述，请参考 :cite:`boneh:twentyyears`
和 :cite:`anderson:mindingyourpsandqs`。在这本书中，我们只高亮
几个有趣的例子。

PKCSv1.5 填充
''''''''''''''''

Salt
'''''

Salt [#]_ 是一个用 Python 编写的配置系统。它有一个主要缺陷：它有一个
名为 ``crypt`` 的模块。它不是重用现有的完整密码系统，而是实现自己的，
使用第三方包提供的 RSA 和 AES。

.. [#]
   所以，有 Salt 配置系统，:term:`salt`\s 的在使用破碎的密码存储中使用的
   东西，NaCl 发音"salt"的密码学库，以及在浏览器中运行原生代码的
   NaCl，可能还有一堆我忘记的。我们可以停止以它命名事物吗？

长期以来，Salt 使用加密指数 (:math:`e`) 为 1，这意味着加密阶段
实际上什么也没做：:math:`P^e \equiv P^1 \equiv P \pmod N`。这意味着
结果密文实际上只是明文。虽然这个问题现在已经修复，但这只是表明
您可能不应该实现自己的密码学。Salt 目前也支持 SSH 作为传输，但
上述 DIY RSA/AES 系统仍然存在，并且在撰写时仍然是推荐和默认传输。
recommended and the default transport.

OAEP
^^^^

OAEP，代表最优非对称加密填充，是 RSA 填充的最新技术。它由 Mihir
Bellare 和 Phillip Rogaway 于 1995 年引入。它的结构如下所示：

.. figure:: Illustrations/OAEP/Diagram.svg
   :align: center

最终被加密的东西是 :math:`X \| Y`，它是 :math:`n` 位长，其中 :math:`n`
是 RSA 模数 :math:`N` 的位数。它接收一个 :math:`k` 位长的随机块
:math:`R`，其中 :math:`k` 是标准指定的常数。消息首先用零填充以达到
:math:`n - k` 位长。如果你看上面的"梯子"，左半部分的一切都是
:math:`n - k` 位长，右半部分的一切都是 :math:`k` 位长。随机块 :math:`R`
和零填充消息 :math:`M \| 000\ldots` 使用两个"陷门"函数 :math:`G` 和
:math:`H` 组合。陷门函数是一个容易在一个方向计算但很难反转的函数。
实际上，这些是密码学哈希函数；我们稍后会在书中看到更多关于它们的内容。

从图中可以看出，:math:`G` 接收 :math:`k` 位并将其转换为 :math:`n - k`
位，而 :math:`H` 相反，接收 :math:`n - k` 位并将其转换为 :math:`k` 位。

产生的块 :math:`X` 和 :math:`Y` 被连接，结果使用标准 RSA 加密原语
加密，产生密文。

为了了解解密如何工作，我们反转所有步骤。接收者在解密消息时得到
:math:`X \| Y`。因为他们知道 :math:`k`，这是协议的固定参数，所以他们
可以将 :math:`X \| Y` 分成 :math:`X`（第一个 :math:`n - k` 位）和
:math:`Y`（最后的 :math:`k` 位）。

在前面的图中，方向是针对应用填充的。反转梯子侧的箭头，您可以看到
如何恢复填充：

TODO: 反转箭头

我们想要到达 :math:`M`，它在 :math:`M \| 000\ldots` 中。只有一种方法
计算它，即：

.. math::

   M \| 000\ldots = X \xor G(R)

计算 :math:`G(R)` 有点难：

.. math::

   G(R) = G(H(X) \xor Y)

如您所见，至少对于函数 :math:`H` 和 :math:`G` 的某些定义，我们需要
:math:`X` 的全部和 :math:`Y` 的全部（因此是整个加密消息）才能了解
关于 :math:`M` 的任何信息。有许多函数可以是 :math:`H` 和 :math:`G`
的良好选择；基于密码学哈希函数，我们将在书中后面更详细地讨论它们。

椭圆曲线密码学
~~~~~~~~~~~~~~~~~~~~~~~~~~~

TODO: This

剩余问题：未经验证的加密
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

大多数 :term:`public-key encryption` 方案一次只能加密小数据块，
比我们想要发送的消息小得多。它们通常也非常慢，比对称对应物慢得多。
因此公钥密码系统几乎总是与秘密密钥密码系统结合使用。
They are also generally quite slow, much slower than their symmetric
counterparts. Therefore public-key cryptosystems are almost always used
in conjunction with secret-key cryptosystems.

当我们讨论 :term:`stream cipher`\s 时，我们仍然面临的剩余问题之一
是我们仍然必须与大量人交换秘密密钥。使用公钥密码系统，如公共加密
和 :term:`key exchange` 协议，我们现在已经看到了两种解决该问题的方法。
这意味着我们现在可以仅使用公开信息与任何人通信，完全免受窃听者
攻击。

到目前为止，我们只讨论了没有任何形式身份验证的加密。这意味着虽然
我们可以加密和解密消息，我们无法验证消息是发送者实际发送的内容。

虽然未经验证的加密可能提供保密性，我们已经知道没有身份验证，
主动攻击者通常可以成功修改有效的加密消息，尽管他们不一定知道相应的
明文。接受这些消息通常会导致机密信息泄露，意味着我们甚至得不到
保密性。我们已经讨论的 CBC 填充攻击说明了这一点。

因此， evidently 我们需要身份验证以及加密我们的秘密通信的方法。这是
通过向消息添加只有发送者才能计算的额外信息来完成的。就像加密一样，
身份验证有两种形式：私钥（对称）和公钥（非对称）。对称身份验证方案
通常称为 :term:`message authentication code`\s，而公钥等价物通常称为签名。

首先，我们将介绍一个新的密码学原语：哈希函数。这些可用于产生签名
方案以及消息身份验证方案。不幸的是，它们也经常被滥用来产生完全不安全的系统。
