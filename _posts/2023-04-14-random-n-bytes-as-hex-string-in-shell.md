---
layout: post
title:  "셸에서 랜덤한 N바이트를 16진수로 출력하기"
date:   2023-04-14 08:55:00 +0900
---
랜덤(random)한 20바이트의 데이터를 얻어와서 그걸 십육진수 문자열(hex string)로 출력하려면 어떻게 해야 할까? 셸에서 아래 한 줄만 실행하면 된다. ([출처](https://stackoverflow.com/questions/6292645/convert-binary-data-to-hexadecimal-in-a-shell-script/6292701#6292701))

```
xxd -l 20 -p /dev/urandom
```

실행 결과는 다음과 같다. 출력되는 값은 실행할 때마다 달라진다.

```console
$ xxd -l 20 -p /dev/urandom
e76f9c0cef52a6603595572df880914d17374eaf
```
**1바이트**는 **십육진수 두 자리**로 표현된다. 따라서 20바이트의 데이터는 위와 같이 십육진수 40자리로 표현된다.

나는 처음에 `dd` 명령으로 `/dev/urandom`에서 20바이트를 가져와서 파이프(`|`)로 `xxd`에 넘기는 걸 생각했는데, 검색해 보니 `xxd` 명령으로 바로 읽을 파일명과 읽을 바이트 수를 지정할 수 있었다.\
읽을 바이트 수는 length를 뜻하는 `-l` 옵션으로 지정하고, 읽을 파일명은 옵션 없이 지정한다.

참고로 위 명령에서 `-p` 옵션은 읽을 파일명을 지정하는 옵션이 아니다. `-p`는 그 뒤에 뭐가 붙지 않고 그냥 단독으로 쓰이는 옵션인데, plain hexdump style([여기](https://www.tutorialspoint.com/unix_commands/xxd.htm)의 설명을 보면, 다소 생소한 호명이기는 하지만, postscript hexdump style이라고도 부르는 모양이다)로 출력하라는 뜻이다. \
만일 위 명령에서 `-p` 옵션을 생략하면 아래와 같이 헥사 에디터(hexa editor) 스타일로 출력된다.

```console
$ xxd -l 20 /dev/urandom
00000000: b201 fb12 0a2d a356 bce1 7ef0 c9ce 30ef  .....-.V..~...0.
00000010: cd7d 3d4c                                .}=L
```

그렇다면 20바이트의 랜덤값 **10개**를 출력하려면 어떻게 해야 할까?

일단, 위 `xxd` 명령을 10번 반복해서 실행하면 된다. 반복 실행 방법에도 여러 가지가 있지만, 가령 이런 식으로 하면 된다.

```console
$ seq 10 | while read line; do xxd -l 20 -p /dev/urandom; done
bb7af05156bd9fa66651f6dc1e7d584361fdb8ee
989fcab7113439f1f5745f30920ba72ca4624797
f5afbe62a8bc45e7e1125fd044f14bb13ef1d2d2
4ea3d76db27be33daa3d9e3a6b7ecc471d3eebe8
af55ac1355a6c7a69bbb31418067347ec08ec7e4
417ef6539984b392e3891cb6df8b7652d70ab593
3e6fd4b64dcace4d09baff9d12060994e4211bb2
23f9fbd69f1b73bc34efb8e57435e24c68fe89cf
7cef905d6e372af8001a7ec36a87ba618b589fd9
d12db712c0170f1d7cf1c6b37e7e127740a6763d
```

하지만 이렇게 하면 `xxd` 명령이 개수만큼 반복해서 실행되기 때문에 비효율적이다. 한번에 200바이트를 통째로 읽어서 이를 20바이트씩 잘라서 출력하는 것이 효과적이다. 다행히 `xxd`에는 이를 위한 옵션이 있는데 바로 `-c` 옵션이다. `-c 20`을 지정하면 20바이트마다 줄바꿈되어 출력된다.

```console
$ xxd -l 200 -c 20 -p /dev/urandom
55c6c1a6303d1ce46166bc16222ec3e1d6e4c47a
528d9dd988709dd221248ff1035440a8321145cb
801a23e962df779e94ec1fd6fa740326cc3d733b
4e9a2858b0f240a872cdc253969753cda2416c28
393ed095c5a5fb4164dfb17398582b541a4a0748
8e02aacd1b4ec80e89222f4d1eb8b6063782f485
89f600bd275082a71a8f95ca62e25894d01c3a74
2477fd3b47858e9d7f9eaf665d332cc6f5d1ddc9
669bb87c3a022c26f4bc78cf7d1b4b4abf6530d6
02e473641469b1ad415edc3946311b9336d7d087
```

`-c` 옵션에 대해 발견한 한 가지 사실이 있다.\
내 노트북의 리눅스 환경(Ubuntu 20.04 LTS on WSL1)에서 `man xxd` 명령을 실행하여 `xxd`의 man page를 보면, `-c` 옵션으로 지정할 수 있는 칼럼 수의 최대치가 256이라고 나와 있다.

```console
       -c cols | -cols cols
              Format <cols> octets per line. Default 16 (-i: 12, -ps: 30, -b: 6). Max 256.
```

하지만 역시 내 노트북에서 실제로는 `xxd`의 `-c` 옵션에 256을 초과하는 값을 지정해도 기대한 대로 잘 실행된다.

```console
$ xxd -l 512 -c 256 -p /dev/urandom
98609fd3905a2901a7ee1ddf22bcf7e1d100f6c2480a751bf989c72d4745de5062d9edcd02a8ba0c149718ac7b1196a2a2fb46bdeb8f58c7f87ca514741801a65b5b3f07ee5c91d0cea11df5bc12e14417f2f35d6cf0b67375a6174a1104cac0815c29ceca8f08c8bb5f72cc44645e1a2dc3fdc1f0ff2636ba0948645a9b8dc40a267c358bc484e1ec0724f8cf05454d3b3acb704557ba55a70662b994e45f7f87d8464fbceef30a7c8e3769aaaa98d585f8ca3f859205a5f84145ae88fc6a70d38abfe79c4fa2f2b6cad4abb8880b66ca765f4ceaadb833713bea24b85a639eae10d8d86a8a9ef840fc9023dbba823cb594f72d85048d18c32378696ac8fe7d
6f47b5174425f326f0f26c6a79fd8d6142e9aef8ef13e0ebd58067e42931a0ae75d7c4f47635d0b9f6735f0c1ec22ebddf935fbbab0284b416e79871ccb3579ac71acc5380498fb90c6668df8aec6b89d2ff948589422c2afa17013eafaf26a7d73a4c8f7238eabffcc25e705c0ce779e039699108a4525c479c410a1dc509bc93da2b5f02849c974a222f7e7a4269c177d0222d914974e9a1ee21ab6a30c50cbbb30fe8fe0136e6696b1d1357abfce8162fdce34ad2de23b48e64f3eb7d890fc0a1fe07e4dd42127860e18061f2d5dc165292270c3f0e5d809618c66a679c2d87a861420794758cc7b286e6de1fb9513f26deece06c4885f4a54c3df0d8973b
$ xxd -l 514 -c 257 -p /dev/urandom
4ca47a8778060727b85d07b30d28c9cea95d4ce466a4e1b591512bd667c82e61dfdc79b37f29aa1b84c98921756bdfc514a564307d306e08b8c43ef74eb92ff007ff6445fa4769aa5d12ecf82e988882b5c7bc4803f1668a3d51b95fee3a7d41eaca4265a178538c86aa8fbc192adfcff496658117b4b3ee4661228c2cb4015831540f061e5422dac0da8bcd14da4c681a2d98bbf4805e0bac5983af0e96c0a7ae2b51ec65ab50cceb10db87720d5138c25499fb0e210b8d711053f68ed29551b8f98c5506723c8dff37d66679b8a92ff1918f0d33c1d3915dafdfe673a51ed98a2fd0ab94f89320922410b8881dcb5577dfd65eadf0d45e397e160ef436661f4e
64d6e3fe9b72949b622a3f353d449361e0c8d4f1ae0b939490d8f562d6bec46e68c456ee2a1316bddee21c4b10c50b09cf3ece4ee3ef5bd10fa94146914dceae6ea83027e03560a9aec2a42c63fd746426cf1bfa7dd3ef9797137269cbc444cbd2e056f8e31f5e8c40418954a2569665322c3e3a9ed0c87dc62fad0f10c2da42e5353caa3f1c65d002959d4cf39da2104a3ead2a7612277daa354540b3cf0e32d977faa4d192e453ae9e53dea7298ab4c12d0aa9abdc8fd1aa8f318f3b19c0228908af925bbb5f638a38178f33f073a15a2c31a2680089166bd2c1b63984ce18f3fba26e9c8f7da33ef60e53d269142a71faccabd65d0f992c128eb1d99f71c0fe
```

내 노트북에서 실행되는 `xxd`의 버전 정보와 도움말은 아래와 같다. `xxd`의 각 구현마다 `-c` 옵션의 실제 최대값이 얼마인지는 좀 더 조사해 볼 필요가 있어 보인다.

```console
$ xxd --version
xxd V1.10 27oct98 by Juergen Weigert
$ xxd --help
Usage:
       xxd [options] [infile [outfile]]
    or
       xxd -r [-s [-]offset] [-c cols] [-ps] [infile [outfile]]
Options:
    -a          toggle autoskip: A single '*' replaces nul-lines. Default off.
    -b          binary digit dump (incompatible with -ps,-i,-r). Default hex.
    -C          capitalize variable names in C include file style (-i).
    -c cols     format <cols> octets per line. Default 16 (-i: 12, -ps: 30).
    -E          show characters in EBCDIC. Default ASCII.
    -e          little-endian dump (incompatible with -ps,-i,-r).
    -g          number of octets per group in normal output. Default 2 (-e: 4).
    -h          print this summary.
    -i          output in C include file style.
    -l len      stop after <len> octets.
    -o off      add <off> to the displayed file position.
    -ps         output in postscript plain hexdump style.
    -r          reverse operation: convert (or patch) hexdump into binary.
    -r -s off   revert with <off> added to file positions found in hexdump.
    -s [+][-]seek  start at <seek> bytes abs. (or +: rel.) infile offset.
    -u          use upper case hex letters.
    -v          show version: "xxd V1.10 27oct98 by Juergen Weigert".
```

한편, 만약에 `xxd` 명령의 `-c` 옵션의 최대치가 실제로 256으로 제한된다면 어떻게 해야 할까?\
이 질문을 일반화하면\
‘일반적인 바이트 배열을 n바이트마다 줄바꿈(개행 문자 `\n` 삽입)해서 출력하려면 어떻게 해야 하는가?’\
라는 질문이 된다.\
구글에서 [add newline at n bytes in shell](https://www.google.com/search?q=add+newline+at+n+bytes+in+shell&newwindow=1)로 검색해 보면 명령행에서 이 질문에 대한 (완전한 혹은 부분적인) 여러 가지 해법을 찾을 수 있는데, 거기까지 논하기엔 글이 너무 길어지므로 다음 기회로 미루는 게 좋을 것 같다.

여기서는 `/dev/urandom`과 관련하여 한 가지만 덧붙이고 끝내겠다.

`/dev/urandom`은 랜덤 바이트를 공급하는 특수 파일이다. 그런데, 이 `/dev/urandom` 파일 역시 `/dev/zero`와 마찬가지로 데이터를 끝없이 공급하기 때문에, 파일 전체를 다 읽거나 파일의 끝을 찾으려고 하면 안 된다. 반드시 읽을 바이트 수를 한정할 수 있는 명령(이를테면 `dd`, `head`, 그리고 방금 소개한 `xxd` 명령 등)을 사용해서 다뤄야 한다.\
`tail`명령으로 `/dev/urandom` 끝의 N바이트를 가져오려고 하면 실행이 끝나지 않고 무한정 계속 실행된다.\
`cp` 명령으로 `/dev/urandom`의 전체 내용을 복사하려고 하면 디스크의 남은 공간을 다 채우기 전까지 계속 실행된다.

`/dev/urandom` 말고 `/dev/random`이라는 것도 있다. 역시 글이 길어질 듯 하여 다음 글에서 이야기해 보겠다.
