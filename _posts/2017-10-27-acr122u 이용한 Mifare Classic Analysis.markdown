---
layout: post
title:  "ACR122U를 이용한 Mifare Classic 분석"
date:   2017-10-27
---


### ACR122U
NFC 리더기로써 libnfc의 사용이 가능한 러더기 제품 중 하나이다. Mifare Series, ISO 14443 호환 태그, ISO/IEC 18092태그까지 Read/Write 할 수 있다.

<img src="{{ '/assets/img/acr122u_spec.jpg' | prepend: site.baseurl }}" alt=""> 

[ACR122U 리더기 스펙](https://www.acs.com.hk/en/products/3/acr122u-usb-nfc-reader/)

Proxmark3에 비하여 저렴한 가격으로 국내에 유통되고 있으며, 무료 Library인 nfc-tools의 nfclib을 이용하여 분석이 가능하다. Mifare Classic Key Crack 프로그램인 MFOC, MFCUK를 사용할 수 있다.

ACR122U리더기를 이용한 Mifare Classic 분석으로 ACS ACR122U SDK(software Develoment Kit)및 NFC TOOLS의 LIBNFC을 이용하여 분석을 수행하였다.

### ACR122u sdk

APDU(Application Protocol Data Uint) Commands를 이용하여 카드의 정보를 읽거나 쓰는 가능을 제공하며, GUI 기반으로 작성되어 사용이 직관적이다. 단, 분석에 있어 Block 단위로 Key를 입력하고 값을 얻어야하기 때문에 번거로움이 있다.

이러한 명령어는 APDU Commands를 기반으로 이루어짐으로써 Send Commands의 APDU Command를 통하여 HEX값으로 명령어를 입력해주면 동일하게 작동한다.

APDU에 관한 명령어는 ISO 7816, ISO 14443 규격에 따라 작성되었고, Card규격에 따라 인증 과정이나 요구하는 Key의 형태가 다르다. 추가적인 기능 개선이나, 소스 코드 개발을 위해서는 APDU Command를 이용하거나, Python의 pyscard, C의 libnfc, Android의 Android.nfc 등이 있다. 

acr122u sdk는 구매를 통하여 얻을수 있으며, 인터넷를 뒤져보면 다운 받을수 있다.

그 밖에도 간단한 프로그램으로는 [ACS Util](https://www.acs.com.hk/en/utility-tools/)에서 제공해주고 있으나, 별로 효율적이지는 않다.

<img src="{{ '/assets/img/APDU%20commands.jpg' | prepend: site.baseurl }}" alt=""> 

Readers Commands의 New Connections를 통하여 장비를 선택하여 연결한다. 연결이 되고 난후 제공되는 기능은 Firmware Version, LED and Buzzer, Antenna ON/OFF, Status, Get Data, Get ATR, Set PICC Operating Parameter, Get PICC Operating Parameter 가 있다.
 
Connection이 끝난 이후에 Get ATR을 통하여 Card Command창이 팝업 되고, Load Authenticate, Read, Write, Read Value, Wire Value, Increment, Decrement, Copy 기능을 제공한다.

Load Authentication Keys를 통하여 알고 있는 Key A, B를 입력 할 수 있으며, Authenticate를 이용하여 Block 단위로 인증을 확인할 수 있다. Read기능을 통하여 Block단위로 Hex, ASCII값으로 출력이 가능하다.


### nfc-tools

[NFC-tools Wiki](http://nfc-tools.org/index.php?title=Main_Page)

[NFC-tools github](https://github.com/nfc-tools)

[LIBNFC-운영체제별 설치방법](http://nfc-tools.org/index.php?title=Libnfc)


NFC Tools의 libnfc는 C언어를 기반으로 작성되었으며, 리더기 인식 및 속도측면에 서 가상환경에 사용했을 시 어려움이 있다. 따라서 host환경에서 사용하여야 하며 개발된 program은 Linux 환경에서 사용하는 것이 최선이다. PENTOO는  Penetration testing과 보안목적의 Lived OS로써 우리의 분석환경에 있어서 좋다고 판단하여 선택하였다.

PENTOO를 universal usb installer와 같은 USB 부팅이미지를 만들어 주는 툴을 이용하여 USB로 부팅 하여 사용한다. 부팅 후 advanced options로 진입하여 pentoo로 부팅하면 된다. CUI OS가 나오게 되면 startx 명령어를 통해 GUI OS로 진입할수 있다. 이후 Command창을 띄워서 관리자 권한을 얻기위한 su명령어를 실행한 후 인터넷 설정(service NetworkManager start), Pentoo 비밀번호 설정(passwd pentoo) 설정되어야 한다. 

Key 크랙 과정은 Key사이의 유사도와 관계가 있어서 한 개의 키당 1분 ~ 30분의 시간의 걸릴 수 있다. 허나 산업군에 따라 유사한 키를 사용하는 것을 확인하였고, 이러한 키를 이용하여 MFCUK를 사용하지 않고 다른 Key 크랙 과정을 수행할 수 있다.


PENTOO OS에서는 emerge libnfc 명령어를 이용하여 nfc-tools의 libnfc를 설치할 수 있다. 사전에 리눅스 모듈에서 pn533, nfc를 제거해주기 위해서 modprobe –r pn533, modprobe –r nfc가 선행되어야 한다. ACR122U의 연결 확인을 위해서는 nfc-list라는 명령어를 이용하여 연결 된 것을 확인 할 수 있고, LIBNFC_LOG_LEVEL=3 nfc-list를 사용하여 해당 연결에 디버그 단계까지 정보를 확인 할 수 있다. 해당 연결이 수행되면 다음과 같은 결과가 출력된 것을 볼 수 있다.

추가적으로 카드 규격 및 정보를 확인하기 위해서 nfc-list –v라는 명령어를 사용한다.

<img src="{{ '/assets/img/Screenshot_2017-10-28_14-42-59.png' | prepend: site.baseurl }}" alt=""> 

이제 mfoc를 사용하기 위해서는 github에서 repository를 복제해야하는데 이를 위해서 git을 설치해야한다. emerge git를 통하여 git를 설치한 다음에 git cloen 명령어를 통하여 nfc-tools의 mfoc, mfcuk를 복제하여 컴파일 과정을 수행한다.

MFOC, MFCUK 컴파일 과정은 autoreconf 환경에 맞게 재빌드를 해주는 명령어로써 autoreconf –vis옵션으로 수행한후, 생성된 ./configure를 실행하고, make를 수행하여 설치할 수 있다. 

### mfoc

기본키나 알고있는 키를 기반으로 하여 mifare classic의 crypto1 취약점을 이용하여 나머지 키를 알아낸다.

mfoc –o card_dump.mfd –P 999 –k ffffffffffff –k 12345678 | tee card_dump.txt
-o 결과 파일 출력
-P 한 개의 키를 얻어 내기 위한 최대횟수
-k 알고 있는 key
tee cmd로 출력된 값을 txt로 받아준다.


MFOC 프로그램은 각 해당 Sector에 대한 Key만 알고 있어도 CRYPTO1의 구조적 취약점을 이용해 나머지 모든 Key를 알아 낼 수 있는 공격기법이다. 

<img src="{{ '/assets/img/Screenshot_2017-10-28_14-46-43.png' | prepend: site.baseurl }}" alt=""> 

mfoc –o card_dump.mfd –P 999 –k ffffffffffff –k 12345678 | tee card_dump.txt
-o 결과 파일 출력
-P 한 개의 키를 얻어 내기 위한 최대횟수
-k 알고 있는 key
tee cmd로 출력된 값을 txt로 받아준다.

<img src="{{'/assets/img/Screenshot_2017-10-28_14-48-01.png' | prepend: site.baseurl }}" alt=""> 


### mfcuk

MFCUK 프로그램은 모든 Sector의 Key를 모르는 경우 사용한다. 이와 같은 경우에도 CRYTO1의 구조적인 취약점을 이용해 하나의 Key를 알아내고 MFOC를 수행한다.

./mfcuk -C -R 0:A -s 250 -S 250 –v 3

다음 그림과 같이 진행 되는 것을 확인 할 수 있고, 
<img src="{{ '/assets/img/Screenshot_2017-10-28_14-50-40.png' | prepend: site.baseurl }}" alt=""> 


다음 그림과 같이 1개의 키값을 얻어내는것을 알 수 있다.
<img src="{{ '/assets/img/Screenshot_2017-10-28_16-31-00.png' | prepend: site.baseurl }}" alt=""> 


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

Mifare Classic의 취약점이  많이 알려진 지금에서도, 다양한 사업군에서 Mifare Classic이 적용되어 사용되고 있다. 

이러한 문제를 해결 하기 위해서는 기본키 또는 알려진 키를 사용하지 않으며,
금액 정보와 같은 중요정보를 인코딩을 사용하고, CheckSum을 사용하여 값의 변조를 확인하고,
사업군 고유의 의미 있는 사용정보를 담아서 위험을 최대한 감소하여야 한다.

무엇보다도 중요한건 자신의 사업에 있어서, 이러한 취약점이 있음에도 사용에 문제가 없다는 수용가능 위험수준(Degree of Assurance, DOA)을 평가하여 카드의 규격을  선택하는 것이다.

더 안전한 모델로는 Mifare 에서 Plus SL(Security Level)3 , Desfire가 있을수 있다.


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