#############
加密类
#############

加密类提供了双向数据加密的方式，它依赖于 PHP 的 Mcrypt 扩展，所以要有 
Mcrypt 扩展才能运行。

.. important:: 这个类库已经废弃，保留只是为了向前兼容。请使用新的 
	:doc:`加密类 <encryption>` 。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

*************************
使用加密类
*************************

设置你的密钥
================

**密钥** 是对字符串进行加密或解密的一段信息片段。实际上，你设置的密钥
是 **唯一** 能解密通过该密钥加密的数据，所以一定要慎重选择你的密钥，
而且如果你打算对持久数据进行加密的话，你最好不要修改密钥。

不用说，你应该小心保管好你的密钥，如果有人得到了你的密钥，那么数据就能
很容易的被解密。如果你的服务器不在你的控制之下，想保证你的密钥绝对安全
是不可能的，所以在在你使用密钥对敏感数据（例如信用卡号码）进行加密之前，
请再三斟酌。

为了最大程度的利用加密算法，你的密钥应该使用32个字符的长度（256位），为了
保证安全性，密钥字符串应该越随机越好，包含数字、大写和小写字符， 
**不应该** 直接使用一个简单的字符串。

你的密钥可以保存在 **application/config/config.php** 文件中，或者使用你
自己设计的保存机制也行，然后在加解密时动态的取出密钥。

如果要通过文件 **application/config/config.php** 来保存你的密钥，那么打开
该文件然后设置::

	$config['encryption_key'] = "YOUR KEY";

消息长度
==============

有一点很重要，你应该知道，通过加密方法生成的消息长度大概会比原始的消息长
2.6 倍。举个例子来说，如果你要加密的字符串是 "my super secret data" ，它
的长度为 21 字符，那么加密之后的字符串的长度大约为 55 字符（这里之所以说
“大约” 是因为加密字符串以 64 位为单位非线性增长）。当你选择数据保存机制时
请记住这一点，例如 Cookie 只能存储 4k 的信息。

初始化类
======================

正如 CodeIgniter 中的其他类一样，在你的控制器中使用 ``$this->load->library()``
方法来初始化加密类::

	$this->load->library('encrypt');

初始化之后，加密类的对象就可以这样访问::

	$this->encrypt

***************
类参考
***************

.. php:class:: CI_Encrypt

	.. php:method:: encode($string[, $key = ''])

		:param	string	$string: Data to encrypt
		:param	string	$key: Encryption key
		:returns:	Encrypted string
		:rtype:	string

		执行数据加密，并返回加密后的字符串。例如::

			$msg = 'My secret message';

			$encrypted_string = $this->encrypt->encode($msg);

		如果你不想使用配置文件中的密钥，你也可以将你的密钥通过第二个可选参数传入::

			$msg = 'My secret message';
			$key = 'super-secret-key';

			$encrypted_string = $this->encrypt->encode($msg, $key);

	.. php:method:: decode($string[, $key = ''])

		:param	string	$string: String to decrypt
		:param	string	$key: Encryption key
		:returns:	Plain-text string
		:rtype:	string

		解密一个已加密的字符串。例如::

			$encrypted_string = 'APANtByIGI1BpVXZTJgcsAG8GZl8pdwwa84';

			$plaintext_string = $this->encrypt->decode($encrypted_string);

		如果你不想使用配置文件中的密钥，你也可以将你的密钥通过第二个可选参数传入::

			$msg = 'My secret message';
			$key = 'super-secret-key';

			$encrypted_string = $this->encrypt->decode($msg, $key);

	.. php:method:: set_cipher($cipher)

		:param	int	$cipher: Valid PHP MCrypt cypher constant
		:returns:	CI_Encrypt instance (method chaining)
		:rtype:	CI_Encrypt

		设置一个 Mcrypt 加密算法，默认情况下，使用的是 ``MCRYPT_RIJNDAEL_256`` ，例如::

			$this->encrypt->set_cipher(MCRYPT_BLOWFISH);

		访问 php.net 获取一份 `可用的加密算法清单 <http://php.net/mcrypt>`_ 。

		如果你想测试下你的服务器是否支持 MCrypt ，你可以::

			echo extension_loaded('mcrypt') ? 'Yup' : 'Nope';

	.. php:method:: set_mode($mode)

		:param	int	$mode: Valid PHP MCrypt mode constant
		:returns:	CI_Encrypt instance (method chaining)
		:rtype:	CI_Encrypt

		设置一个 Mcrypt 加密模式，默认情况下，使用的是 **MCRYPT_MODE_CBC** ，例如::

			$this->encrypt->set_mode(MCRYPT_MODE_CFB);

		访问 php.net 获取一份 `可用的加密模式清单 <http://php.net/mcrypt>`_ 。

	.. php:method:: encode_from_legacy($string[, $legacy_mode = MCRYPT_MODE_ECB[, $key = '']])

		:param	string	$string: String to encrypt
		:param	int	$legacy_mode: Valid PHP MCrypt cipher constant
		:param	string	$key: Encryption key
		:returns:	Newly encrypted string
		:rtype:	string

		允许你重新加密在 CodeIgniter 1.x 下加密的数据，这样可以和 CodeIgniter 2.x 的
		加密类库保持兼容。只有当你的加密数据是永久的保存在诸如文件或数据库中时，并且
		你的服务器支持 Mcrypt ，你才可能需要使用这个方法。如果你只是在诸如会话数据
		或其他临时性的数据中使用加密的话，那么你根本用不到它。尽管如此，使用 2.x 版本
		之前的加密库加密的会话数据由于不能被解密，会话会被销毁。

		.. important::
			**为什么只是提供了一个重新加密的方法，而不是继续支持原有的加密方法呢？**
			这是因为 CodeIgniter 2.x 中的加密库不仅在性能和安全性上有所提高，而且我们
			并不提倡继续使用老版本的加密方法。当然如果你愿意的话，你完全可以扩展加密库，
			使用老的加密方法来替代新的加密方法，无缝的兼容 CodeIgniter 1.x 加密数据。
			但是作为一个开发者，作出这样的决定还是应该小心谨慎。

		::

			$new_data = $this->encrypt->encode_from_legacy($old_encrypted_string);

		======================    ===============    =======================================================================
		参数                      默认值             描述
		======================    ===============    =======================================================================
		**$orig_data**            n/a                使用 CodeIgniter 1.x 加密过的原始数据
		**$legacy_mode**          MCRYPT_MODE_ECB    生成原始数据时使用的 Mcrypt 加密模式，CodeIgniter 1.x 默认使用的是 MCRYPT_MODE_ECB ，
		                                             如果不指定该参数的话，将默认使用该方式。
		**$key**                  n/a                加密密钥，这个值通常在上面所说的配置文件里。
		======================    ===============    =======================================================================
