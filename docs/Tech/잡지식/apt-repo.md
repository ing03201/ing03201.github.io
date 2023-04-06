---
layout: page
title: reprerpo로 나만의 Ubuntu/Debian APT 저장소를 만들어보자!
parent: 잡지식
grand_parent: Tech
permalink: /Tech/잡지식/Ubuntu-repo
---

# 나만의 Ubuntu/Debian APT 저장소를 만들어보자!

	Debian Project에서 나와 Debian 계열 운영체제가 채택하는 패키징 툴이다.
	리눅스를 처음 입문하는 사람들도 익숙할 것이다.
	현재 회사에서 라이브러리들이 중구난방에 소스코드 리포지토리에 업로드하는 등
	너무 난잡하게 있기에 업데이트서버를 구축하면서 같이 작업하게 되었다.

# 1. GPG key 생성


아래 명령어를 실행해 gpg key를 생성한다

```Shell
gpg --full-gen-key
```

만일 gpg command not found 에러가 뜨게 되면 gnupg 패키지를 설치해주자

```Shell
sudo apt-get install gnupg rng-tools -y
```

위 명령어를 실행하면 다음과 같이 나오게 된다.

```Shell
$ gpg --full-gen-key
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Kim Yoonsoo
Email address: ing03201@gmail.com
Comment: To make Kim yoonsoo repository
You selected this USER-ID:
    "Kim Yoonsoo (To make Kim yoonsoo repository) <ing03201@gmail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key F02E0BB264430FAE marked as ultimately trusted
gpg: revocation certificate stored as '/home/kim/.gnupg/openpgp-revocs.d/18D11101B3935D116505C69BF02E0BB264430FAE.rev'
public and secret key created and signed.

pub   rsa4096 2023-04-06 [SC]
      18D11101B3935D116505C69BF02E0BB264430FAE
uid                      Kim Yoonsoo (To make Kim yoonsoo repository) <ing03201@gmail.com>
sub   rsa4096 2023-04-06 [E]
```

  gpg: key <span style="background-color:#ffdce0;"> F02E0BB264430FAE </span> marked as ultimately trusted  <- 형광색 부분이 Key Id 이다. 

# 2. reprepro 설치 및 디렉토리 구성

 우선 repository를 만들어줄 reprepro 부터 설치한다
 
```Shell
sudo apt-get install reprepro -y
```

이 포스트에서는 ./apt/ 에 모두 구성할 예정이다.

```Shell
mkdir -p apt/incoming
mkdir -p apt/conf
mkdir -p apt/key
cd apt/
```

# 3. 키 내보내기 및 리포지토리 생성

이후 아까 만들어 놓았던 gpg key를 key folder에 넣는다  

```Shell
gpg --armor --export F02E0BB264430FAE >> /var/www/apt/key/deb.gpg.key
```
이제 리포지토리 배포 설정파일(conf/distributions)을 만들어야한다  
```shell
vim conf/distribution
```
distribution 내용은 이렇게 사용한다  
```
Origin: (**yourname**)
Label: (**name of repository**)
Suite: (**stable or unstable**)
Codename: (**the codename for the distribution you are using, like trusty**)
Version: (**the version for the distribution you are using, like 14.04**)
Architectures: (**the repository packages  architecture, like i386 or amd64**)
Components: (**main restricted universe multiverse**)
Description: (**Some information about the repository**)
SignWith: yes
```

여기서 OS 코드네임 별로 구분하고싶다면 코드네임은 복수를 지원하지 않기때문에 
위 내용을 그대로 하나 더 복사해서 붙이면 된다.
컴포넌트와 architecture는 띄어쓰기로 복수 컴포넌트를 정의할수있다.  
	* 리포지토리 어떤 행동마다 key 비밀번호를 넣는 옵션을 넣고싶을 때 *
	conf/options 파일을 만든다.  파일 내용은 다음과 같다

```
ask-passphrase
```

리포지토리를 생성한다  
```Shell
 reprepro --ask-passphrase -Vb . export
```

# 4. 생성된 리포지토리에 패키지 추가

저장소에 추가할 deb 패키지를 준비한다.   
deb 패키지를 만드는 방법은 다음 포스트에 작성하도록 하겠다.


```Shell
$ reprepro -Vb  apt folder url  include  codename   package url 
```

근데 가끔 해당 명령어가 안먹힐때가 있다. 보통 System-Essential 하지 않은 패키지들때문에 그런것으로 알고있다.
이때 Suite에 Utils 추가해주면 해결되는 경우가 있다.
나는 다중 컴포넌트(main  preview test)를 사용했기 때문에 컴포넌트 사용법도 추가한다.

```shell
reprepro -Vb  apt_folder_url  -S utils -C  component name  includedeb  codename   package_url 
```

# 5. 리포지토리 업로드

정적 호스팅 서비스를 활용하면 바로 적용할수 있다.
나는 nginx 호스팅을 해보았다.  
 
```Shell
cd .. # apt parent directory
sudo cp -r apt/ /var/www/apt/
```

이런식으로 하게 되면 같은 네트워크를 쓰는 장비에서 연결이 가능하다.

업데이트 받아야하는 컴퓨터에서 실행해야하는 명령어이다.

```Shell
sudo echo "deb your_repo_PC_url codename component " > /etc/apt/sources.list.d/my.list && \
wget -O -  your_repo_PC_url /key/deb.gpg.key | apt-key add -
```

혹시나 apt-key 에서 에러가 난다면 gnupg 패키지가 없는 것이다.

```shell
sudo apt-get install gnupg -y
```


또한 정적 호스팅 되어있는 AWS S3 Bucket에도  sync 하면 바로 적용이 된다. 
이때는 url을 버킷에 접근 가능한 access key와 secret key를 url에 넣거나
apt-transport-s3 패키지를 설치해야한다
