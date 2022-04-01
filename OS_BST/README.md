# 이진탐색트리란?

이진탐색트리는 이진트리의 일종으로, 노드의 왼쪽가지에는 노드의 값보다 작은 값을 오른쪽 가지에는 큰 값들만 있도록 구성된 트리입니다. 자식노드들도 마찬가지의 룰을 가지고 있어 규칙성을 가지고 있는 트리입니다. 

이진탐색트리를 concurrency하게 접근하게 해주는 것이 이번 과제의 목표이다. 코딩의 순서를 우선 이진탐색트리의 삽입과 삭제에 관한 코드를 선결적으로 작성한 후 mutex lock과 unlock을 이용해 멀티스레드에서 동일성을 지키게 만들어주는 식으로 코딩을 진행했습니다.

### 이진탐색트리 구성

삽입함수는 현재 노드를 가리키는 변수와 현재 노드의 부모노드를 가리키는 두 개의 변수를 통해 삽입부분의 부모노드를 탐색하여 삽입 노드의 크기를 비교해 올바른 위치에 값을 삽입해주도록 함수를 작성했습니다.

삭제함수는 삭제를 할 값을 탐색하는 함수를 만들어 확인하고 자식노드의 개수에 따라 자식노드가0인 경우에는 해당노드를 삭제하고, 자식노드가1인 경우에는 삭제 후 그 위치에 자식노드를 삽입, 자식노드가2인 경우에는 삭제하는 값의 왼쪽 트리의 큰 값을 넣어주는 방법으로 작성했습니다.

이진탐색트리의 삽입, 삭제 함수를 구현한 후 트리전체에 락을 걸어주는 coarse-grained lock과 노드 당 개별로 락을 걸어주는 fine-grained lock 두 가지 방식으로 mutual알고리즘을 적용시켰습니다.

