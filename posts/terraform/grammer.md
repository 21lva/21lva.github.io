
# Terraform HCL 기본 내용

- 여기서는 테라폼의 코드 기본적인 구조, 문법, hcl 문법 등에 대해서 다룬다


## Resource

- 테라폼의 인프라를 정의
```
resource "label" "name" {
	key = value
}
//예시
resource "aws_instance" "web" { 
	ami = "ami-a1b2c3d4" 
	instance_type = "t2.micro" 
}
```

- - -
## Data

- 테라폼 외부 혹은 테라폼 다른 정보들을 조합하여 새로운 데이터를 만들때 사용.
- provider가 제공하는 리소스 정보를 가공해서 테라폼에서 사용할 수 있는 정보로 변환한다

```
data "NAME" "name" {
  //처리
}

//예시
data "aws_ami" "example" { 
	most_recent = true 
 
	owners = ["self"] 
 
	tags = { 
		Name = "app-server" 
		Tested = "true" 
	} 
}
```

- 위 예시 코드는 "aws_ami"에서 "example"로 변환한다. *data.aws_ami.example*로 접근해서 사용 가능.
	- *most_recent, owners, tags* 등은 query의 조건문으로 사용된다.

- - -
## 변수(variable)

- 변수는 테라폼에서 리소스의 입력 값을 전달할 때 사용

```tf
variable <NAME> {
	description = "설명"
	type = list | map | object | bool | string | set
	default = "default value"	 
}
```

- 변수를 초기화하는 방법은 2가지가 있다
	1. *terraform plan* 명령어로 대화형으로 값 지정
	2. 환경변수 설정: *TF_VAR_변수_이름*
	3. 명령어 인자로 전달: *terraform plan -var "변수이름=변수값"*
	4. default 값 지정

- 변수 값은 리소스에서 참조로 사용 가능
```
resource "resource_type" "resource_name" {
	content = var.변수이름
}
```

- 다른 모듈에 있는 variable은 접근 불가능하다. 
	- 다른 모듈과의 상호작용은 다른 모두의 변수값에 값을 지정해주거나 해당 모둘의 output 값을 이용하는 것만 가능하다. 

- - -

## output

- 코드가 인프라에 반영된 이후에 반영된 세부사항을 다른 테라폼 코드에 저장
- 모듈이 외부에 값을 전달할 때도 사용된다

```
output <output_name> {
	value = "output value"
	description = "설명"
}
```

- - -

## local value

- 동일한 값을 모듈내의 여러곳에서 사용하는 경우에 사용된다
- variable과의 차이는 모듈의 입력으로 전달받을 수 없어서 외부에 노출하고 싶지 않은 값을 정의할 때 사용된다. 

```
locals {
	key = value
}

// 사용
resource "a" "b" {
	a = local.key
}
```


- - -

## Module

- 테라폼은 코드를 재사용 가능한 모듈로 정의한다
- 하나의 모듈은 폴더를 의미한다. 모듈내에는 서브 모듈이 존재 가능하다. 그러나 보통은 flat한 구조로 만드는 것이 권장된다
- 여러 모듈을 사용한다면 *terraform init* 명령어로 모듈을 초기화 해야한다.

```
module "NAME" {
	// count = 조건 ? 1 : 0 과 같인 meta-argument 사용 가능
 
	source = "모듈 경로"

	 // module의 variable에 값을 넣어줄 수 있다.
	input_key1 = input_value1
	input_key2 = input_value2
}
```

- 모듈을 사용할 때 해당 모듈의 output 을 이용할 수 있다

```
resource "a" "b" {
	bucket_name = module.bucket.name
}
```

- 모듈 경로는 상대경로로 접근한다. 특히, 모듈의 파일에 접근할 때는 path에서 제공하는 함수들을 사용한다
	- path.module: module이 위치한 경로
	- path.root: root module의 경로
	- path.cwd: 현재 작업중인 모듈의 경로
- 모듈내의 요소들(resource, data, variable)은 모두 모듈 외에서는 접근 불가능하다. 오직 output만이 접근 가능하다. 

- 다른 모듈에서 사용할 목적만 갖고 있는 모듈의 경우에는 *provider.tf*를 제외해야 한다. provider는 인프라로 배포될 모듈에서 사용한다.

```
$ tree
beta/vpc/main.tf
beta/vpc/version.tf
beta/vpc/provider.tf
modules/vpc/main.tf
modules/vpc/variables.tf
modules/vpc/version.tf
modules/vpc/outputs.tf
```
- - -
## reference

- https://rahullokurte.com/concept-of-terraform-module-and-module-scope
- https://malwareanalysis.tistory.com/category/%EC%97%B0%EC%9E%AC%EC%99%84%EB%A3%8C/terraform
- https://velog.io/@gentledev10/terraform-data-source
- https://developer.hashicorp.com/terraform/language