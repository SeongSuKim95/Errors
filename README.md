# Linux

Server관리하면서 생긴 일들 기록하기..

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