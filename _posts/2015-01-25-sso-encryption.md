---
title:  "간단한 SSO #3 - 암호화"
date:   2015-01-25 18:30:00
categories: mvc
author: Jake Ryu
permalink: /sso-encryption/
---

암호화는 그 자체로 하나의 큰 주제이다. 여러 가지의 암호화 기술이 있고 발전해 온 역사도 길어서 대학에서 한 학기 강의로 다룰 정도로 내용이 방대하다.

이 글에서는 암호화의 원천적인 기술에 대해서는 가볍게 설명하고 소스 위주로 그 사용에 대해 자세해 알아보겠다.

쿼리스트링으로 전달하는 데이터를 알아 볼 수 없도록 처리해야 하는 것이 목표이다. 상식적인 내용이지만 원문을 암호화하고 복호화를 통해 원문을 복원하려면 같은 알고리즘을 사용해야 한다. 예제에서는 같은 키를 사용해서 암호화-복호화를 수행하기 때문에 대칭(aymmetric) 알고리즘을 사용한다. 

>다른 예로, 우리가 잘 아는 SSL은 비대칭(asymmetric) 알고리즘을 사용하는데 아래와 같은 과정을 거친다.

>1. 사용자 브라우저는 https 로 시작하는 URL을 사용해서 서버에 접속한다.
2. 웹 서버는 브라우저로 공개 키(public key)와 비밀 키(private key)를 보낸다.
3. 서버와 클라이언트(브라우저)는 암호화 통신을 위한 암호화 정도를 협상한다.
4. 브라우저는 서버의 공개 키로 암호화한 세션 키를 서버에 보낸다.
5. 서버는 자신의 비밀 키(비공개 키)를 사용하여 세션 키를 알아내고 세션을 맺는다.
6. 서버와 클라이언트는 세션 키를 사용하여 데이터를 암호화하고 복호화한다.

여기서 사용하는 대칭 알고리즘은 AES(Advanced Encryption Security)이고 미국연방정부에서 현재 표준으로 권장하는 기술이다. AES는 엄격히 말해서 알고리즘이 아니라 린델(Rijndael) 알고리즘을 기반으로 한 스펙이라고 할 수 있다. AES는 린델 알고리즘이 제공하는 기능내에서 키 길이를 기준으로 AES-128 / AES-192 / AES-256 처럼 표준을 정하고 있다. 굳이 AES 와 Rijndael의 차이를 언급하는 이유는 MSDN에서 구현 예제를 찾아 볼때, 두 가지 예제를 접하게 되어 무엇을 사용해야 할지 혼선이 생길 수 있기 때문이다. 두 가지중 어떤 것을 사용하더라도 현재 시점에서 가장 안전한 방법을 택하는 것이다. 자세한 내용은 아래 링크를 참조하자.

[The Differences Between Rijndael and AES]

이제 알고리즘을 사용하여 암호화하는 과정을 살펴보자. 암호화-복호화를 수행할 클래스를 AesCryptography 라고 이름 짓고 Model 폴더에 추가한다.

