# FFS 분석

이번 과제에서는 fast file system, FFS에 대한 기본적인 이해를 필요로 하는 과제로 파일시스템의 구조와 흐름을 이해하고 팀원의 학번에 지정된 파일 두 개를 파일시스템 구조를 분석하여 찾아내고 마운트 시에 특정 문장을 출력하는 것을 목표로 하고 있습니다.
구축환경은 가상머신에서 우분투 환경에서 실행되었습니다.

![image](https://user-images.githubusercontent.com/26988563/160741713-a8c653ca-9827-46cb-b132-48de6931032b.png)

처음으로 파일시스템에 접근하기 위해 관리자 권한을 얻는 sudo명령어를 써줍니다.
관리자 권한을 얻은 후 make를 해서 필요한 파일들을 생성하고 인스톨모듈 명령어를 사용하여 파일시스템을 마운트할 램 디스크를 만들어줍니다.
그 후 모듈이 설정되었는지 lsmod사용해 모듈이 올라간 것을 확인할 수 있습니다. 만들어진 램 디스크의 사이즈를 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/26988563/160741719-2c9803f6-e281-42a3-8271-052849ab09dc.png)

다음으로 파일시스템을 마운트할 디렉터리를 생성했습니다.

![image](https://user-images.githubusercontent.com/26988563/160741719-2c9803f6-e281-42a3-8271-052849ab09dc.png)

dd명령어를 사용하여 램 디스크의 하위 폴더 중 zero 즉 비어있는 것들을 청소해줬습니다.
그 후 mkfs명령어로 파일시스템을 생성해준 후 /dev/ramdisk 의 마운트 디렉터리에 만들어진 파일시스템을 마운트해준 후 df 명령어를 실행 후 제일 아랫줄을 보면 지정된 곳에 마운트 된 것을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/26988563/160741730-87c9e1a3-6af0-4ad1-9825-e76158bd7aa7.png)

마운트가 제대로 되었다면 준비된 쉘 스크립트를 실행시켜 줍니다. 마운트 디렉토리에 뭔가 작업이 계속해서 되고 있으며 완료된 시점에서는 여러 파일들이 생성된 것을 볼 수 있습니다. 아마 쉘 스크립트가 폴더들을 생성되며 파일시스템이 이를 인식하여 데이터들을 입력했을 것이라 사료됩니다.
성공적으로 쉘 스크립트를 실행했다면 xxd명령어를 사용하여 hex파일에 출력값을 입력으로 >를 사용하여 넣어줍니다. 여기서 
xxd –a –g 1 –s+0x00이라는 옵션에 대하여 설명하자면 첫 번째 옵션 a는 토글 자동생략으로 0인 부분을 *으로 출력해주고 두 번째 옵션 g는 바이트 단위로 출력하겠다는 뜻입니다. 따라서 g 1은 1바이트 단위로 출력을 하게 됩니다. 마지막 옵션인 s는 seek기능을 뜻합니다.
여기까지가 파일 traking을 위한 준비입니다. 정리하자면, 

> 1. 관리자 권한을 얻습니다.
> 2. 램 디스크, 저장소를 지정해줍니다.
> 3. 파일시스템을 생성 후 지정된 저장소에 마운트 합니다.
> 4. 저장소에 파일들이 생성됩니다. (여기선 작위적으로 생성)
> 5. 16진수로 파일들을 보여주는 명령 또는 툴을 사용하여 파일시스템 해석

### FFS 분석

우선 분석에 앞서 파일시스템의 부트 레코드의 구조에 대하여 먼저 알아봤습니다.

