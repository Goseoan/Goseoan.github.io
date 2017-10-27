---
layout: post
title:  "ACR122u를 이용한 Mifare Classic Anaylsis"
date:   2017-10-27
---

# ACR122u를 이용한 Mifare Classic Anaylsis

## ACR122U
NFC 리더기로써 libnfc의 호환이 가능성 러더기 제품 중 하나로써, Mifare Serise, ISO 14443 호환 태그 뿐만 아니라, ISO/IEC 18092태그까지 Read/Write 할 수 있다.

https://www.acs.com.hk/en/products/3/acr122u-usb-nfc-reader/

nfc-tools의 nfclib를 이용하여 카드의 규격 확인 및 다양한 정보를 얻을 수 있고,

mfoc, mfcuk를 사용하여 mifare classic card의 password를 크랙 할 수 있다.

http://nfc-tools.org/index.php?title=Main_Page
https://github.com/nfc-tools

## ACR122u sdk

ACR122u를 사용방법으로는 ACS ACR122U SDK(software Develoment Kit), libnfc가 좋다고 판단한다.

acr122u sdk는 구매를 통하여 얻을수 있으며, 인터넷를 뒤져보면 자료가 있다.

그 밖에도 간단한 프로그램으로느 https://www.acs.com.hk/en/utility-tools/에 acr122u프로그램을 제공해주고 있다.


APDU(Application Protocol Data Uint) Commands를 이용하여 정보를 얻어오거나 실행하는게 가능하며 GUI 기반으로 작성되었으며, 사용이 직관적이다.

## nfc-tools

NFCLIB를 이용하기 위해서는 host 환경에 linux에서 사용하는 것이 최선이며, windows에서는 많은 오류와 버그가 발생한다. nfc 분석환경에서 hack rf one를 최선으로 사용하기 위해 pentoo를 사용하기 때문에 pentoo linux를 이용하여 libnfc를 이용하는 환경을 구축한다.

pentoo에서는 emerge libnfc를 이용하여 libnfc를 설치하며 다른운영체제의 설치 방법은 다음 아래 사이트를 참조하면 된다.

http://nfc-tools.org/index.php?title=Libnfc

pentoo는 liveos 운영체제로 하였으며, usb에 설치 하여 동작하였다.
booting과정은 boot순서를 usb를 제일 위로 올려주어서 부팅하여야한다.
LG gram모델같은 uffi모드와 secure boot모드를 해제 하여 실행하였다.
advanced에서 pentoo를 실행하여
startx명령어를 통하여 gui 운영체제로 부팅하였고

command를 통하여 기본적인 설정을 한다.

sudo 
passwd pentoo (스크린세이버가 걸리면 패스워드를 요구 한다. 스크린세이버 데몬을 죽여도 된다.)
service NetworkMange start(Wifi 설정을 위하여 실행하였다.)

emerge nfclist
modprobe –r pn533
modprobe –r nfc

를 하여 acr122u를 통하여 nfclib tool을 사용할 수 있다.

nfc-list

카드 정보 확인
nfc-list –v

mfoc
기본키나 알고있는 키를 기반으로 하여 mifare classic의 crypto1 취약점을 이용하여 나머지 키를 알아낸다.

mfoc –o card_dump.mfd –P 999 –k ffffffffffff –k 12345678 | tee card_dump.txt
-o 결과 파일 출력
-P 한 개의 키를 얻어 내기 위한 최대횟수
-k 알고 있는 key
tee cmd로 출력된 값을 txt로 받아준다.

./mfcuk -C -R 0:A -s 250 -S 250 -v 3

pentoo libnfc , mfoc, mfcuk install shell

```
# OS is pentoo

# pentoo use libnfc

# su
# passwd pentoo
# service NetworkManager start

# Access WIFI OR ETHERNET

# emerge --sync # choice
# emaint -a sync
# emerge-webrsync
# eix-sync
# emerge -a eix

#install libnfc
emerge libnfc
echo "blacklist nfc\nblacklist pn533\nblacklist pn533_usb" >> /etc/modprobe.d/blacklist.conf
modprobe -r pn533
modprobe -r nfc
LIBNFC_LOG_LEVEL=3 nfc-list
nfc-list

#install mfoc
# emerge mfoc
emerge --ask dev-vcs/git
cd ~/Desktop/
git clone https://github.com/nfc-tools/mfoc
cd mfoc
autoreconf -vis
./configure
make

#install mfcuk
cd ~/Desktop/
git clone https://github.com/nfc-tools/mfcuk
cd mfcuk

autoreconf -vis
./configure
make

cd src
./mfcuk -C -R 0:A -s 250 -S 250 -v 3


# one console pcscd - purpose deduging

# one console mfoc
# mfoc -O coinup.mfc | tee coinup.txt 

# card dump 
# nfc-mfclassic w B output.mfd output.mfd

# nfc-mfclassic R A dump.mfd
# mfoc -P 500 -O keyFileDump.mfd

# nfc-mfclassic r A dump.mfd keyFileDump.mfd
# vi dump.mfd
# esc :%!xxd

```

그 밖에도 분석하는 방법에는 

MCT(Mifare Classic Tool) App을 통한 분석을 할수 있다.
- http://blog.naver.com/ndb796
