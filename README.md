# Install-Arch-Linux-EFI-Dual-Boot-I3wm
## Instalação e configuração do Arch Linux UEFI/GPT e Dual Boot com Windows

 _Ao bootar a mídia de instalação do Arch, você estará logado como # (root) em automatic login._<br>
 _*Será utilizado o editor de texto nano ao decorrer do tutorial (para salvar: "ctrl+o", sair: "ctrl+x")_<br><br>
1- Definir um layout de teclado (PT-BR)
> loadkeys br-abnt2 

2- Mudar a fonte do terminal (tty)
> setfont lat0-16

3- Configurar localização/idioma
> nano /etc/locale.gen
>>**descomentar essa linha:**
>>> pt_BR.UTF-8 UTF-8

4- Criar o arquivo de configuração de idioma
> locale-gen
>>export LANG=pt_BR.UTF-8

**5- Conectar sua internet**<br>
*Caso sua internet seja cabeada:
> dhcpcd

Caso sua rede seja wireless:
> wifi-menu

6- Testar a Conexão
> ping -c 3 www.google.com

7- Listar partições existentes:
> fdisk -l

**8- Configurar as partições no disco ( _fica a seu critério, um exemplo para 50gb:_ )**<br>
_*Aqui entra o Dual Boot_
> cgdisk /dev/sda 
>>Observação: _‘sda’ (Primeiro disco rígido SATA) ou ‘sdb’ (Segundo disco rígido SATA) ou 'nvme0n1' (Primeiro SSD M.2)_

Ex de partições:<br> sda1 150MB Recovery<br>
    sda2 512MB Partição EFI<br>
    sda3 700GB Partição do Windows<br>
sda4 * 10G /<br>
sda5 8G swap<br>
sda6 32G /home<br><br>
Escrever/Write — Sim/Yes - Quit/Sair<br><br>
Então nesse exemplo ficou: **/ (root)** no sda4, **swap** no sda5, **/home** no sda6 e já existia uma partição EFI que o windows criou no sda2 (que vai ser usada posteriormente para o /boot)<br>

9- Formatar as partições com o sistema de arquivos ext4:<br>
**_*A partir deste passo, verifique os números dos sda, pois estou utilizando os do exemplo acima!_**
> mkfs.ext4 /dev/sda4
>> mkfs.ext4 /dev/sda6

10- Formatar e ativar swap:
>mkswap /dev/sda5
>>swapon /dev/sda5

**11- Montar partições** 
> mount /dev/sda4 /mnt

12- Montar /home:<br>
_*Partições adicionais são montadas da mesma maneira._
>mkdir /mnt/home
>>mount /dev/sda6 /mnt/home

13- Para o /boot:
>mkdir /mnt/boot
>>mount /dev/sda2 /mnt/boot

14- Visualizar o particionamento atual:
>lsblk /dev/sda

**15- Instalar o sistema base**
>pacstrap /mnt base base-devel
 
16- Gerar arquivo fstab (FSTAB: File System Table):
>genfstab -U -p /mnt/ >> /mnt/etc/fstab

17- Utilizar o ambiente chroot (Chroot: Change Root)
> arch-chroot /mnt /bin/bash

18- Configurar Localização novamente:
> nano /etc/locale.gen
>>**descomentar essa linha:**
>>> pt_BR.UTF-8 UTF-8

19- Gerar arquivo de localização
>locale-gen

20- Criar o arquivo de configuração de idioma novamente:
>echo LANG=pt_BR.UTF-8 > /etc/locale.conf
>>export LANG=pt_BR.UTF-8

21- Recarregue as configurações, pois o ambiente mudou:
>loadkeys br-abnt2
>>setfont lat0–16

22- Para que tais configurações fiquem guardadas, edite o arquivo vconsole.conf:
>nano /etc/vconsole.conf

Escreva e depois salve:
>KEYMAP=br-abnt2
>>FONT=lat0–16
>>>FONT_MAP=

23- Configurar o fuso-horário
>ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime

24- Sincronizar o relógio do hardware com o do sistema
>hwclock --systohc --utc

25- Configurar o repositório para x64
>nano /etc/pacman.conf

_**Descomentar essas linhas e depois salvar:**_
>[multilib]
>>include = /etc/pacman.d/mirrorlist

26- Sincronizar os repositórios
>pacman -Sy

27- Definir um nome para o SO:
> echo nomedopc > /etc/hostname

28- Adicionando entrada em hosts
>nano /etc/hosts

Deixe parecido com isso:
>127.0.0.1 localhost.localdomain localhost
>>::1 localhost.localdomain localhost
>>>127.0.1.1 nomedopc.localdomain nomedopc

29- Baixar Sudo e Grub
> pacman -S sudo grub efibootmgr os-prober

**30- Instalar o GRUB**
>grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub --recheck

31- Criar um ambiente ramdisk inicial
> mkinitcpio -p linux

32- Gerar arquivo de configuração do GRUB
> grub-mkconfig -o /boot/grub/grub.cfg

33- Criar usuario e definir senha
>useradd -m -g users -G wheel,storage,power -s /bin/bash seuuser 
>>passwd seuuser

34- Definir a senha de root
>passwd root

35- Editar o arquivo sudoers:
>nano /etc/sudoers

_**Descomente a opção e salve**_
> %wheel ALL=(ALL) ALL

**36- Instalar componentes do Wi-Fi.**
>pacman -S wpa-supplicant networkmanager net-tools
>>systemctl enable NetworkManager

Em seguida, você pode configurar Ethernet ou Wifi:
>ifconfig -a

No meu caso foi detectado o wlp3s0:
>ifconfig wlp3s0 up

_**37- Caso esteja instalando em um notebook, o seguinte comando para drivers de touchpad:**_
>pacman -S xf86-input-synaptics

**Pronto para reiniciar o sistema!**
>exit
>>umount /mnt/boot
>>>umount /mnt/home
>>>>umount /mnt

>reboot

**O sistema base está instalado!!!**