![image](https://user-images.githubusercontent.com/26988563/160741742-cd17fe41-7bad-4f22-b902-c1916b64ce34.png)

위의 구조표에 입각하여 부트 레코드를 분석해봤습니다. 여기서 실제로는 파티션의 부트 레코드 영역을 찾아가는 게 선결되어야 합니다. 보통 부트 레코드는 예약된 영역의 첫 번째 섹터에 존재하기 때문에 파티션 테이블에서 LBA 시작주소를 찾아보고 그 주소를 따라 ??번재 섹터를 찾아가 부트 레코드를 찾아가야 하지만 이번 과제에서는 파티션이 따로 나뉘지 않았기 때문에 부트 레코드는 0부터 시작합니다.

![image](https://user-images.githubusercontent.com/26988563/160741749-5ce4f505-205d-4614-b45b-a553862f9c8d.png)

표와 대조를 해본다면 모든 부위에 대한 값을 알 수가 잇습니다. 여기선 중요 포인트만 체크하도록 하겠습니다. 해석할 때 리눅스이므로 리틀엔디안 방식으로 해석하는 것을 주의합니다.

* JUMP BOOT code: 0x9058eb
* Bytes per Sector:0x0200, 섹터의 크기는 512byte
* Sector per Cluster:0x08 , 하나의 클러스터에 8개의 섹터, 클러스터의 크기는 4096바이트
* Total Sector 32:0x002000, 320개의 섹터
* FAT size 32:0x000007fc, 7252개의 섹터
* Root Directory Cluster:0x00000002, cluster 2
* File system information:0x0001, sector 1

나머지 정보들도 표와 대조해보면 알 수 있습니다. 여기서
(boot record의 섹터 크기) * (한 섹터의 크기) = 20 * 200 = 4000 (hex),
boot record의 영역이 4000(hex)의 크기를 가진다는 것을 알아냈습니다.

뿐만 아니라 fat1, fat2의 크기 또한
(fat1의 섹터 크기) * (한 섹터의 크기) = 7fc * 200 = ff800 (hex)
(fat2의 섹터 크기) * (한 섹터의 크기) = 7fc * 200 = ff800 (hex)
로 각각 ff800(hex)라는 것을 알아냈습니다.

따라서 fat1의 시작주소는 0x00000000 + 4000 = 0x00004000이고, fat2의 시작주소는 4000 + (792 * 200) = 0x000ff800 이고, data영역의 시작주소는 ff800 + (7fc * 200) = 0x00203000입니다.

부트 레코드 영역 외에도 
Reserved area: 파일시스템 information구조체나 백업 정보가 기록됩니다.
FAT area: 파일시스템의 주요정보가 입력됩니다. 파일과 디렉토리의 할당에 대한 정보가 클러스터 단위로 관리됩니다.
Data area: 실제 데이터가 저장되는 영역으로 이름이나 확장자, 파일에 대한 정보들이 저장됩니다.

![image](https://user-images.githubusercontent.com/26988563/160741752-4a418976-4e72-4227-a634-dd6d8b65f5aa.png)

위 사진은 데이터 영역의 구조입니다. 이 구조를 기반으로 파일의 위치를 탐색 합니다. 위의 구조에서 이번 과제에서 핵심적으로 봐야 할 곳은
Name 영역과 First Cluster Low영역입니다. 이름을 파악하고 해당 클러스터의 위치를 찾아 갈수 있게 해줍니다. 예를 들어

![image](https://user-images.githubusercontent.com/26988563/160741755-11e363d1-1c10-457f-822a-9b93fdb010b6.png)

![image](https://user-images.githubusercontent.com/26988563/160741763-56f4cdd1-a973-4b01-8d37-06f2a4bd012d.png)
16진수, 아스키 해석을 도와준 프로그램입니다.

4c414232==LAB2디렉토리의 위치는 0x0005, 5번 클러스터라는 것을 알 수 있습니다. 여기서 현재 클러스터를 빼주고 클러스터당 섹터 개수만큼 곱해주면 데이터영역에서 몇 섹터만큼 이동해야 하는지를 알 수 있습니다.

### FFS 분석을 기반으로 파일탐색

분석을 끝냈으니 실제 파일을 루트부터 시작해서 헥사코드를 해석해가며 디렉토리 주소들을 찾아가며 목표에 도달하는 일만 남았습니다. 저희의 학번은 이수민:32143313 전세호: 32144107로 %40을 시킨다면 각각 27과 33이 나옵니다. 따라서 저희의 목표의 위치는

![image](https://user-images.githubusercontent.com/26988563/160741775-7c74a81a-ad3a-41b0-9de5-c9687830ac01.png)

가 되겠습니다.

이제 위에서 설명한 방식대로 이번 목표를 찾아가겠습니다.
0x00203000부터 트래킹을 시작하면서 저희 팀이 찾아야 하는 27과 33의 디렉토리를 탐색했습니다.

![image](https://user-images.githubusercontent.com/26988563/160741777-3f4266b8-6ac6-46e1-8e9c-19e3f96dafea.png)

lab_folder_27 : 0x00203a00

lab_folder_33 : 0x00203b80

라는 정보를 찾아내었습니다. 그리고 각 디렉토리에 해당하는 High-order부분과 Low-order 부분을 찾았습니다. 이를 밑줄로 표기한 후 Low 부분 – High 부분을 해준 후 첫 두 바이트는 BS와 Root부분이므로 제외하고 다루기 때문에 이를 적용하여 계산합니다. 첫 번째 계산을 다한 후 구해진 값은 클러스터 단위에서 섹터단위로 바꾸어 주어야 하므로 마지막을 *8을 해주어야 합니다. 각 디렉토리의 하위 디렉토리에 대한 정보를 얻을 수 있는 값은 아래와 같습니다. 

0x00203000 + {(59 – 00 - 2) * 200} * 8 = 0x0025a000 (lab_inside_folder_27)

![image](https://user-images.githubusercontent.com/26988563/160741782-1844a9cb-f4df-446e-86ea-06d7a9ac3ae8.png)

0x00203000 + {(6d – 00 - 2) * 200} * 8 = 0x0026e000 (lab_inside_folder_33)

이제 각 lab_inside_folder의 하위 디렉토리 혹은 파일을 찾아야 합니다. 방식은 전과 동일하게 하위 데이터에 대한 정보를 추적합니다. 그 결과 다음과 같은 결과를 도출했습니다.

0x00203000 + {(dc – 00 – 2) * 200} * 8 = 0x002dd000 (27.txt)

0x00203000 + {(ec – 00 - 2) * 200} * 8 = 0x002ed000 (33.txt)

![image](https://user-images.githubusercontent.com/26988563/160741788-5c39165f-8b13-4a23-b258-37bc885ae6af.png)

마지막으로 각 txt파일의 내용을 추적합니다. 방법은 마찬가지로 같습니다.

0x00203000 + {(178 – 00 – 2) * 200} * 8 = 0x00379000 (27.tx의 내용부분)

0x00203000 + {(13c – 00 - 2) * 200} * 8 = 0x0033d000 (33.txt의 내용부분)

![image](https://user-images.githubusercontent.com/26988563/160741799-0cfdb093-40a6-4901-b11a-57604b125604.png)

이 부분은 결과를 확인하면서 찍은 화면입니다. -g 옵션을 4로 바꾸어 출력해 보았습니다. 각 결과의 시작부분은 찍지 못했지만 파일의 긴 내용을 확인할 수 있었습니다. 

아래는 좀 더 가독성있는 캡처를 위해 다른 팀원이 해당 내용을 다시 수행하며 캡처한 결과사진입니다. 순서대로 경로를 찾아가는 내용에 해당됩니다.

![image](https://user-images.githubusercontent.com/26988563/160741809-6cfd9e8a-4d71-4f06-8313-fe31e8f7432e.png)

![image](https://user-images.githubusercontent.com/26988563/160741818-6b274b9c-2715-49d3-a2ca-43ff20166f5a.png)

![image](https://user-images.githubusercontent.com/26988563/160741822-62675e7d-cdb8-4d91-bd53-160f2f862204.png)

![image](https://user-images.githubusercontent.com/26988563/160741825-5074b265-6b48-4c3b-a784-776e71e99f3c.png)

![image](https://user-images.githubusercontent.com/26988563/160741835-08710661-1c7b-47c1-ad4b-c234a42785b3.png)

![image](https://user-images.githubusercontent.com/26988563/160741841-2e62bb17-a015-423a-8af7-37e69fe11884.png)

### 마운트 수행과정 이해와 마운트 시 출력

이번의 목표는 마운트 시에 팀원의 학번과 이름을 출력하는 것입니다.
간단하게 마운트 시에 실행되는 코드의 경로를 찾아가서 그 코드부분에 출력부분을 추가한
후 마운트를 실행시켜 제대로 출력되는지 확인하는 것으로 끝냈습니다. 아래는
그 과정에 대한 캡처사진입니다.

![image](https://user-images.githubusercontent.com/26988563/160741846-3cc0ea26-b6d3-4cb8-b5b5-8ad20ac9bbe1.png)

우선 일전과 마찬가지로 준비를 해줍니다.

![image](https://user-images.githubusercontent.com/26988563/160741852-9910c161-1c39-490d-9dc9-af3935e8345e.png)

마운트 코드가 있는 경로로 찾아가는 내용입니다.

![image](https://user-images.githubusercontent.com/26988563/160741856-93cccbe0-502e-4082-97b2-be4bd9de723d.png)

그리고 출력하고 싶은 내용을 작성해줍니다.

![image](https://user-images.githubusercontent.com/26988563/160741861-9dcb2b44-49a3-4689-9505-2e2155617597.png)

성공적으로 출력되는 것을 확인할 수 있습니다.

