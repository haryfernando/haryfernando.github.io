---
layout: post
lang: id
title: Pindah Partisi di Linux
---
Pernah merasa bingung ketika partisi system linux Anda sudah penuh?

Mount sana, mount sini?

<!-- more -->

Atau apakah Anda mau memindahkan partisi karena merasa partisi ke 5 (/dev/sdb5) terlalu jauh dari /dev/sda dan ingin mengurutkan secara teratur partisi-partisi Anda?

Well, saya sempat bingung dan ingin memperluas space untuk system linux yang makin membengkak ke dalam 1 partisi saja.


Berikut salah satu cara yang telah saya lakukan, dan berjalan baik di system saya:

Misalkan system linux kita di harddisk 1 partisi ke 5. 
Kita ingin memindahkan system kita ke tempat lain, anggaplah hardisk 1 partisi 1 (hore..format ulang tu system yg satu lagi :P).

Maka dapat kita lakukan mount sda1 ke suatu tempat, misal /mnt/sda1

    # mount /dev/sda1 /mnt/sda1

Lalu jalankan perintah sakti berikut :

    # find / -xdev -print0 | cpio -pa0V /mnt/sda1

Perintah di atas secara rekursif menyalin filesystem kita ke /mnt/sda1.

Jika ada partisi yang berbeda lokasi, misalkan /boot berada di partisi yang berbeda dan kita ingin menyatukannya, maka :

    # cd /boot
    # find ./ -xdev -print0 | cpio -pa0V /mnt/sda1

Setelah itu, kita install bootloader.

Boot lewat cd / dvd bootable linux, kemudian mount partisi baru ke suatu lokasi, misalkan /media/baru :

    # mkdir /media/baru
    # mount /dev/sda1 /media/baru

Lalu install grub :

    # grub-install --root-directory=/media/baru /dev/sda

Kemudian (harusnya) kita bisa reboot ke harddisk (atau partisi) baru, tapi system lama. :P

Demikian.

Note : cara ini juga bisa untuk melakukan backup system.
