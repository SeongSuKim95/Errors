
# 연구실 Server 관리하면서 생긴 일들 기록하기..

- 0314
    - Server들의 Ubuntu를 update 하고 , 재부팅을 하는데 43번만 로그인이 안되어서 보니 booting 과정에 문제가 있었음 
    - BIOS mode로 진입해있길래, 수동으로 booting을 하려고 했더니 아래 사진과 같은 문제가 생겼다.
        - Failure : File system check of the root filesystem failed. The root filesystem on /dev/nvme0n1p2 requires a manual fsck
    - ![fig](https://user-images.githubusercontent.com/62092317/158155225-207ad40c-1265-4140-88b7-f059c3bf0439.jpg)
    - Nvme 문제인것으로 보여서 google링 한 결과 [[LINK]](https://clay-atlas.com/us/blog/2020/03/19/linux-english-note-root-filesystem-manual-fsck/)와 같은 해결책을 찾을 수 있었다.
        ```console
        sudo fsck -f /dev/nvme0n1p2
        reboot
        ```
    - 다시 재부팅한 결과 정상적으로 동작하였는데, 강제 종료로 인한 hardware fault 문제라고 한다. 혹시 모르니 hdd에 weight 파일들은 backup해놓아야 할듯..

- 0318 (Github clone)
    - TransReID 관련 실험을 더 많이 하기 위해 45번을 사용하려고 하니, 내 repository에서 git clone이 안되더라.
        ```console
        git permission denied (publickey). fatal: Could not read from remote repository.
        ```
    - 생각해보니 먼 옛날에 내 github에서 지금 사용하고 있는 서버에 대한 SSH key를 등록하는 과정을 했어야 했음을 깨달았다. 까먹을 까봐 정리해 놓으려고 한다.
    - Git은 SSH 또는 http 기반으로 사용을 하게 되는데, SSH key로 접속해서 사용하는 경우는 PC마다 SSH key를 등록해주어야 한다.
        - Terminal에 ssh key 생성 명령어를 입력한다.
        ```console
        ssh-keygen -t rsa -C "SeongSuKim95@github.com"
        ```
        - .sss/id_rsa에 id_rsa 파일이 생성되면 enter
        - 비밀번호 입력을 원하면 비밀번호 입력, 아니면 enter
    - Github의 settings 메뉴에서, New SSH keys를 누른후 id_rsa.pub의 내용을 복붙한다.
    - 이후 git clone을 다시 해보면 clone이 잘될것이야

- 0318 ssh-keygen을 통한 비밀번호 자동인증
    - SSH Key : 서버에 접속할 때 비밀번호 대신 key를 제출하는 방식
    - 공개키(Public key)와 비밀키(Private Key)로 이루어 지며 private key는 local에 위치하고, public key는 서버에 위치한다. 암호를 걸 때와 풀 때 사용하는 키가 다르기 때문에 비대칭 암호방식이라고 한다. Public key는 평문을 암호화 한다면 비밀키는 암호문을 다시 평문으로 변환
    - Windows [[LINK]](https://sojinhwan0207.tistory.com/62) , Linux [[LINK]](https://jjeongil.tistory.com/1288)
    - Windows (local)
        - 1.power shell 에서 ssh-keyfen으로 키를 생성(저장위치 : c:\users\.ssh\id_rsa)
        - 2.scp .ssh\id_rsa.pub ID@server_ip 로 서버에 public키 전송
    - Server (Linux)
        ``` console
        chmod 700 .ssh/
        cat id_rsa.pub >> .ssh/authorized_keys
        chmod 600 .ssh/authorized_keys
        ```
    - 비밀번호 없이 접속!
- 0319 
    - https://www.manualfactory.net/13414
    - https://conory.com/blog/19194

- 0404
    - 연구실에 있는 서버를 서버실로 전부 옮기면서 각 서버마다 고정 ip설정을 바꿔줘야하는 일이 생겼다.
    - Ubuntu version에 따라 gui가 있는 경우가 있고 없는 경우가 있는데, 없는 경우 직접 network 설정 파일을 수정해줘야한다.(처음해봐서 삽질 했음...)
        - 일단 제일 먼저 랜선의 불이 깜빡이는지 확인하고, ifconfig를 통해 device가 잡고 있는 network의 이름을 확인하다.  
        - 확인 했다면, etc/netplan의 yaml파일을 직접 수정
            - 1. ethernet의 이름은 아까 확인한 network의 이름으로 바꿔주기
            - 2. 고정ip사용하므로 dhcp는 False
            - 3. addresses는 변경할 ip를 적어주면 되고(ex: 163.239.13.24/24) , gateway는 1번 (ex: 163.239.13.1)
            - 4. nameservers addresses : gate way와 netmask 적어주면 된다. [163.239.1.1,8.8.8.8] (Google DNS Server)
        - 파일을 수정했으면,  sudo netplan apply 이후 재부팅 해주면 ip 변경 완료!
- 0526 SSL certificate 갱신 
    - 갑자기 애들이 서버 로그인이 안된다고 해서, 확인해보니 permission denied 가 뜨더라
    - 처음엔 user 계정의 권한 문제인줄 알았는데 한참 헤매다보니 NAS의 LDAP Server 인증서가 만료된 문제 인걸 알았다.
        - Synology NAS 관리 페이지의 Security -> Certificate 에서 Add
        - Replace an existing certificate
        - Create self-signed certificate 
        - 인증서 만들고 난 후, Export certificate을 통해 pem파일을 다운 받는다
    - Pem 파일 중 root certificate만 각 server에 넣고 등록해주면 되는데, zip파일 안의 syno-ca-cert.pem 파일이다.
    - 확장자 pem을 crt로 바꾼 후, 각 server의 /usr/local/share/ca-certificates에 복사 
    - 복사했다면 sudo update-ca-certificates 이후, 재부팅 하면 끝이다.[[LINK]](https://ubuntu.com/server/docs/security-trust-store)
        ```console
        sudo apt-get install -y ca-certificates
        sudo cp local-ca.crt /usr/local/share/ca-certificates
        sudo update-ca-certificates
        ```
    - 만료된 crt 파일은 지워도 상관없음.

- 0604 Conda Enviornment
    - 연구실 구성원 한명이 본인의 로그인 환경에서 cuda가 잡히질 않는다고 며칠을 쩔쩔매고 있었다.
    - 문제를 파악 해본 결과 (문제가 되고 있는 User ID를 A, 가상환경을 B라고 할때)
        1. A로 로그인 하여, B 가상환경에서 torch.cuda.is_available() 확인시 False
        2. A로 로그인 하였을때, A의 나머지 가상환경에서 torch.cuda.is_available() 확인시 True
        3. C로 로그인 할 경우, A의 모든 가상환경(B포함)에서 torch.cuda.is_available() 확인 시 True
    - 처음엔 이게 무슨 문제인지.. 도저히 이해가 안갔다. 로그인 계정에 따라 같은 가상환경에서 결과가 다르다니?
        - 일단 B 가상환경에 깔린 torch를 확인해보니 , 서버의 gpu와 cuda version이 일치했다.
        - nvidia-smi 명령어도 잘 된다.
        - Conda 설정 문제라고 생각해서 A의 home directory에 있는 bash.rc의 cuda와 conda initialize 부분을 check했다. 문제가 없었다.
        - A가 B 가상환경으로 프로젝트 code를 실행시키는 과정에서 오류가 나서 이를 해결하려고 bin의 x86_6-conda-linux-gnu-ld 파일을 건드린 적이 있다고 해서, 찾아보니 library loader 파일이라고 한다. 지금까지 이파일이 문제가 됐던 기억은 없는데,, 이상하다고 생각했다.
        - 몇시간을 삽질하다가 차분히 error를 debugging해보니, B 가상환경에서 A의 다른 가상환경(D)에 있는 torch package를 불러오는 것을 알 수 있었다. D의 torch version이 cpu version이다.
        - 일단 임시방편으로, D의 torch version을 gpu version으로 바꾸고 cuda version도 맞춰주었다.
    - 문제가 해결되었다.. 굉장히 단순한 문제였는데, 복잡한 문제일거라고 생각해서 삽질을 좀 했다..
