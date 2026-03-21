术语表
========

.. glossary::
   :sorted:

   AEAD
      带关联数据的认证加密 (Authenticated Encryption with Associated Data)

   AES
      高级加密标准 (Advanced Encryption Standard)

   AKE
      认证密钥交换 (authenticated key exchange)

   ARX
      加、旋转、异或 (add, rotate, XOR)

   BEAST
      针对 SSL/TLS 的浏览器漏洞 (Browser Exploit Against SSL/TLS)

   CBC
      密码分组链接 (cipher block chaining)

   CDN
      内容分发网络 (content distribution network)

   CSPRNG
      密码学安全伪随机数生成器 (cryptographically secure pseudorandom number generator)

   CSRF
      :term:`跨站请求伪造 <cross-site request forgery>`

   DES
      数据加密标准 (Data Encryption Standard)

   FIPS
      联邦信息处理标准 (Federal Information Processing Standards)

   GCM
      Galois 计数器模式 (Galois Counter Mode)

   HKDF
      基于 HMAC 的密钥派生函数 (HMAC-based (Extract-and-Expand) Key Derivation Function)

   HMAC
      基于哈希的消息认证码 (Hash-based Message Authentication Code)

   HSTS
      HTTP 严格传输安全 (HTTP Strict Transport Security)

   IV
      :term:`初始化向量 <initialization vector>`

   KDF
      密钥派生函数 (key derivation function)

   MAC
      消息认证码 (message authentication code)

   MITM
      中间人攻击 (man-in-the-middle)

   OCB
      偏移码本 (offset codebook)

   OTR
      离 record (off-the-record)

   PRF
      伪随机函数 (pseudorandom function)

   PRNG
      伪随机数生成器 (pseudorandom number generator)

   PRP
      伪随机置换 (pseudorandom permutation)

   RSA
      Rivest Shamir Adleman (RSA 算法命名源自三位发明者)

   SMP
      社会百万富翁协议 (socialist millionaire protocol)

   secret-key encryption
      使用相同密钥进行加密和解密的加密。也称为对称密钥加密。
      与 :term:`public-key encryption` 对比

   symmetric-key encryption
      参见 :term:`secret-key encryption`

   keyspace
      所有可能密钥的集合

   block cipher
      对称加密算法，加密和解密固定大小的块

   substitution-permutation network
      块密码的通用设计，其中块通过重复替换和置换进行加密

   stream cipher
      对称加密算法，加密任意大小的流

   mode of operation
   modes of operation
      通用构造，用于加密和解密流，由分组密码构建

   ECB mode
      电子密码本模式；操作模式，其中明文被分成块，在相同密钥下分别加密。
      尽管有许多安全问题，它是许多加密库的默认模式

   CBC mode
      密码分组链接模式；常见的操作模式，其中前一个密文块在加密过程中
      与明文块进行异或。采用初始化向量，它扮演"第一个块之前的块"的角色

   initialization vector
      用于初始化某些算法（如 :term:`CBC mode`）的数据。通常不需要保密，
      但需要不可预测。与 :term:`nonce`、:term:`salt` 比较

   CTR mode
      计数器模式；:term:`nonce` 与计数器组合产生分组密码的输入序列；
      结果密文块是密钥流

   nonce
      **N**umber used **once**。用于许多加密协议。通常不需要保密或
      不可预测，但必须唯一。与 :term:`initialization vector`、:term:`salt` 比较

   AEAD mode
      :term:`block cipher` :term:`mode of operation` 的一类，提供认证加密，
      以及对未加密关联数据的认证

   OCB mode
      偏移码本模式；高性能 :term:`AEAD mode`，不幸受到专利限制

   GCM mode
      Galois 计数器模式；:term:`AEAD mode` 结合 :term:`CTR mode` 与
      :term:`Carter-Wegman MAC`

   message authentication code
      用于验证消息真实性和完整性的小段信息。常称为标签

   one-time MAC
      只能安全使用一次的消息的 :term:`message authentication code`。
      主要优点是可重用 :term:`MAC` 上提高性能

   Carter-Wegman MAC
      由 :term:`one-time MAC` 构建的可重用 :term:`message authentication code` 方案。
      结合了性能和易用性的优点

   GMAC
      :term:`GCM mode` 的一部分，可单独使用的 :term:`message authentication code`

   salt
      添加到加密基元（通常是一个单向函数，如加密哈希函数或密钥派生函数）的随机数据。
      定制这些函数以产生不同的输出（如果盐不同）。可用于防止字典攻击等。
      通常不需要保密，但保密可能提高系统安全属性。与 :term:`nonce`、:term:`initialization vector` 比较

   public-key algorithm
      使用一对相关但不同的密钥的算法。也称为 :term:`asymmetric-key algorithm`。
      示例包括 :term:`public-key encryption` 和大多数 :term:`key exchange` 协议

   asymmetric-key algorithm
      参见 :term:`public-key algorithm`

   public-key encryption
      使用一对用于加密和解密的 Distinct 密钥的加密。也称为非对称密钥加密。
      与 :term:`secret-key encryption` 对比

   asymmetric-key encryption
      参见 :term:`public-key encryption`

   key exchange
      使用特定加密协议通过不安全介质交换密钥的过程。通常设计为
      抵抗窃听者。也称为密钥协商

   key agreement
      参见 :term:`key exchange`

   oracle
      为你执行某些计算的"黑盒"

   encryption oracle
      将加密某些数据的 :term:`oracle`

   OTR messaging
      离 record 消息传递，旨在模拟真实私人会话属性的消息协议。
      依附于现有的即时消息协议

   cross-site request forgery
      恶意网站欺骗浏览器向另一个网站发出请求的攻击类型。
      可以通过正确验证请求而不是依赖会话 cookie 等环境权限来防止


.. raw:: latex

   \renewcommand{\indexname}{Index}
   \printindex
