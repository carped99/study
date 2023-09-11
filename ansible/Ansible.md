# Ansible 설치

## Ansible
* Ansible은 Configuration Management and Application Deployment 시스템으로, 관리 및 운영 프로세스를 위해 많은 수의 서버를 제어하는 프로세스를 간소화하도록 설계되어 있다. 즉, 자동화된 방식으로 여러 원격 시스템을 제어할 수 있다.

* Ansible은 SSH를 사용하여 원격 시스템/서버를 제어한다. 

* Ansible은 원격 호스트 또는 머신과 상호 작용할 수 있는 두 가지 다른 방법이 있다.
  1. **ad-hoc:** command-line 에서 직접 앤서블 모듈을 호출해서 실행하는 방식으로 재사용이 필요 없는 명령를 실행할 때 사용
  2. **playbook:** YAML 형식



## 설치

[Docker Machine Test](#docker-machine-사용하기)

Ansible을 시스템에 설치할 수 있는 여러 가지 방법이 있다. 여기서는 [PPA(Personal Package Archive)](PPA.md)방법에 대해 설명한다.

### Installing via PPA :
먼저 다음 명령을 실행하여 공식 프로젝트의 PPA를 시스템에 포함해야 합니다:
```shell
$ sudo apt update && sudo apt-add-repository -y --update ppa:ansible/ansible
```

* add-apt-repository command not found 오류 

    add-apt-respository 패키지를 시스템에 설치한다.

    하지만 apt install add-apt-respository를 사용하려고 하면 작동하지 않는다.

    add-apt-repository 명령어는 software-properties-common 패키지의 일부이고 add-apt-repository를 설치하려면 이 패키지를 설치해야 한다.

    ```shell
    $ sudo apt install -y software-properties-common
    ```

다음 명령을 실행하여 Ansible을 설치한다.
```shell
$ sudo apt update && sudo apt install -y ansible
```

다음 명령을 사용하여 설치를 확인한다.
```shell
$ ansible --version
ansible [core 2.12.10]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/tykim/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/tykim/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.8.2 (default, Mar 13 2020, 10:14:16) [GCC 9.3.0]
  jinja version = 2.10.1
  libyaml = True
```

## Inventory File 생성
인벤토리 파일은 Ansible로 관리할 호스트 시스템/서버에 대한 정보를 담은 파일이다.

3가지 방법으로 지정할 수 있다.

1. `/etc/ansible/hosts` 기본 설정
2. `ANSIBLE_INVENTORY` 환경 변수
3. ansible 실행시 `-i` 옵션 지정

적용 우선 순위는 3, 2,1 순이며 여러 개가 지정되어 있을 경우 우선 순위 높은 항목만 적용된다.

기본적으로 Ansible 설치는 인벤토리 파일을 설정할 때 참조할 수 있는 많은 경우를 제공합니다. 그러나 이를 위해 그룹 이름 [myremote] 및 하위 그룹 이름 [myremote:vars]로 세 개의 서버를 정의하고 변수를 설정할 것입니다. remote1, remote2 및 remote3에서 Ansible 호스트의 IP 주소를 바꾸십시오.
```ini
[myremote]
remote1 ansible_host=111.111.111.111
remote2 ansible_host=222.222.222.222
remote3 ansible_host=333.333.333.333
 
[myremote:vars]
ansible_python_interpreter=/usr/bin/python3
```

호스트 매개 변수로 myremote:vars 하위 그룹에 있는 ansible_python_interpreter를 사용하고 있으며, 이 매개 변수는 myremote:vars 하위 그룹에 지정된 모든 호스트에 대해 호출됩니다.

다음 명령을 실행하면 인벤토리 파일의 변경 내용을 확인할 수 있습니다.
```shell
$ ansible-inventory --list --yaml
all:
 children:
   servers:
     hosts:
       remote1:
         ansible_host: 111.111.111.111
         ansible_python_interpreter: /usr/bin/python3
       remote2:
         ansible_host: 222.222.222.222
         ansible_python_interpreter: /usr/bin/python3
       remote3:
         ansible_host: 333.333.333.333
         ansible_python_interpreter: /usr/bin/python3
   ungrouped: {}
```

## Setup SSH Keys
[SSH 기본 설명](ssh.md)


새로운 키를 생성한다. 물론 이미 있는 키를 사용해도 된다.
```shell
$ ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible
```

이제 ssh 키를 remote1, remote2, remote3인 Ansible 호스트에 복사한다.
```shell
$ ssh-copy-id -i ~/.ssh/ansible.pub root@111.111.111.111
$ ssh-copy-id -i ~/.ssh/ansible.pub root@222.222.222.222
$ ssh-copy-id -i ~/.ssh/ansible.pub root@333.333.333.333
```

## 연결 테스트
ping 모듈을 실행해서 ping 결과를 확인한다.
```shell
$ ansible all -m ping -u root --private-key ~/.ssh/ansible
remote1 | SUCCESS => {
   "changed": false,
   "ping": "pong"
}
remote2 | SUCCESS => {
   "changed": false,
   "ping": "pong"
}
remote3 | SUCCESS => {
   "changed": false,
   "ping": "pong"
}
```

Ansible 설정이 완료되면 서버에서 임시 명령어 및 플레이북 실행을 시작할 수 있습니다.

Ansible Ad-hoc 명령어는 /usr/bin/ansible 명령줄 도구를 사용하여 원격 서버에서 하나의 작업을 자동화합니다. Ad-hoc 명령어는 빠른 작업에 사용되지만 재사용이 불가능하기 때문에 플레이북으로 대체할 수 없습니다. Ansible로 진행하면 거의 사용하지 않을 것입니다.

Ad-Hoc 명령어는 다음과 같습니다:

ansible [\[pattern\]](https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html#intro-patterns) -m [\[module_name\]](https://docs.ansible.com/ansible/latest/module_plugin_guide/index.html#working-with-modules) -a [module_args]

```shell
$ ansible all -m shell -a ‘whoami’
```

## Docker Machine 설치하기
Docker를 사용하여 테스트용 Linux Machine을 설정한다.
```shell
$ cd docker
$ docker compose up -d
```

* 도커 컨테이너의 IP 조회
    ```shell
    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <<Container_ID>>
    ```