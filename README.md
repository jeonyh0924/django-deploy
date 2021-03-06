# [AWS](https://www.slideshare.net/awskorea/your-first-10-million-users-channy)
인프라 서비스

개개인이 시도하는 서비스는 99% 망한다.
인프라는 비싸고 시작부터 체계적으로 준비를 하기에는 소요되는 시간과 자원이 비싸다. -> 리스크가 크다.

- scale up >> 성능 올리기 : 무조건 성능을 올릴 시 한번은 꺼야 한다.(다운타임 발생)
- scale out >> 개수 늘리기

대부분 scale out 하다가 적절히 scale up도 같이 한다. 

AWS의 핵심은 비지니스 로직에 집중 할 수 있다는 것이 가장 큰 장점이다. 

### 엣지 - static cache - CDN


### ec2
메뉴얼적인 서버 한 개 이다.
가장 기본이 되는 시작 

- AMI(머신 이미지) : EC2에 올라간 값들에 대한 정보를 카피 하고 새로운 EC2를 생성 할 때 기존에 이미 올라와 있는 EC2에 대한 정보를 그대로 사용하여 생성한다. 
- 스냅샷 :


- 한 서버에 너무 많은 기능을 추가하면 부하가 너무 크다 ( DJango, redis, postgres 등등..)
	- 해법은 기능에 따라 인스턴스를 나누는 것 (웹 서버용 인스턴스, 디비용 인스턴스)

	
### ELB
elb를 어떤 존에 넣을 것이냐 -- 대부분 다 넣는다. 

웹서버는 무한정 늘릴 수 있다.
하지만 write하는 마스터 디비는 한개까지만, read 하는 디비는 5~7개까지밖에 늘릴 수 없다.
마스터 디비에 대해서 스케일 업 하는 순간이 서버점검을 할 수 밖에 없는 순간이다. 
마스터 디비가 상태가 안좋을 경우 바로 다른. read replica 디비가 새로운 마스터가 되어야 한ㄷ. 

write의 자원 소요가 read보다 훨씬 더하다 

--> SSL을 쓰기 위해서는 로드 벨런서를 써야 한다. 

## P38
Master 는 20%의 write에 대한 자원을 받고,
read는 80% 에 대한 작업이다 .
- 데이터를 디스크에 써야 하기 떄문에 , 인덱스를 만드는 과정이 헤비한 과정이다 ->> write가 헤비한 이유 

## P41
서버는 장애가 나게 되면 라이프 사이클에 상관 없이 바로 죽이기 떄문에
서버 안에 데이터를 넣으면 안된다.
영속성적인 데이터를 주지 않아야 한다. 

Gunicon(자바는 톰캣)//랭귀지를 서빙. - Nginx(자바는 아파치) -- 이미지, 스태틱한 데이터를 서빙.

##### s3 주의사항
- 한 폴더 안에 넣을 수 있는 파일에 대한 관리가 필요하다( 한 폴더 안에들어가는 한계도 있고, 한 폴더안 너무 많은 파일이 관리되면 성능측에서 좋지 않다.)
- 단점: AWS 리전이 약 11개 인데, S3는 한 지역에 의존성이 있기 때문에 타 리전에서는 내 s3에 대한 접근을 하기 위해서는 내 오리진 서버에 들어가야 하기 때문에 느리다. : 해결 방법 == 근처 엣지서버에서 원하는 데이터를 찾고 없으면 오리진 서버에서 가져 온 뒤 엣지 서버에 넣는다. TTL 도 줄 수 있으며 스태틱 파일들은 TTL을 길게 준다. (aws cloud front 라는 서비스 이다. CDN (캐시 서버), 동적 데이터도 캐시)


- collect static - s3에 스태틱 파일 업로드
- whitenose라는 라이브러리를 사용하면 관리자페이지 만큼은 gunicon에서 작업을 하도록 한다. 


## P44
- 세션 : session time out // 영속적이지 않은 삭제가 되어도 ttl 의 단위가 낮아도 되는 데이터들 >> redis기반의 메모리 안에 저장이 되는 것이 좋다. read가 엄청 많다. (토큰이 유효한지 확인하기 위해서)

- session, cache, ttl등을 사용을 할 것이면 elasticCache(redis, memcache)를 사용한다. 
- DynamoDB가 밖에 있는 이유 : 리전이 없다. 글로벌 리전이며 전세계 어디에서 접근하여도 비슷한 레이턴시를 제공한다. 
- 강의에서 사용하던 인프라들을 aws를 사용하여 다 분리를 할 것이다. 
- 다음 날 수요가 확 늘더라도 동적으로 빠른 대응을 할 수 있다.
- AWS가 비쌈에도 사용하는 이유는 서버라는건 끊임없이 관리를 해 주어야 하는 주체인데, 관리에 대한 자원을 줄이면서 비즈니스에 집중을 할 수 있다는것이 큰 장점이다. 


### 오토 스케일링
- 트래픽을 받고 있는 상황에서 다운타임을 
- ELB, Ec2, condition을 설정한다. 
- condition 은 cpu 가 80%가 넘는 클럭이 3번 이상 발생하면 2대 증설 
- 기준점이 되는 대상을 cpu, diskIO, networkIO, 등에 있어서 여러 조건에서 걸 수 있다. 
- 반대로 특정 조건 속에서 서버를 줄일 수 있다. (cpu가 30% 5분동안 2번 유지 되면 서버 하나 감소 등)


#### 오토스케일링처럼 대비를 하여도 서버가 터지는 이유
- 오토스케일링을 하는 것도 시간이 걸리는데 그 시간조차도 버틸 수 없기 때문에
- 인력이 미리 수동으로 미니멈 갯수를 올려버리는 등의 대응 방법이 있따.
- 한국의 대부분의 서비스들은 클라우드 서비스에 대해서 수동적이다.


