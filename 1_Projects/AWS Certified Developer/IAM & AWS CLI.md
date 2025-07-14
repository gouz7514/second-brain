### Users, Group
- IAM = Identity and Access Management, Global Service
- **루트 계정**은 기본으로 생성되며, 계정 생성 이외에는 사용되거나 공유되면 안됨
- 대신 **Users**를 생성해야 함.
- Users를 묶는 개념이 **Groups**
- 꼭 group에 속할 필요는 없으며, 하나의 user가 여러 개의 group에 속할 수 있음

Q : 왜 사용자와 그룹을 생성하는가?
A : permission을 부여하기 위해서 (JSON 형태로 지정됨)

- 그룹에 속한 사용자는 그룹의 정책을 상속받고, 이와 별개로 inlnie 정책을 부여받을 수 있다 ([관련 링크])

### IAM Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
	      "AWS": ["arn:aws:iam:123456:root"]
      },
      "Action": [
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::example-bucket/*"
      ]
    }
  ]
}
```
- Version : 정책 문서의 버전
- Id (optional) : 정책을 식별하는 ID
- Statement : 정책에서 하나의 규칙
	- Sid : 문장 ID
	- Effect : 특정 api에 접근하는 걸 허용/거부할지
	- Principal :특정 정책이 적용될 사용자, 계정, 혹은 역할
	- Action : 허용/거부할 api 의 목록
	- Resource : 적용할 AWS 리소스
	- Condition (optional) : Statement가 언제 적용될지를 결정