![image](https://user-images.githubusercontent.com/26988563/160964710-f7bb49a9-e8c1-413a-9dfe-4d93252c0292.png)

위 코드와 같이 삽입함수에 전체 락을 적용했습니다. 노드 구조체의 데이터 변수인 mutex를 인자로 받은 pthread_mutex_lock함수를 통해 락을 걸어주었고 스레드 생성과 동시에 원자성을 보장해 줄 수 있었고 생성이 끝나며 pthread_mutex_unlock함수로 락을 풀어주었습니다.

![image](https://user-images.githubusercontent.com/26988563/160964715-10267e19-380d-4a29-a8d0-847a5a1840cb.png)

다음으로 위 코드와 같이 삽입함수에 노드별로 락을 적용해봤습니다. fine-grained의 특성에 따라 임계영역을 쪼개서 각각 락을 적용시켜주었습니다. 그 후에 인자로 받은 노드값을 비교하는 모든 조건문에서 스레드가 원자성을 가지게 해주는 노드생성함수의 일부를 위 사진에서 볼 수 있습니다.

![image](https://user-images.githubusercontent.com/26988563/160964725-82056b00-b470-44b0-8fdc-4a6d84bdde50.png)

삽입함수의 두 가지 락을 적용이 끝나고 삭제함수에 두 가지 락을 적용시켜봤습니다. 위 사진은 삭제함수에 전체 락을 걸어준 코드입니다. 현 노드의 자식유무를 판단하는 if문의 앞뒤에 락을 걸어주었고 이렇게 함으로써 스레드들의 원자성을 지켜줄 수 있게 되었습니다.
  삭제함수에 부분 락을 걸어줄 때는 공유자원에 접근하는 임계영역에 락을 걸어주었습니다.
  
### 주요 소스코드

* 헤더

![image](https://user-images.githubusercontent.com/26988563/160964729-e10a0c8a-6c8a-4863-b0ee-d5dcc2eb37c7.png)

* 이진탐색트리 삽입탐색

![image](https://user-images.githubusercontent.com/26988563/160964731-e7d31aa4-6346-4ae9-96fe-a614edad046b.png)

* 트리생성

![image](https://user-images.githubusercontent.com/26988563/160964738-35bfb175-9826-4dc6-9057-e172f7b60b9e.png)

* 노드생성

![image](https://user-images.githubusercontent.com/26988563/160964746-ab1e41ee-c752-4852-90e7-1cc4e0bb3e97.png)

* 노드삽입(single thread)

![image](https://user-images.githubusercontent.com/26988563/160964752-eb6da785-4bd9-47ee-9e01-5c6fe21aaa34.png)

* 노드삽입(fine-grained)

![image](https://user-images.githubusercontent.com/26988563/160964758-e4679b13-db5f-4c8e-989a-5bb0090e7107.png)

* 노드삽입(coarse-grained)

![image](https://user-images.githubusercontent.com/26988563/160964761-3ea53de0-38db-47fa-94ce-a6bbdbb9dc4d.png)

* 노드삭제(single thread)

![image](https://user-images.githubusercontent.com/26988563/160964769-c037766f-1d3f-48d1-9041-e1392907507e.png)

* 노드삭제(fine-grained) 임계영역부분마다 락

![image](https://user-images.githubusercontent.com/26988563/160964777-1c146f5d-77c3-44f4-8e88-970748e9bbc4.png)

* 노드삭제(coarse-grained) 조건문 앞뒤에 락

![image](https://user-images.githubusercontent.com/26988563/160964786-db868112-9e6f-4de5-8803-8f6482450874.png)

### 결과값

![image](https://user-images.githubusercontent.com/26988563/160964797-ab54244a-8dac-44fb-8ea1-6329cf03ed89.png)

### 예상 결과값과 실제 결과값, 결과의 이유 예측

우선 실행시키기 전에 예상값은 mutex적용이 적용하지 않은 스레드보다 실행시간이 무조건 짧을 것이다. 또한 큰 락을 걸어주는 것은 작은 락을 걸어주는 것보다 트레이드오프관점에서 오버헤드가 적긴 하지만 화장실 전체를 잠근 것처럼 하나의 스레드가 작동하는 동안 나머지 스레드는 놀고 있으므로 속도가 느려지고 작은 락은 오버헤드가 커지지만 그만큼 스레드들이 자원을 효율적으로 사용할 수 있다는 생각을 기반으로 스레드가 4개밖에 없으므로 오버헤드가 차지하는 시간보다 자원을 효율적으로 사용함으로써 얻을 수 있는 이득이 크다고 생각하여 작은 락이 스레드 4개에서는 실행시간이 짧을 것이라고 예상했습니다.

예상 결과 값

fine-grained > coarse-grained > single thread순으로 실행시간이 짧을 것이다.

실제 결과 값으로는

insert :: single thread=1.086066, coarse-grained=0.979328, fine-grained=1.273821로 

coarse-grained > single thread > fine-grained이라는 결과가 나왔고

delete :: single thread=0.754380, coarse-grained=0.750070, fine-grained=0.543745로

fine-grained > coarse-grained > single thread라는 결과가 나왔습니다.

이에 따라 삭제함수에서는 예상대로의 결과가 나왔지만 어째서인지 삽입함수에서는 부분락이 제일 느리게 나오는 결과가 나왔습니다. 저희 조는 거기에 따른 추론으로 삭제함수는 여러번 실행해도 예상값이 나오니 제쳐두고 삽입함수에서 삭제함수와 또, 예상값과 다르게 나오는 이유에 대한 가설을 세워봤습니다. 

첫째, 삽입함수는 삭제함수에 비해 덜 복잡하므로 락의 영향을 덜 받는다. 그러니 따라서 락을 많이 걸어줄수록 오히려 더 느려지는 결과가 도출될 수 있다.

둘째, 그냥 모평균 집단이 작은 것도 원인이 될 수 있다. 첫 번째 가설도 그럴 듯 하다 생각했지만 삭제함수와 비교를 한 가설이라 만약 삭제함수가 없었다면 삭제함수를 기준으로 잡지 못할 것이고 우리는 삽입함수를 기준으로 결론을 내릴 수도 있었을 것이다.

셋째, 삽입과 삭제가 락에 영향을 크게 받는 어떤 기준점을 두고 대치하는 것이 아니라 두 함수 모두 락에 영향을 적게 받고 우연히 나온 결과가 근소한 차이로 예상값에 맞게 되어 맞는 결과라 착각하는 걸 수도 있다.

이렇게 예상값이 무조건 맞다고 생각하지 않고 예상값, 삽입함수결과, 삭제함수결과, 모두 의심을 해보며 여러 가지 가설을 세워봤습니다. 우선 여러번 실행해본 결과 순위가 뒤집히는 경우도 있었기 때문에 두 번째 가설 모평균집단이 작다는 것과 또 세 번째 가설 애초에 함수자체가 락에 영향을 적게 받아 외적인 환경, 네트워크환경이나 가상머신이 설치된 cpu환경에 오히려 더 영향을 많이 받게되어 항상 결과값이 동일하지는 않다는 결론을 내렸습니다. 하지만 여기서 또 의문인 것은 삭제함수는 대체로 예상값을 잘 따랐습니다. 그리하여 첫 번째 내린 결론에 이어 실행시간을 보았을 때 삭제함수는 삽입함수보다 수행할 작업이 적지만 작업크기에 상관없이 락에 더 영향을 많이 받는 수행을 한다는 결론을 도출하게 되었습니다.













