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

![Tabela corrigida](/imagens/06-imagem-após-ajuste.png) 

Opa parece que a imagem agora mostra corretamente as partições.

Agora vamos proceder com a gravação da imagem no cartão microsd. Fica ao seu critério utilizar um programa gráfico ou não.
Como eu curto fazer as coisas pelo terminal pela praticidade vamos lá:

Para ler o cartão no meu computador vou usar o meu leitor de cartão. Use um cartão microsd de pelo menos 16GB para a imagem caber.

![Tabela corrigida](/imagens/07-leitor-cartão-cartão.jpeg) 


