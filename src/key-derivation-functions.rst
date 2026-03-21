.. _key derivation function:

密钥派生函数
--------------

.. _description-8:

描述
~~~~

密钥派生函数是一个从一个秘密值派生一个或多个秘密值（*密钥*）的函数。

许多密钥派生函数还可以接受一个（通常可选的）:term:`盐值`参数。该参数
使密钥派生函数对于相同的输入秘密不总是返回相同的输出密钥。与其他
密码系统一样，:term:`盐值`从根本上不同于秘密输入：:term:`盐值`通常不必是
秘密的，可以重复使用。

密钥派生函数可能很有用，例如，当密码协议以一个单一秘密值开始时，
如共享密码或使用 Diffie-Hellman 密钥交换派生的秘密，但需要多个
秘密值才能操作，如加密和 MAC 密钥。密钥派生函数的另一个用例是
在密码学安全的随机数生成器中，我们将在下一章更详细地看到，其中
它们用于从每个熵密度低的许多源中提取高熵密度的随机性。

根据秘密值的熵含量，有两种主要类别的密钥派生函数，这决定了秘密值
可以取多少不同的可能值。

如果秘密值是用户提供的密码，例如，它通常包含非常少的熵。密码可能
取的值非常少。正如我们在:ref:`关于密码存储的上一节<a previous section on password storage>` 中已经建立的，
这意味着密钥派生函数必须难以计算。这意味着它需要非平凡数量的计算资源，
如 CPU 周期或内存。如果密钥派生函数容易计算，攻击者可以简单地枚举
共享秘密的所有可能值，因为可能性很少，然后为其所有可能值计算密钥
派生函数。正如我们在关于密码存储的那一节中看到的，这就是大多数
现代攻击密码存储的方式。使用适当的密钥派生函数将阻止这些攻击。
在本章中，我们将看到 scrypt 以及此类中的其他密钥派生函数。

另一方面，秘密值也可能具有高熵含量。例如，它可能来自 Diffie-Hellman
:term:`密钥协商` 协议的共享秘密，或由密码学随机字节组成的 API 密钥
（我们将在下一章讨论密码学安全的随机数生成）。在这种情况下，
不需要难以计算的密钥派生函数：即使密钥派生函数计算是琐碎的，秘密
可以取的可能值也太多了，所以攻击者将无法枚举它们全部。我们将看到
此类最佳 breed 的密钥派生函数 HKDF。

密码强度
~~~~~~~~

TODO: NIST Special Publication 800-63

PBKDF2
~~~~~~

bcrypt
~~~~~~

scrypt
~~~~~~

HKDF
~~~~

在 RFC 5869 中定义并在相关论文中详细解释的 HKDF 是专为高熵输入
设计的密钥派生函数，例如来自 Diffie-Hellman 密钥交换的共享秘密。
它专门*不*设计用于低熵输入（如密码）。

HKDF 的存在是为了给人们一个适当的、现成的密钥派生函数。以前，密钥
派生通常是为特定标准临时做的事情。通常这些临时解决方案没有 HKDF
所做的额外规定，如 :term:`盐值` 或可选 info 参数（我们将在本节后面讨论）；
而这只是在 KDF 从根本上就不是完全broken的最好的情况。

HKDF 基于 HMAC。像 HMAC 一样，它是一个使用哈希函数的通用构造，并且
可以使用您想要的任何密码学安全哈希函数构建。

仔细看看 HKDF
^^^^^^^^^^^^^^^^^^^^^

.. canned_admonition::
   :from_template: advanced

HKDF 由两个阶段组成。在第一阶段（称为*提取阶段*），从输入熵中提取
固定长度的密钥。在第二阶段（称为*扩展阶段*），该密钥用于产生多个
伪随机密钥。

提取阶段
''''''''''''

提取阶段负责从可能大量但熵密度较小的数据中提取少量高熵含量的数据。

提取阶段只使用带 :term:`盐值` 的 HMAC：

.. code:: python

   def extract(salt, data):
       return hmac(salt, data)

:term:`盐值` 参数是可选的。如果未指定 :term:`盐值`，则使用等于哈希函数输出长度
的零字符串。虽然 :term:`盐值` 在技术上可选，但设计者强调其重要性，
因为它使密钥派生函数的独立使用（例如在不同应用程序中或与不同用户一起）
产生独立的结果。即使相当低熵的 :term:`盐值` 已经可以对密钥派生函数的
安全性做出显著贡献。

提取阶段解释了为什么 HKDF 不适合从密码派生密钥。虽然提取阶段非常
擅长*集中*熵，但它不能*放大*熵。它设计用于将分布在大量数据中的少量
熵压缩成少量数据中的相同熵量，但不设计用于在少量可用熵面前创建
难以计算的一组密钥。也没有使该阶段计算密集化的规定。

在某些情况下，如果共享秘密已经具有所有正确属性，例如如果它是足够
长度的伪随机字符串，并且具有足够的熵，则可以跳过提取阶段。然而，
有时根本不应该这样做，例如处理 Diffie-Hellman 共享秘密时。RFC 对
是否应该跳过此步骤的话题稍微详细了一些；但通常不建议这样做。

扩展阶段
''''''''''''

在扩展阶段，从提取阶段的输入中提取的随机数据扩展成所需数量的数据。

扩展步骤也非常简单：使用 HMAC 生成数据块，这次使用提取的秘密，
而不是公共 :term:`盐值`，直到产生足够的字节。被 HMAC 的数据是前一个输出
（以空字符串开始）、"info" 参数（默认也是空字符串）和计算当前正在
生成哪个块的计数器字节。

.. code:: python

   def expand(key, info=""):
       """Expands the key, with optional info."""
       output = ""
       for byte in map(chr, range(256)):
           output = hmac(key, output + info + byte)
           yield output

   def get_output(desired_length, key, info=""):
       """Collects output from the expansion step until enough
       has been collected; then returns that output."""
       outputs, current_length = [], 0
       for output in expand(key, info):
           outputs.append(output)
           current_length += len(output)

           if current_length >= desired_length:
               break
       else:
           # This block is executed when the for loop *isn't*
           # terminated by the ``break`` statement, which
           # happens when we run out of ``expand`` outputs
           # before reaching the desired length.
           raise RuntimeError("Desired length too long")

       return "".join(outputs)[:desired_length]

像提取阶段中的 :term:`盐值` 一样，"info" 参数完全可选，但实际上可以大大
增加应用程序的安全性。"info" 参数旨在包含使用密钥派生函数的特定于
应用程序的上下文。像 :term:`盐值` 一样，它会使密钥派生函数在不同上下文中
产生不同的值，进一步提高其安全性。例如，info 参数可能包含关于所处理的
用户、执行密钥派生函数的协议部分或类似内容的信息。
