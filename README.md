# tutorial-r36s-clone
Tutorial instalação ArkOS no R36S Clone

Dados do hardware utilizado:
Console R36S (Clone)
Identificação da placa: G80C - MB V1.1 - 20250319

Esse tutorial é focado em usuários linux que estão com problemas para gravação da imagem "ArkOS_K36_v2.0_08112025.img.xz".
Alguns usuários está relatando que a imagem está vazia e não apresenta partições (ocorreu inclusive no meu caso).
Inicialmente eu acreditava ser um problema na minha conexão de internet que corrompia o download. Mas analizando
o arquivo constatei que o disponibilizado no repositório realmente precisa de alguns ajustes.

Por quê isso ocorre?
Ao tentar gravar a imagem no cartão sd utilizando um sistema operacional diferente do windows alguns softwares têm um
comportamento levemente diferente ( Incluindo o balena etcher).

Obtenha o arquivo de imagem no repositório do Ark com:
```bash
wget https://github.com/AeolusUX/ArkOS-K36/releases/download/ArkOS.K36.08112025/ArkOS_K36_v2.0_08112025.img.xz
```

Extraia o arquivo com:
```bash
xz -d ArkOS_K36_v2.0_08112025.img.xz
```

Ao tentar montar o arquivo você vai notar que ele está vazio (não apresenta as partições - devido ao erro na tabela GPT)

![Imagem Original Vazia](/imagens/02-visualização-arquivo-imagem-original.png) 

Se você tentar gravá-lo no estado atual não obterá êxito.

Vamos para a correção. Eu tento utilizar o máximo possível de ferramentas nativas do sistema (Debian 13 / OpenSUSE16 Leap/ RockyLinux10...)

Segue a lista de ferramentas que utilizei:
* gnome-disk-utility (opcional)
* fdisk
* gdisk
* dd
* dc3dd
* mousepad (ou qualquer editor de texto simples de sua preferência)
* wget ( ou qualquer acelerador de download de sua preferência)




  
