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