{% highlight C# %}
public class AesCryptography
{
    private readonly byte[] _key = Convert.FromBase64String("NK1+Uu/zWGXgf4dEVUeZ3v4wShzgbq21");  // 192 bits
    private readonly byte[] _iv = Convert.FromBase64String("K3C6iRQA086MexIVypNHqQ==");  // 128 bits
    private Aes CreateCipher()
    {
        Aes aesAlg = Aes.Create();
        aesAlg.Key = _key;
        aesAlg.IV = _iv;
        aesAlg.Mode = CipherMode.CBC;
        aesAlg.Padding = PaddingMode.ISO10126;
        return aesAlg;
    }
}
{% endhighlight %}

<소스 1> AES 알고리즘을 사용하는 chipher 생성하기

Cipher는 하나의 알고리즘을 사용해서 암호화-복호화를 처리할 수 있는 수단을 의미하는데, 위의 코드에서 `CreateCipher` 메서드를 따로 분리한 이유는 암호화, 복호화 메서드에서 같은 cipher를 사용하기 위함이다. 일종의 cipher 팩토리로 생각할 수 있다. 이 메서드에서 설정하고 있는 값들은 block cipher를 사용할 때 가장 추천되는 설정이다 (block cipher와 함께 stream cipher도 많이 사용한다. 스트림 개념이 더해지면 더욱 복잡해질 것을 우려해서 block cipher를 선택했다).

Block cipher가 동작하는 방식을 이해하면 각각의 왜 키(key)와 IV(Initial Vector) 값이 필요한지 알 수 있다. 

![Block cipher의 CBC 모드](/assets/mvc/CipherBlockChainingModeEncryption.png)

<그림 1> Block cipher의 CBC 모드 [(출처)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation)

Block 은 한번에 암호화를 수행하는 단위이다. 한 번에 8개의 문자를 암호화 한다면 block size는 8 이다. <그림 1>에서는 한 번에 8 bytes(64 bits)의 block 을 암호화하고 그 결과(ciphertext)를 다음 번 암호화에 주입하는 과정을 보여주고 있다. 이러한 방식을 Cipher Block Chaining이라고 한다.

[여러가지 Cipher Mode, MSDN 참조](https://msdn.microsoft.com/en-us/library/system.security.cryptography.ciphermode(v=vs.110).aspx)

하나의 Plaintext를 같은 알고리즘을 통하여 암호화 하면 항상 같은 결과(Ciphertext)를 만들어 내는데 이 같은 패턴은 반복적인 유추를 통해 키를 알아낼 수 있는 위험을 내포하고 있다. 따라서, 이전 블럭의 결과를 다음 블럭 암호화에 사용한다면 Randomness가 증가하고 결과 예측은 더욱 어려워진다. 최초 블럭을 암호화할 때는 이전 ciphertext가 없기 때문에 random seed 역할을 하는 것이 initial vector의 역할이다.

마지막으로 block 단위로 암호화를 하다 보니 문자를 채우고 마지막에 남는 공간이 생길 수 있는데 이 공간을 어떻게 채울지 결정하는 것이 padding mode이다. 예제에서 사용하는 ISO10126 방식은 남은 공간을 랜덤하게 채우는 가장 복잡한 방식이다.

[여러가지 Padding Mode, MSDN 참조](https://msdn.microsoft.com/en-US/library/system.security.cryptography.paddingmode(v=vs.80).aspx)

이제 암호화-복호화를 수행하는 핵심 구현이 남았는데 닷넷 프레임워크에서 제공하는 cipher의 기능을 사용하면 된다. 

{% highlight C# %}
public byte[] EncryptStringToBytes(string plainText)
{
    // Check arguments. 
    if (plainText == null || plainText.Length == 0)
        throw new ArgumentNullException("plainText");
    byte[] toEncode = Encoding.UTF8.GetBytes(plainText);
    byte[] encrypted;
    // Create an Aes object with the specified key and IV.
    // Create a encrytor to perform the byte array transform.
    using (var aesAlg = CreateCipher())
    using (ICryptoTransform encryptor = aesAlg.CreateEncryptor())
    {
        encrypted = encryptor.TransformFinalBlock(toEncode, 0, toEncode.Length);
        aesAlg.Clear();
    }

    return encrypted;
}
{% endhighlight %}

<소스 2> 암호화

{% highlight C# %}
public string DecryptStringFromBytes(byte[] cipherText)
{
    // Check arguments. 
    if (cipherText == null || cipherText.Length == 0)
        throw new ArgumentNullException("cipherText");

    // Declare the string used to hold the decrypted text. 
    string plainText = null;
    // Create an Aes object with the specified key and IV. 
    // Create a decrytor to perform the byte array transform.
    using (Aes aesAlg = CreateCipher())
    using (ICryptoTransform decryptor = aesAlg.CreateDecryptor())
    {
        plainText = Encoding.UTF8.GetString(
            decryptor.TransformFinalBlock(cipherText, 0, cipherText.Length)
        );
        aesAlg.Clear();
    }

    return plainText;
}
{% endhighlight %}

<소스 3> 복호화

암호화에 대한 설명이 많이 부족하지만 동작을 이해하는 수준은 되었으리라 생각한다. 암호화에 관심이 많은 분은 스탠포드 대학의 [암호학 강좌] 도전해 보는 것도 좋겠다.
<br />

*연재 글*

- [간단한 SSO #1 - 접근법](/sso-approach/)
- [간단한 SSO #2 - 라우팅](/sso-routing/)
- **간단한 SSO #3 - 암호화**
- [간단한 SSO #4 - 위조방지](/sso-hmac/)

[The Differences Between Rijndael and AES]: http://blogs.msdn.com/b/shawnfa/archive/2006/10/09/the-differences-between-rijndael-and-aes.aspx
[암호학 강좌]: https://www.coursera.org/course/crypto

