---
layout: post
title: AWS EC2, GCP VM Instance에 SSH로 접속하기 with Ubuntu
date: 2020-07-22 01:00:00 am
update: 2020-08-20 10:00:00 am
permalink: posts/93
description: AWS EC2, GCP VM Instance에 SSH로 접속하는 법을 간단히 정리한다.
categories: [Dev, DevOps]
tags: [ssh, AWS, GCP]
---

> AWS EC2, GCP VM Instance에 SSH로 접속하는 법을 간단히 정리한다.

    주의

이 방법은 보안이슈가 생길 수 있기 때문에 실무환경보다는 테스트환경에서 사용하기 쉬운 방법이다.

## AWS EC2에 접속

### 0. AWS Private Key 사용하는 법

먼저, EC2 생성 시에 Key pair를 생성하고 Private Key를 발급받는다. Security Group(보안그룹) 설정 시 IP를 원하는 대로 설정한다.

Private Key는 한 번만 발급해주기 때문에 잘 관리해야 한다. 잃어버릴 경우, 복잡한 과정을 거쳐서 새로 Key를 다시 등록해야 한다.

![aws_ssh_1]({{site.baseurl}}/assets/img/aws/ssh_aws_1.png)


그 후, Private Key 파일의 권한 설정을 변경하고 SSH로 접속한다.

``` python
sudo chmod 400 ~/aws_ec2_test.pem

# ubuntu의 default 사용자명은 ubuntu
ssh -i ~/aws_ec2_test.pem ubuntu@[Public DNS or IP]
```

참고 : [Amazon Linux 인스턴스의 사용자 계정 관리 - AWS 설명서](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/managing-users.html){:target="_blank"}

파란 부분은 SSH로 처음 접속할 경우 known_hosts에 instance를 등록하는 모습이고

빨간 부분은 Private Key 권한 설정을 하지 않았을 경우, 생기는 오류이다.

![aws_ssh_2]({{site.baseurl}}/assets/img/aws/ssh_aws_2.png)

참고로, user(owner)만 읽을 수 있도록 권한이 변경된다.

![aws_ssh_3]({{site.baseurl}}/assets/img/aws/ssh_aws_3.png)

### 1. EC2 INSTANCE CONNECT

만약, EC2 Instance를 여러 사람이 공유하는 경우, Private Key를 공유해야 하는 보안 이슈가 생길 수 있다.

EC2 INSTANCE CONNECT 방식은 AWS CLI를 활용해 IAM과 연동하여 접속할 수 있다.

추가해야 할 IAM 권한은 아래 reference 링크에서 확인할 수 있다.

이 방식은 매번 임시의 SSH KEY를 발급해 등록 후 사용하는 방식이라고 한다. 

먼저, SSH를 통해 **인스턴스**에 접근 후 인스턴스용 패키지를 설치해야 한다.

[ Install EC2 Instance Connect on an instance - AWS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-set-up.html#ec2-instance-connect-install){:target="_blank"}

``` python
# Ubuntu의 경우
sudo apt update
sudo apt upgrade
sudo apt install ec2-instance-connect
```

**로컬 PC**에 EC2 CLI CONNECT 패키지를 설치한다.

``` python
pip install ec2instancecli
```

**mssh**라는 명령어를 통해 EC2 instance에 접근한다. ( 별도의 mssh 패키지가 아니다.)

``` python
mssh ubuntu@[instance ID]

# scp 역할을 하기 위해 다음처럼 활용
rsync -a -e mssh [로컬 파일] ubuntu@[instance ID]:/home/ubuntu/[파일명]
```

![aws_ec2connect]({{site.baseurl}}/assets/img/aws/aws_ec2connect.png)

## GCP VM Instance에 접속

GCP는 SSH Key를 직접 생성하고 등록하는 작업을 해야 한다. 2가지 방법으로 설정할 수 있다.


### 0. SSH Key 생성하기

``` python
# ssh-keygen -t rsa -f ~/.ssh/[KEY_FILENAME] -C [USERNAME]
ssh-keygen -t rsa -f ~/.ssh/gcp_ssh_test -C yahwang
```

gcp_ssh_test와 gcp_ssh_test.pub 파일이 생성된다.

### 1. 웹 console에서 등록하는 법

![gcp_ssh_1]({{site.baseurl}}/assets/img/gcp/ssh_gcp_1.png)

public key 파일 내용을 복사해서 Compute Engine - Metadata에 등록한다.

![gcp_ssh_2]({{site.baseurl}}/assets/img/gcp/ssh_gcp_2.png)

SSH KEY 생성 시 설정한 사용자명으로 로그인된다.

![gcp_ssh_3]({{site.baseurl}}/assets/img/gcp/ssh_gcp_3.png)

### 2. gcloud sdk로 등록하는 법

**OS 로그인 방식**이라고 한다. OS 로그인은 Google 계정과 연결하여 많은 사용자를 관리하는 데 권장되는 방법이라고 한다.

참고 : [OS 로그인의 이점](https://cloud.google.com/compute/docs/oslogin){:target="_blank"}

먼저, Project 또는 해당 instance에 OS login 사용 설정을 해야 한다. 웹 console에서 직접 또는 gcloud 명령어로 설정 가능하다.

![gcp_ssh_6]({{site.baseurl}}/assets/img/gcp/ssh_gcp_6.png)

``` python
# gcloud compute os-login ssh-keys add --key-file [public key]
gcloud compute os-login ssh-keys add --key-file ~/.ssh/gcp_ssh_test.pub
```

G Suite와 연동해 설정하지 않을 경우, 이메일 주소로 사용자명이 설정된다.

![gcp_ssh_5]({{site.baseurl}}/assets/img/gcp/ssh_gcp_5.png)

![gcp_ssh_4]({{site.baseurl}}/assets/img/gcp/ssh_gcp_4.png)


## 터미널 ssh config 등록

~/.ssh/config 파일에 다음과 같이 입력하면 터미널에서 간편하게 접속할 수 있다.

``` python
# ssh [Host 명]
ssh aws_ssh
```

![ssh_config]({{site.baseurl}}/assets/img/aws/ssh_config.png)


`References` : 

* [Set up EC2 Instance Connect - AWS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-set-up.html){:target="_blank"}


* [EC2 CLI CONNECT scp 해결 이슈](https://github.com/aws/aws-ec2-instance-connect-cli/issues/1){:target="_blank"}


* [Managing SSH keys in metadata - GCP 설명서](https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys#linux-and-macos_1){:target="_blank"}

* [OS 로그인 설정 - GCP 설명서](https://cloud.google.com/compute/docs/instances/managing-instance-access#gcloud_1){:target="_blank"}

* [How to Set Up SSH for a Google Cloud Platform Instance](https://www.cloudsavvyit.com/4358/how-to-set-up-ssh-for-a-google-cloud-platform-instance/){:target="_blank"}