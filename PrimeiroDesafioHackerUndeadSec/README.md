# Write-up - Primeiro Desafio Hacker Undead

Um pequeno guia de como realizar cada uma das etapas deste **[desafio](https://www.youtube.com/watch?v=YpFD9Fxm6YU)**.

# I - Obter o arquivo ZIP
O primeiro passo é acessar a URL onde se encontram as **[regras do grupo no Telegram](https://undeadsec.github.io/)**, ao inspecionar o source-code podemos observar o seguinte trecho no início do código HTML:
```
$  curl https://undeadsec.github.io/ -sS | head -6
<!-- 
[ PRIMEIRO DESAFIO HACKER UNDEAD ]
A SUA JORNADA COMEÇA AQUI:
https://github.com/WarTzu
Boa Sorte :)
-->
```
Ao acessar esta **[URL](https://github.com/WarTzu)**, podemos verificar que existe um repositório **[.-](https://github.com/WarTzu/.-)**, dentro deste repositório existe um arquivo ZIP chamado **SunS3cur3.zip**. Vamos então clonar este repositório:
``` 
$ git clone https://github.com/WarTzu/.-.git 
```
Após clonar o repositório, ao tentarmos extrair o ZIP, uma senha será requisitada:
```
$ unzip SunS3cur3.zip
Archive:  SunS3cur3.zip
[SunS3cur3.zip] SunTzu.txt password: 
```
Esta senha se encontra no vídeo, exatamente em 1:01, podemos observar que em meio ao código existe uma variável **p4ssw0rd**:

![alt text][password]

[password]: https://i.imgur.com/xUKznQE.png "Password"
```
p4ssw0rd = "Z0mB!3C0Wb0y&&*@"
```
Esta senha é a que você deve usar para extrair o arquivo ZIP.

# II - Decode da mensagem em código morse
O arquivo ZIP possui a seguinte estrutura:
```
SunS3cur3.zip
└ SunTzu.txt
```
O arquivo SunTzu.txt possui um link para um vídeo do Youtube, que está separado por quebras de linha, para obter o [link](https://youtu.be/XFnPOTnwSDc) podemos usar o comando:
```
$ tr -d '\n'< SunTzu.txt && echo $'\n'
https://youtu.be/XFnPOTnwSDc

```
O vídeo contém alguma mensagem em chinês, mas isso não tem nenhuma relação com o desafio, o importante está no som do vídeo, há alguma mensagem em código morse. 
Para decifrá-la você pode fazer isso manualmente, ou, usar alguma ferramenta disponível na internet para obter o texto da mensagem. O primeiro passo é converter o vídeo para mp3, geralmente uso o [Online Video Converter](https://www.onlinevideoconverter.com/mp3-converter), pois não depende de instalação.
Com o arquivo mp3 em mãos, é possível usar outra ferramenta para obter o texto, como o [Morse Code Adaptive Audio Decoder](https://morsecode.scphillips.com/labs/audio-decoder-adaptive/), ao reproduzir o mp3, obtemos a mensagem:

![alt text][morsedecode]

[morsedecode]: https://i.imgur.com/lyruPR7.png "Morse decode"

A mensagem então é um [link](http://suntzuwar.github.io) para o próximo passo:
```
SUNTZUWAR.GITHUB.IO
```

# III - Envio do e-mail

Ao acessarmos esta URL, podemos visualizar novamente uma mensagem em chinês e um botão **Show Mail**:

![alt text][suntzuio]

[suntzuio]: https://i.imgur.com/OO0FHCH.png "suntzuwar.github.io"

A tradução da mensagem nos indica que devemos enviar um e-mail para alguém:

![alt text][translatemessage]

[translatemessage]: https://i.imgur.com/DxXMiOZ.png "Tradução da mensagem"

Sabendo disso, podemos então entender o motivo do botão **Show Mail** e ao clicar no botão, um e-mail surge (URL encoded):
```
b%D0%BE%D0%BEtr%D0%BE%D0%BEtw%D0%B0r@gmail.com
```
Você pode realizar o decode do endereço de e-mail ou inspecionar no source da página, aonde está o e-mail sem o encode:
```
$ curl https://suntzuwar.github.io -sS | head -10
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="chrome=1">
    <script>
	function myFunction() {
    	var uri = "bооtrооtwаr@gmail.com";
    	var res = encodeURI(uri);
    	document.getElementById("demo").innerHTML = res;}
```
Com estas informações podemos concluir que é necessário enviar um e-mail para esta conta, ao enviar o e-mail recebemos uma resposta com um [link](https://goo.gl/Ek8Ehj) para o próximo passo:

![alt text][e-mail]

[e-mail]: https://i.imgur.com/o1AUZGO.png "E-mail"

# IV - Análise do arquivo PCAP

A URL nos redireciona para um arquivo PCAP hosteado no Google Drive, este arquivo PCAP deve ser analisado para podermos passar ao próximo passo. Para analisar este arquivo PCAP vamos usar o [Wireshark](https://www.wireshark.org/download.html).
O primeiro filtro que apliquei foi **http** para filtrar somente requisições HTTP e em seguida criei a coluna Custom chamada **Content-Type** com o content-type da requisição **"http.content_type"**, a partir deste filtro, você pode verificar pelos content-types e procurar por arquivos específicos. 

![alt text][custom-column]

[custom-column]: https://i.imgur.com/ANGYuiY.png "Custom column"

![alt text][http-filter]

[http-filter]: https://i.imgur.com/TiL5gMD.png "HTTP filter"

Em meio as requisições, existe uma solicitando um arquivo Python, seguindo a stream TCP você consegue encontrar o pacote aonde está o conteúdo do arquivo:

![alt text][python-package]

[python-package]: https://i.imgur.com/so8V6Gz.png "Python content package"
O conteúdo do arquivo Python:
```
# -*- coding: iso-8859-15 -*-
from os import system
from base64 import b64decode

def main():
    system('clear')
    senha = 'SW5zaXJhIGEgc2VuaGE6IA=='
    entrada = raw_input(b64decode(senha))
    if entrada == b64decode(senha).lower()[::-1].upper():
        print '\n[*] Bem vindo.\n'
        raw_input('Parabéns Você conseguiu! Envie a senha para @A1S0N no telegram :D\n ---> https://t.me/A1S0N')
    else:
        print '\n[!] Senha inválida.\n'   
        raw_input('Pressione ENTER para continuar...')

while True:
    main()
```
Salvei este arquivo Python como **pcap.py**, ao executá-lo foi requisitada uma senha:
```
$ python2 pcap.py
Insira a senha: teste
[!] Senha inválida.
```
Para decifrarmos a senha basta executar a linha que checa a senha:
```
$ python2
>>> from base64 import b64decode
>>> senha = 'SW5zaXJhIGEgc2VuaGE6IA=='
>>> b64decode(senha).lower()[::-1].upper()
b' :AHNES A ARISNI'
```
Executando o código novamente e fornecendo a senha correta **" :AHNES A ARISNI"**, obtemos a mensagem:
```
$ python2 pcap.py
Insira a senha: :AHNES A ARISNI

[*] Bem vindo.

Parabéns Você conseguiu! Envie a senha para @A1S0N no telegram :D
 ---> https://t.me/A1S0N
```

Ao enviar a senha para o [@A1S0N](https://t.me/A1S0N), você recebe o [link](https://goo.gl/dBHBJm) para o próximo passo.

![alt text][msg-telegram]

[msg-telegram]: https://i.imgur.com/X7rVYyb.png "Mensagem enviada ao A1S0N"

# V - A Cifra de César

A URL enviado pelo [@A1S0N](https://t.me/A1S0N) nos redireciona para um arquivo TAR.XZ hosteado no Google Drive. O primeiro passo é extrair os arquivos:
```
$  tar -xJf ERROR.tar.xz
```
E podemos ver então que o arquivo TAR.XZ possui a seguinte estrutura:
```
ERROR.tar.xz
└ cipher.jpg
└ senha.txt
└ smith.zip
```
O primeiro passo foi procurar por algo no conteúdo (esteganografia) ou nos metadados da imagem **cipher.jpg**, mas nada foi encontrado. Então pesquisei pela imagem no Google Images para saber de quem era esta escultura.

![alt text][img-search]

[img-search]: https://i.imgur.com/3aF0O7T.png "Busca pela imagem no Google Images"

Sabendo que se tratava de Julius Caesar e analisando o conteúdo do arquivo **senha.txt**:
```
$ cat senha.txt
coxrk: BooFocvGkVvc
SHIFT: 10
```
Temos uma string **SHIFT**, correlacionando ambos, chegamos então na Cifra de César. Para obter a mensagem em texto claro, basta usar uma ferramenta online como a [Dcode - Caesar Cipher](https://www.dcode.fr/caesar-cipher) e preencher os campos para obter a mensagem em texto claro:

![alt text][decode-caesar]

[decode-caesar]: https://i.imgur.com/dmW5ZQd.png "Decode da mensagem"

```
senha: reeveslwalls
```
Esta senha deve ser usada para extrair o segundo arquivo ZIP **smith.zip**, que é protegido por senha.
```
$ unzip smith.zip
Archive:  smith.zip
[smith.zip] smith.jpg password:
```
Este arquivo ZIP possui a seguinte estrutura:
```
smith.zip
└ smith.jpg
```

# VI - Análise da imagem

O primeiro passo foi novamente procurar por algo no conteúdo (esteganografia) ou nos metadados da imagem, que neste caso, possuía informações importantes:
```
$ exif smith.jpg
EXIF tags in 'smith.jpg' ('Motorola' byte order):
--------------------+----------------------------------------------------------
Tag                 |Value
--------------------+----------------------------------------------------------
Image Description   |aHR0cHM6Ly9nb28uZ2wvR3VLSnNu
Resolution Unit     |Internal error (unknown value 1)
Artist              |VW5kZWFk
YCbCr Positioning   |Centered
Copyright           |VW5kZWFkU2Vj (Photographer) - [None] (Editor)
X-Resolution        |72
Y-Resolution        |72
Exif Version        |Unknown Exif Version
Components Configura|Y Cb Cr -
FlashPixVersion     |FlashPix Version 1.0
Color Space         |Internal error (unknown value 65535)
--------------------+----------------------------------------------------------
```
Uma string encodada em base64 no campo Image Description **aHR0cHM6Ly9nb28uZ2wvR3VLSnNu**, basta realizar o decode para obter o texto claro:
```
$ openssl enc -base64 -d <<< aHR0cHM6Ly9nb28uZ2wvR3VLSnNu && echo $'\n'
https://goo.gl/GuKJsn

```
Este [link](https://goo.gl/GuKJsn) nos redireciona para um arquivo EXE hospedado no Google Drive. 

# VII - Análise do executável

O primeiro passo ao baixar o arquivo foi procurar por strings que pudessem trazer alguma informação, como URLs:
```
$ strings Trinity.exe | grep 'http'
5.0.0 (http://llvm.org/git/clang.git dba970f4d143480b964f77b363ec23f22cea0390) (http://llvm.org/git/llvm.git 52ebe03cb0a728134e66d04f85281bc5a60d7091)
https://www.chiark.greenend.org.uk/~sgtatham/putty/
         xmlns="http://schemas.microsoft.com/SMI/2005/WindowsSettings">
3A_MATRIX_ESTA_EM_VOCE_https://goo.gl/tm7ojX
(...)
```
Dentro das strings podemos ver que existe um outro [link](https://goo.gl/tm7ojX) para um arquivo WAV também hospedado no Google Drive. 

# VIII - Análise do arquivo WAV

O primeiro passo foi novamente procurar por alguma mensagem em código Morse, com o [Morse Code Adaptive Audio Decoder](https://morsecode.scphillips.com/labs/audio-decoder-adaptive/). Ao reproduzir o arquivo WAV, obtemos a seguinte mensagem:

![alt text][morsedecodetwo]

[morsedecodetwo]: https://i.imgur.com/cu4B6ll.png "Morse decode"

A mensagem então é **"O ESCOLHIDO"**, mas não era somente este o objetivo. Então o segundo passo foi a busca no conteúdo do arquivo (esteganografia), pois ao escutar um pequeno chiado no início e notar que o som produzido por este trecho não faz sentido algum quando executado, podemos suspeitar do conteúdo do arquivo. Para confirmar podemos analisar o espectrograma do arquivo por alguma ferramenta, como o [Spectrum Analyzer - Academo.org](https://academo.org/demos/spectrum-analyzer/) e assim podemos notar claramente que no início do arquivo existe um padrão diferente de frequências, indicando que realmente poderia existir algo neste trecho do arquivo.

![alt text][spectrum-analyzer]

[spectrum-analyzer]: https://i.imgur.com/anVX3yK.png "Spectrum Analyzer"

Para tentar encontrar arquivos ocultos neste arquivo WAV foi usado o programa [DeepSound](http://jpinsoft.net/deepsound/).
Ao selecionar o arquivo WAV, uma senha é solicitada para obter os arquivos secretos. Sabendo que a mensagem em código Morse era **"O ESCOLHIDO"** e que as referências eram a Matrix, o escolhido foi **Neo** (ou não, muitas teorias envolvidas nisso), portanto, a senha deveria ser **neo**. A senha foi aceita e surgiram dois arquivos disponíveis para serem extraídos **key.txt** e **encrypted.txt**

![alt text][deepsound]

[deepsound]: https://i.imgur.com/37ma6p0.png "Deepsound"

# IV - RSA decode
Após extrair os arquivos pelo programa, podemos analisar os conteúdos, que eram os seguintes:
**encrypted.txt**
```
hL+hw6kuJvyE65hyHi9CB+FUEmzd9S2VCBRWlWua04xVeBkIn6zwaaJ/aND8aeJdvO88B3uUCY5ATpvhh/2Vrez8bC/tbhRLzCFa2CFeHoJoBDCmbwHMn7B185YrQiMrt6vReGukQB3EL2s0MpAuFyRfVKrdSTqgqUtgfbSx4ZE=
```
**key.txt**
```
MIICWwIBAAKBgQDB/3t0owBQia5WPA81CHOah1Dclzoowm9FE+K6zRdGAIw9JSWM
HASNviczTkugn5d88w0kXfjWyg3wOv5E1pgiltZjl0u4sTkOVAunhqW9R+s/MfWQ
fgxQ52vgHqMNfogo2dN3WLK9Fd9T2hY1X2iqbK5pab7rJet4PLTfUTJMiQIDAQAB
AoGAIXGLtOXMzhWOKmucK4ZTd5ZQSFcBvbkXOY9eDNoCYx0BECFxQaAq4MyhMWUU
AJLCqNW1tElG9rBKitmAsBlWjIMVcl+DP5AWXLmkWYJwW789K6KarC0Z12UX9fy6
j5bX1g/PDvIJShETUR+wQ/onKypBbzK0OMwJfH+7cC+Yb6kCQQDm/igkD62VjTOD
90fRd0SdHie59IZzHa97yj0E9a3cv6FlYTiErR2aE0KfDf7T4a3B99hQXRNB5wjH
GjHftM2PAkEA1wAIlPWj+tzqW1ES1LnGH9YLdvRsV5BnWOJqwTReMf9X1X3UZbaW
I83Uhv+cOoS8jiY9+7ZTsWQWyk301zboZwJADiFUAUi4PKLDmPoCeazLFLVohraP
lvEk7/SiIPCKbyuFyvbUh0Ezw14UQDiR8xImF+x6Xggjim+AmPVgQagEvwJAQlSN
UT+TjqK3XuLdV2nVGR9VPCbegglYCREZdG/um6g2dfQzIgo5ueQXrGqRzXAEKCre
NpkiqvjBGzr/zaHwAwJAI11F+JQ//ZwTtYRANUaSYwzSGgD83S2KJ2DP7VHIDURn
kLonskUdCVeWAWgygkjXJYKQ+NFBhE87G7aNYZgCGw==
```
Parecia um arquivo criptografado acompanhado de uma chave para descriptografar, a chave era muito semelhante a uma chave RSA. O primeiro passo é editar a chave e adicionar os campos identificadores de uma chave privada RSA para que o OpenSSL a reconheça:
```
-----BEGIN RSA PRIVATE KEY-----
(...)
-----END RSA PRIVATE KEY-----
```
Foi usado então o OpenSSL para converter o conteúdo do arquivo **encrypted.txt** para binário (o arquivo está encodado em base64):
```
$ base64 -d encrypted.txt > blob
```
O próximo e último passo foi tentar descriptografar o conteúdo binário usando a chave privada **key.txt**:
```
$ openssl rsautl -decrypt -inkey key_base.txt < blob > decrypted.txt
$ cat decrypted.txt && echo $'\n'
Você venceu o desafio!
Aqui está sua chave de vitória:
h2he-12e-0210-9124-04=3240=23432edxUndead@@

```
E fim! 

**CREDITS**:
- [@A1S0N](https://t.me/A1S0N) pelo suporte e pela criação do desafio
- [@exofelipe](https://github.com/exofelipe/) pelo suporte, pela ajuda na construção e revisão do Writeup
