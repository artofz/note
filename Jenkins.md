# Jenkins를 이용한 CI/CD

## Docker를 이용한 젠킨스 설치
```bash
docker pull jenkins/jenkins:lts-jdk17
docker run -d -p 8080:8080 -p 50000:50000 --restart=on-failure --name jenkins-server -v /home/ec2-user/jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk17
```

localhost:8080으로 접속하면 비밀번호를 입력하라고 나오는데 도커 컨테이너 로그에 찍힌 비밀번호를 입력해주면 된다.
`docker logs jenkins_home` 명령어를 통해 로그 확인가능하다.

![젠킨스초기패스워드](/젠킨스_초기패스워드.png)
- 다음 화면에서 플러그인은 Install suggested plugins로 설치한다.

## 준비작업
### KEY 생성
젠킨스가 설치된 서버에서 원격 서버에 자동으로 접속하기 위해 `SSH` 키를 생성합니다. 
```
ssh-keygen -t rsa -b 4096 -C "arto_jenkins"
```
- RSA 알고리즘을 4096비트 길이의 키를 생성. -C를 통해 주석을 달 수 있다.

### 원격서버에 공개키 등록해주기
원격서버에 내가 생성한 키가 안전한 사용자라는 것을 알려주기 위한 작업이 필요합니다. 원격 서버에서 `.ssh/authorized_keys`에 접속한 뒤 공개키(.pub로 끝나는 파일)의 값을 복사/붙여넣기 해줍시다.



### Jenkins Credentials 등록하기
> Jenkins에서 사용되는 인증 정보를 안전하게 관리할 수 있도록 도와주는 기능으로 자격 증명 정보가 젠킨스 인스턴스에 암호화된 형태로 저장되며, 해당 자격 증명 ID를 통해서만 Pipeline 프로젝트에서 처리된다. 

원격 서버에 접속하기 위한 계정을 등록하는 과정입니다.

- ID: 추후 이 username을 이용해 Credential 정보를 사용합니다.
- Username: 원격 서버의 계정 이름
- Private Key 등록하기
- Passphrase: 필수 아님 Key-Gen 명령어로 키를 생성할 때 passphrase설정을 했으면 등록해주면 된다.
- Global - 파이프라인 프로젝트에 사용되는 경우
- System - 이메일 인증, 상담원 연결 등과 같은 시스템 관리 기능과 상호 작용하기 위한 Jenkins 인스턴스 자체인 경우


등록한 정보를 파이프라인에서 사용하기 위해 SSH-Agent플러그인을 설치해야 한다.  

### SSH-Agent 설치하기
> Jenkins 빌드에서 빌드할 때 SSH 키를 사용할 수 있도록 도와주는 툴.

## 테스트
새로운 Pipeline 프로젝트를 생성합니다. 
```
pipeline {
    agent any
    stages {
        stage('deploy') {
            steps {
                sshagent(credentials: ['jenkins_key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no {{ 계정이름 }}@{{ IP주소 }} '
                        ls
                        '
                    """
                }
            }
        }   
    }
}
```


Jenkins Credentials 이란? 
- Jenkins에서 사용되는 인증 정보를 안전하게 관리하도록 도와주는 기능.
- Jenkins에서 사용되는 비밀번호, API토큰, SSH키 등의 민감한 정보를 안전하게 저장하고 관리할 수 있다.
- How it works?
- 애플리케이션에서 Jenkins 전용으로 사용하도록 자격 증명을 구성할 수 있다.
- Jenkins에서 구성된 자격 증명은 컨트롤러 Jenkins 인스턴스에 암호화된 형태로 저장되며(Jenkins 인스턴스 ID로 암호화됨), 해당 자격 증명 ID를 통해서만 Pipeline 프로젝트에서 처리된다.
- `Manage Jenkins(젠킨스 관리)` -> `Seucirty-Credentials` -> Under Stores scoped to Jenkins `Click the Jenkins`
- From the Scope field, choose either:
      




[참고](https://www.jenkins.io/doc/book/security/credentials/)
[젠킨스 CI/CD 구축 NGINX 참고](https://nginxstore.com/blog/microservices/jenkins-%EB%A5%BC-%ED%86%B5%ED%95%B4-github-ci-cd-pipeline-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0/)