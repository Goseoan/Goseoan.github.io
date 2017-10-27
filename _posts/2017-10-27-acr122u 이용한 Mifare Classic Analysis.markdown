---
layout: post
title:  "ACR122u를 이용한 Mifare Classic Anaylsis"
date:   2017-10-27
---


### ACR122U
NFC 리더기로써 libnfc의 호환이 가능성 러더기 제품 중 하나로써, Mifare Serise, ISO 14443 호환 태그 뿐만 아니라, ISO/IEC 18092태그까지 Read/Write 할 수 있다.

[ACR122U 리더기 스펙](https://www.acs.com.hk/en/products/3/acr122u-usb-nfc-reader/)

nfc-tools의 nfclib를 이용하여 카드의 규격 확인 및 다양한 정보를 얻을 수 있고,

mfoc, mfcuk를 사용하여 mifare classic card의 password를 크랙 할 수 있다.



### ACR122u sdk

ACR122u를 사용방법으로는 ACS ACR122U SDK(software Develoment Kit), libnfc가 좋다고 판단한다.

acr122u sdk는 구매를 통하여 얻을수 있으며, 인터넷를 뒤져보면 다운 받을수 있다.

그 밖에도 간단한 프로그램으로는 [ACS Util](https://www.acs.com.hk/en/utility-tools/)에서 제공해주고 있다.

별로 효율적이지는 않다.

APDU(Application Protocol Data Uint) Commands를 이용하여 정보를 얻어오거나 실행하는게 가능하며 GUI 기반으로 작성되었으며, 사용이 직관적이다.

### nfc-tools

[NFC-tools Wiki](http://nfc-tools.org/index.php?title=Main_Page)

[NFC-tools github](https://github.com/nfc-tools)

NFCLIB를 이용하기 위해서는 host 환경에 linux에서 사용하는 것이 최선이며, windows에서는 많은 오류와 버그가 발생한다. nfc 분석환경에서 hack rf one를 최선으로 사용하기 위해 pentoo를 사용하기 때문에 pentoo linux를 이용하여 libnfc를 이용하는 환경을 구축한다.

pentoo에서는 emerge libnfc를 이용하여 libnfc를 설치하며 다른운영체제의 설치 방법은 다음 아래 사이트를 참조하면 된다.

[LIBNFC-운영체제별 설치방법](http://nfc-tools.org/index.php?title=Libnfc)

pentoo는 liveos 운영체제로 하였으며, usb에 설치 하여 동작하였다.
booting과정은 boot순서를 usb를 제일 위로 올려주어서 부팅하여야한다.
제가 사용하는 LG gram모델같은 경우 uffi모드와 secure boot모드를 해제 하여 실행하였다.
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

연결확인 nfc-list

카드 정보 확인 nfc-list –v

### mfcuk
mfcuk는 Mifare Classic의 Crypto1 취약점을 이용하여 하나의 키를 알아내는 프로그램으로써 아무런 키값을 모르는 경우 사용한다.
키를 모르는 경우 MFOC를 사용하기 위해서 선행되어야 한다.

./mfcuk -C -R 0:A -s 250 -S 250 -v 3

### mfoc
기본키나 알고있는 키를 기반으로 하여 mifare classic의 crypto1 취약점을 이용하여 나머지 키를 알아낸다.

mfoc –o card_dump.mfd –P 999 –k ffffffffffff –k 12345678 | tee card_dump.txt
-o 결과 파일 출력
-P 한 개의 키를 얻어 내기 위한 최대횟수
-k 알고 있는 key
tee cmd로 출력된 값을 txt로 받아준다.



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


nfclib 를 통하여 Key를 크랙하고, 얻은 key 를 가지고 MCT APP을 통하여 분석하는 방법이 제일 효율적이였다.

### 보안적인 대응방법

아직까지 Mifare Classic이 사용되는 있는 시점에서 이러한 문제를 해결 하기 위해서는 기본키 또는 알려진 키를 사용하지 않으며,
금액 정보와 같은 금액 정보에 인코딩을 사용하고, CheckSUM을 사용하여 위험을 최대한 감소 하는 방법이 있을수 있다.

무엇보다도 중요한건 자신의 사업에 있어서, 이러한 취약점이 있음에도 사용에 문제가 없다는 수용가능 위험수준(Degree of Assurance, DOA)을 평가하여 카드의 규격을  선택하는 것이다.

더  안전한  모델로는 Mifare 에서 Plus SL(Security Level)3 , Desfire가 있을수 있다.


### 추가적인 분석하는 방법 

- [MCT(Mifare Classic Tool) App을 통한 분석](http://blog.naver.com/ndb796)을 할수 있다.

- Proxmark3를 이용한 분석 
현재 Proxmark3가 rfid 연구 분석 목적으로 제일 좋으며, 다양한 Read Card Spec를 보유하고 있다.
ACR122U 가격과 얼마 차이가 안나는 Proxmark3 easy를  구매할수 있는 걸로 확인된다.
다만 해외 배송이라는 단점이 있다.

### 도움 되는 정보

- 카드 규격
Mifare Classic
Mifare Ultralight

- APDU관련
iso 1816
iso 14443

### 주의사항
- 윤리의식
자료의 내용은 사회적으로 보안적으로 도움이 되기 위해서 작성하였으며, 잘못 사용하여 문제가 발생하였을 시 모든 책임은 본인에게 있습니다.

- 안내
여기의 기재된 내용은 본인의 연구를 통한 자료이며, 개인적인 의견이 반영 되었습니다. 잘못된 부분이 있으면 이메이을 통하여 문의하여 주세요.