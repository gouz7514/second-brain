## AWS Region
- 데이터 센터가 모여 있는 곳
- 그렇다면 어떤 기준으로 region을 선택하는가?
	- 규정 준수 : data never leaves a region without your explicit permission
	- Latency : 고객에게 최대한의 접근성을 제공해야 한다 (ex : region은 호주, 유저는 미국이면 latency가 많이 발생)
	- 모든 region이 모든 서비스를 제공하지는 않는다. 배포하려는 region이 서비스를 지원하는지 확인
	- 가격 : region마다 가격이 다를 수 있다

### AWS Availzbility Zones
- region에 포함되는 구성 요소. 일반적으로는 3개
- 각각의 AZ는 하나 이상의 독립적인 데이터 센터로 구성됨
- 각각의 AZ는 재해로부터 안전하도록 분리되어 있음

AWS는 또한 400개가 넘는 Points of Presence를 제공 (후에 다시 언급)