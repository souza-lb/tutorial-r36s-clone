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

Comecei a inspecionar a imagem usando o fdisk para consultar o seu conteúdo:

Extraia o arquivo com:
```bash
sudo fdisk -l ArkOS_K36_v2.0_08112025.img
```

Você vai receber algo do tipo:

```bash
Dispositivo                   Início      Fim  Setores Tamanho Tipo
ArkOS_K36_v2.0_08112025.img1   32768  1056767  1024000    500M Microsoft dados básico
ArkOS_K36_v2.0_08112025.img2 1056768 17833983 16777216      8G Linux sistema de arquivos
```

Repare que há duas partições 

![Tabela comrrompida](/imagens/03-inspeção-arquivo-imagem-original.png) 

Atenção também no trecho no topo da tela.

```bash
A tabela GPT primária está corrompida...
```

Vamos corrigir isso usando o "gdisk"

```bash
sudo gdisk ArkOS_K36_v2.0_08112025.img
```

Ele vai abrir a tela abaixo:

```bash
Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help):
```

![Uso gdisk](/imagens/04-uso-gdisk.png) 


Siga os passo abaixo:

1 - Menu de recuperação ( pessione a recla "r" e depois enter)

2 - Criar nova tabela de partições (backup) (pressione a tecla "c" e depois ENTER - confirme com Y e depois ENTER)

3 - Escrever alterações ( pressione a tecla "w" e depoist ENTER - confirme com Y e depois ENTER)

Se você seguiu os comandos corretamente receberá uma tela do tipo:

![Saída gdisk](/imagens/05-confirmação-nova-tabela-partição.png) 

Agora tente visualizar (montar a imagem) estou usando o gnome-disk-utility

![Imagem após correção](/imagens/06-imagem-após-ajuste.png) 

Opa parece que a imagem agora mostra corretamente as partições.

Agora vamos proceder com a gravação da imagem no cartão microsd. Fica ao seu critério utilizar um programa gráfico ou não.
Como eu curto fazer as coisas pelo terminal pela praticidade vamos lá:

Para ler o cartão no meu computador vou usar o meu leitor de cartão. Use um cartão microsd de pelo menos 16GB para a imagem caber.

![Leitor de cartão microsd](/imagens/07-leitor-cartão-cartão.jpeg) 

Se você curte usar o dc3dd (inclusive é uma boa ferramenta para backup do disco original)

Depois de plugar o leitor de cartão com o microsd inserido no computador eu consulto qual é a identificação da unidade com o "lsblk"

```bash
lsblk
```

A saída do lsblk será algo como abaixo. No meu caso o microsd é a unidade sdc. Atenção para não efetuar a gravação no disco incorreto 
e causar uma perda de dados!

![Tela lsblk](/imagens/08-lista-unidades.png) 

Procedo com a gravação da imagem:

```bash
sudo dc3dd if=ArkOS_K36_v2.0_08112025.img of=/dev/sdc
```

Saída esperada dc3dd:

![Gravação de imagem com dc3dd](/imagens/09-saída-esperada-dc3dd.png) 


Se você preferir usar o "dd" ( apesar da menor valocidade )

```bash
sudo dd if=ArkOS_K36_v2.0_08112025.img of=/dev/sdc status=progress
```

Vamos finalizar com:

```bash
sudo sync
```

Depois remova a unidade com:

```bash
sudo eject /dev/sdc
```

Remova e insira novamente o leitor de cartão no seu computador

Observe as partições do disco recem gravado:

![Partições disco](/imagens/10-partições-disco.png) 

Abra o gerenciador de arquivos de sua preferência e explore o conteúdo:


![Conteúdo disco](/imagens/11-conteúdo-disco-gravado.png) 


Vamos para a partição "BOOT" fat16 precisamos adicionar o arquivo dtb para o nosso modelo de tela.
Observe o seu conteúdo abaixo:

![Partição boot](/imagens/12-partição-BOOT.png) 

Como eu sou precavido e fiz backup da imagem original que veio logo na chegada do console.
Segue abaixo os arquivos dtb do tipo de tela que meu console utiliza ( tipo 8)

rf3536k3ka.dtb
rk3326-evb-lp3-v12-linux.dtb

https://github.com/souza-lb/tutorial-r36s-clone/tree/main/arquivos_dtb_originais

Vamos copiar esses dois arquivos para a raiz da partição BOOT conforme abaixo

![dtb na partição boot](/imagens/13-arquivos-dtb-fábrica.png) 

Agora vamos acessar a pasta "extlinux" na mesma partição "BOOT". Precisamos referenciar o arquivo referente ao meu tipo de tela 8.

![arquivo extlinux](/imagens/15-arquivo-extlinux.png) 

Abra o arquivo com o editor de texto de sua preferência

Ele vai apresentar o conteúdo abaixo:

```bash
LABEL ArkOS
  LINUX /Image
  FDT /rf3536k4ka.dtb
  INITRD /uInitrd
  APPEND earlyprintk console=ttyFIQ0 rw root=/dev/mmcblk1p2 rootfstype=ext4 loglevel=7 init=/sbin/init rootwait rootdelay=10 fsck.repair=yes fbcon=rotate:0 quiet splash plymouth.ignore-serial-consoles consoleblank=0
```

Substitua por:

```bash
LABEL ArkOS
  LINUX /Image
  FDT /rf3536k3ka.dtb
  INITRD /uInitrd
  APPEND earlyprintk console=ttyFIQ0 rw root=/dev/mmcblk1p2 rootfstype=ext4 loglevel=7 init=/sbin/init rootwait rootdelay=10 fsck.repair=yes fbcon=rotate:0 quiet splash plymouth.ignore-serial-consoles consoleblank=0
```

Repare que a mudança foi feita na linha para referenciar o arquivo dtb inserido na partição 

```bash
  FDT /rf3536k3ka.dtb
```

Salve as mudanças feche o arquivo

Execute :

```bash
  sudo sync
```

Depois:
```bash
  sudo eject /dev/sdc
```

Em seguida insira o cartão no seu console e aguarde. No primeiro boot ele irá expandir a partição easyroms para ocupar todo o cartão. Em seguida carregará a insterface do ArkOS.

## ❤️ Apoie o Desenvolvedor

**Dúvidas, sugestões e contribuições?**  
Leonardo Bruno  
souzalb@proton.me  

**Gostou do tutorial e quer realizar uma contribuição voluntária?**  
*(Pode ser o valor de uma xícara de café ou chá...) ☕ 🍵*  

Chave PIX:  
`8dcc7e3c-0c6a-4c6f-a4c0-26a5e62686db`  

Ou utilize o QR Code abaixo:  

<p align="center">
  <img src="imagens/qrcode-pix.png" alt="QR Code PIX" width="200">
</p>

**Você também pode utilizar o PayPal:**  

[![PayPal](https://img.shields.io/badge/Donate-PayPal-00457C?style=for-the-badge&logo=paypal)](https://www.paypal.com/donate/?hosted_button_id=EQVW5QQ7GBGSY)

<p align="center">
  <img src="imagens/qrcode-paypal.png" alt="QR Code PayPal" width="200">
</p>

**A utilização deste projeto é livre para alterações e adaptações**  
*Desde que feita a devida referência ao repositório original e seu criador.*