### VPN 프라이빗 네트워크
- elastic cache - 인 메모리 기 떄문에 휘발성으로 다 날라간다. 
- DanymoDB는 비 휘발성 이라 둘 다 사용한다, 비 휘발성

### 모니터링과 로깅
- 찰나의 실수에 대한 원인을 찾기 위해 모니터링 툴과, 로깅을 찍어야 한다. 
- 모니터링을 할 수단은 많을수록 좋다. 

### 무중단 배포를 처리하는 방법을 제공해주는 서비스가 있다.
code deploy- 다운타임 없이 서버들을 퍼센트, 또는 순회하며 서버를 연결을 끊고 리뉴어 서버 구동, 다시 연결하는 방식들이 있다. 



### rabbimQ
- 용량문제(100메가)를 넘어가게 되면 예전의 데이터가 날라가는 상황이 있다. 


### 59P
- 워커 인스턴스는 로드벨런서와 연결이 되어있지 않다. 이 워커 인스턴스는 웹 인스턴스에서 SQS에 보낸 큐 안에서 일을 가져온다. 

- async 작업의 힘든 점은 디버깅이 힘들다는 것이다. 그래서 sentry, logdna 가 필요 하다. 




### 프로비저닝
배포 자동화 툴


### private subnet
사설 ip(172, 192) 와 공공 ip 물리적 차이 


### DB 부하 (p63 정말 극 소수의 회사에 한해서 )
- 마스터 디비는 하나만 존재한다. 마스터에 부하가 많이 가게 되면 결국 뻗어진다. 
- 데이터베이스를 분리해야 한다. (sharding)
- 데이터베이스 FK가 되지 않아서 쿼리가 어려워진다. 
- 제대로 하기 굉장히 어렵다 



### 병목은 항상 디비에서 발생한다. 
- 이유는 Network, Disk IO에 의해 발생한다. 그 이후에 신경을 써야 하는것이 캐시
- 리스폰스 자체를 캐시하는 방법도 있다. (json 자체로 )

#### 첫날 사용자 한 명 
```shell
ssh -i "fc-wps-13-deploy.pem" ec2-user@ec2-15-164-80-228.ap-northeast-2.compute.amazonaws.com

sudo amazon-linux-extras install nginx1.12
sudo yum install python3
sudo yum install telnet
- sudo systemctl start nginx
- sudo systemctl status nginx
- sudo systemctl stop nginx

- sudo yum install git
- git clone https://github.com/jeonyh0924/django-deploy.git
- sudo pip3 install -r requirements.txt
- pip3 list 
```


```
# 퍼미션 에러
The authenticity of host 'github.com (15.164.81.167)' can't be established.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,15.164.81.167' (RSA) to the list of known hosts.
Permission denied (publickey).
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.

해결 방법 -깃에서 ssh말고 https로 받아옴.

# django.core.exceptions.ImproperlyConfigured: SQLite 3.8.3 or later is required (found 3.7.17).
로컬 환경과 다른 에러가 발생한다. 
sqlite가 없다고 에러가 나는 것 이다. 
서버환경과 배포 환경은 미묘하게 항상 다르다. 
인프라 설정을 바꿀 때 이러한 것에 집중하여 처리를 해야 하는 것 . 
로컬에서 도커환경을 구축 한 뒤 잘 동작이 된다면
서버 위에서도 정상적으로 동작하기 때문에
도커가 각광받게 되었다. 
```

- 표준생성, 프리티어
1. <인스턴스 식별자> : fc-wps-13-RDS
2. <마스터 사용자 이름>/<암호> : root/<password>
3. 스토리지 는 하드웨어 / 스토리지 자동 조정 활성화
4. 퍼블릭 액세스 가능정보 열고 , 보안그룹으로 특정 값에 대해서 방어
5. 자동 백업은 프로덕션 단계에서는 필수지만 지금은 사용하지 않겠다. 


- 생성 된 RDS로 연결 

0810
2 : 도커컴포즈를 통해서 디비 연결 
3 : 업데이트를 한 후 깃 푸쉬
4 : AWS 서버에서, git pull https://github.com/jeonyh0924/django-deploy.git를 하여 최신화를 지속한다. 


 python3 manage.py runserver --settings=config.settings.staging 0.0.0.0:80
 0.0.0.0으로 바꾸어야 외부에서 접속이 가능하다. 
 80번 포트를 열기 위해서는 루트를 통해서 열어야 한다. 그래서 sudo 를 앞에 붙인다. 

gunicorn 은 기본적으로 static file을 서빙하지 않는다. 

[whiteNoise](http://whitenoise.evans.io/en/stable/)

#### 런서버의 성능이 좋지 않아서 퍼포먼스적인 이유로 gunicorn을 쓴다. 
sudo /usr/local/bin/gunicorn config.wsgi -b 0.0.0.0:80 --access-logfile - -e DJANGO_SETTINGS_MODULE=config.settings.staging




1. git 레포 생성
2. local runserver
3. git push
4. git pull to ec2
5. local docker compose


```shell
# 프로세스 중에 80번 포트에 해당하는 값들을 찾아본다. 
- ps -efc | grep -i 0.0.0.0:80
# 해당 포트 삭제
- sudo kill -9 4519
```




08 11 

- ec2 생성 / 서브넷 A 
- elastic IP 생성 - 연결
- git repo 생성, 로컬 실행.
- local db postgres 연결, 실행
	- environ 실행
	- 

- sudo yum install python3
- sudo yum install git
- 