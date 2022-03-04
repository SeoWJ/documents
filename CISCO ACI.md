# CISCO ACI 교육

## 1일차(22.03.04)

### SDN이 왜 필요한가
	- 복잡한 네트워크에 대한 관리
		- 물리적 네트워크
		- 가상화 네트워크 서비스(가상 스위치/라우터/방화벽 등)
		- 이 두개를 어떻게 관리할 것인가가 목적
	- S/W를 이용한 제어(자동화)
	
### SDN을 어떻게 구현하는가
	- SDN의 구조(Architecture)
		- 중앙집중화
			- 장비들을 그대로 두고 컨트롤러를 따로 만드는 방법(SNMP)
			- SNMP는 장비 외부에 있는 컨트롤러. MIB이나 OID가 정의가 안되어있으면 관리가 어려움. 표준도 없기 때문에 반쯤 실패한 모델.
			- L2, L3, L4등 스위치를 Data-Plane라 함.
			- 전송 테이블을 Control-Plane이라 함.
			- 이 모든걸 구성하는 부분을 Mgmt Plane이라 함.
			- 대부분의 전송장비는 컨트롤플레인까지 트래픽이 올라가지 않도록 하기 위해 전송테이블을 데이터플레인에 장착해(ASIC) 데이터플레인선에서 끝내도록 하는 중.
			- 이 세개의 구조를 SDN Controller라 함.
			- SDN 컨트롤러 중 DataPlane부분을 SDN Fabric이라 함. 패브릭은 L2, L3기능을 제공해야 한다.
			- 엔드포인트 기준으로 DataPlane에 연결되었을때 L2, L3각각 어디에 붙었을때 작동이 달라짐.
			- 따라서 L2를 추상화.
		- Object

### CISCO의 SDN
	- SDN은 1세대 2세대가 존재. 현재 1세대는 문제가 있어 거의 쓰지않음.
	- 2세대는 시스코가 제안한 모델로 현재 많이 사용하는 중.
	
	- DataCenter용 ACI
		- 시스코의 ACI는 Mgmt / Control, Data plane으로 분리. 아까 위에서는 M,C / D로 분리하였음.
		- 시스코는 Control Plane 하드웨어에 Programmable ASIC를 집어넣었다. 에이직의 성능이 나오면서 프로그래밍이 가능해짐. => 이게 Nexus 9000대 스위치.
		- 클라우드 스케일이라는 이름의 칩셋이 들어감. 데이터플레인과 컨트롤플레인을 하나의 칩셋으로 합침. 이 칩셋이 프로그래머블 ASIC이 들어간 칩셋.
		- 여기서 Control, Data plane를 ACI Switch, MGMT plane를 Policy Plane이라 부름.
		- Policy Plane은 objected 기반 구성정보를 담고 이를 Managed Object라 함. 여기에 속성값, 오브젝트에 사용하는 함수값들이 들어감. 이게 ACI의 기능.
		
		- spine/leaf의 2tier network 설계. IS-IS를 사용함. Nexus 9000대 시리즈.
		
	- VXLAN
		- 이더넷을 UDP/IP
			- 이더넷에 VXLAN 헤더를 붙임.
			- 이 헤더를 VXLAN ID라 부름.
			- 
