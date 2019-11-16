# DAY2_HyperLedger

## 필요 소프트웨어
- docker : 서버 설치
- docker-compose : 서버 설치
- golang : 서버 설치
- VSCode(IDE) <br> : 
: https://insights.stackoverflow.com/survey/2019
- (optional)git

<br><br>

## 환경설정
### **1. docker 설치(linux)**
- 용도 : fabirc network 구축
>https://docs.docker.com/install/linux/docker-ce/ubuntu/ 참조
```
$) sudo apt-get update
$) sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
$) curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$) sudo apt-key fingerprint 0EBFCD88
$) sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   //repository 추가
$) sudo apt-get update
$) sudo apt-get install docker-ce containerd.io
$) sudo docker ps
$) sudo docker run hello-world
```

<br>

> docker 예제(MySQL)
```
$) sudo apt install mysql-install
$) sudo docker run -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true —name mysql mysql:5.7
$) mysql -h 127.0.0.1 -uroot
$) show database;
```

```
$) mysql -h 127.0.0.1 -uroot
```
<br>

> docker 권한 주기(일반 사용자)
```
$) sudo usermod -aG docker $(whoami)
```
<br><br>


### 2. docker-compose
> https://docs.docker.com/compose/install/
```
$) sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$) sudo chmod +x /usr/local/bin/docker-compose
$) sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
$) docker-compose --version

```

<br><br>

### **3. VisualStudio Code**
**이번 Lab 에서는 아래 명령어를 주로 사용할겁니다**
- Show all command = Ctrl + Shift + P
- File Quick Open = Ctrl + P

<br>

> vsCode Extension 설치하기
- SSH FS <br>
: Host - 192.168.56.10 <br>
: UserName - test <br>
: Password - testpw <br>
: Root - /home/test<br>

- Go

<br><br>

### 4. golang

#### **golang 다운로드**
```
$) curl -O https://dl.google.com/go/go1.12.5.linux-amd64.tar.gz
```

<br>

#### **압축풀기**
```
$) tar -xvzf go1.12.5.linux-amd64.tar.gz
```

<br>

#### **파일 경로 변경**
```
$) mv go /usr/local/
```
<br>

#### **GOPATH 설정하기**<br>
```
$) sudo  vi ~/.bash_profile
export PATH=$PATH:/usr/local/go/bin
```

<br>

#### **go 설치 확인하기**<br>
```
$) sudo go version
```
<br><br>

### (optional) 5.git
**이번 Lab 에서는 아래 명령어를 주로 사용할겁니다**
- git commit = 로컬(master) 저장<br>
- git push = 원격(origin) 저장<br>
- git config --global user.email "email@example.com"
<br><br>
