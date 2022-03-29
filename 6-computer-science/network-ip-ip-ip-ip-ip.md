# NetWork, IP종류 (공인IP, 사설IP, 고정IP, 유동IP)



### IP종류 <a href="#ip" id="ip"></a>

| 구분           | 종류         |
| ------------ | ---------- |
| 범위에 따라       | 공인IP, 사설IP |
| 변경여부         | 고정IP, 유동IP |
| VPN, 프록시 사용시 | 가상IP (VIP) |

![isp](https://user-images.githubusercontent.com/55946791/83545520-4acf3080-a53a-11ea-83e7-c19dd1bf38c2.jpg)

### 범위에 따라 <a href="#undefined" id="undefined"></a>

#### 공인IP <a href="#ip" id="ip"></a>

* 각 나라마다 관리하는 기관이 있다. 우리나라는, 한국 인터넷 진흥원에서 관리
* 통신사와 ISP에 의해 개별 사용자에게 IP주소가 배정된다
* **세상에 단 하나뿐인 IP**
  * 동적 vs 고정
  * 가정용 IP : 유동IP
  * 기업용 IP : 고정IP

#### 사설IP <a href="#ip" id="ip"></a>

* local N/W에서 host를 인식하기 위해 사용되는 공인IP주소의 하위IP주소
* local N/W에서만 식별된다
* **내부망 전용 IP**
* 공유기 사용을 통한IP (공인기관에서 허가받지X IP)

#### NAT <a href="#nat" id="nat"></a>

* 하나의 공인IP를 여러 서버가 공유가능
* ex) 핸드폰, 노트북, 데스크탑을 하나의 IP로 연결해서 사용가능

### 변경여부 <a href="#undefined" id="undefined"></a>

#### 고정IP <a href="#ip" id="ip"></a>

* 인터넷서비스공급자(ISP)에 의해 특정IP를 특정user에게 변경되지 않는 고유한 IP 지정
* pc, server등이 리부팅, 재접속 되도 IP주소 변경X

#### 유동IP <a href="#ip" id="ip"></a>

* ISP가 보유한 IP주소 중에 미사용중인 IP할당
* DHCP라고도 불림
* 즉, pc, server등이 리부팅, 재접속시 IP주소 변경 될수있다
* 왜 사용해?
  * 개인 정보 보호를 위해.
  * 고정IP는 접속 주소가 일정해서 외부에서 접속이 쉽다. 따라서 계속 변경해준다

### VIP <a href="#vip" id="vip"></a>

* proxy나 VPN을 활용하여 사용자측 IP와 다른 IP를 인터넷 서비스에 활용하는 IP
* **외부에 노출된 IP와 실제 내부에서 사용되는 IP 맵핑**

> VIP, MIP, DIP
>
> * IP를 연결시켜주는 방식

#### VIP : Virtual IP <a href="#vip--virtual-ip" id="vip--virtual-ip"></a>

* **1 (외부 대표 IP) : N (내부 IP )**
* 대표적으로 L4스위치를 이용해서 여러 대의 서버 운영시 사용자 요청 받아들이는 VIP

#### MIP : Mapped IP <a href="#mip--mapped-ip" id="mip--mapped-ip"></a>

* **1 (외부 대표 IP) : 1 (내부 IP )**

#### Policy-based NAT-dst (DIP : Dynamic IP) <a href="#policy-based-nat-dst-dip--dynamic-ip" id="policy-based-nat-dst-dip--dynamic-ip"></a>

* [https://ojava.tistory.com/118](https://ojava.tistory.com/118)

> [https://m.blog.naver.com/xcripts/70121283191](https://m.blog.naver.com/xcripts/70121283191) [http://ip.esran.com/what\_ipv4.php](http://ip.esran.com/what\_ipv4.php)

출처 : [https://devham76.github.io/network/network-typeOfIP/](https://devham76.github.io/network/network-typeOfIP/)
