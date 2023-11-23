---
description: Make의 명령어 다뤄보기
---

# Make 명령어

Make 명령어는 Linux와 macOS에서 사용할 수 있습니다.

다음 예제는 UbuntuOS(22.04LTS)를 기준으로 작성되었습니다.

#### Make --help

우선 --help 옵션을 통해서 어떤 명령어들이 있는지를 확인해보실 수 있습니다.\
make의 사용법은 아래와 같이 \`make \[옵션들] \[타겟]

```bash
$ make --help
사용법: make [옵션] [타겟] ...
옵션:
  -b, -m                      무시됩니다, 호환을 위해 유지.
  -B, --always-make           조건에 관계 없이 모든 타겟을 만듭니다.
  -C <디렉터리>, --directory=<디렉터리>
                              뭔가 하기 전에 <디렉터리>로 이동합니다.
  -d                          여러 가지 디버깅 정보를 출력합니다.
  --debug[=플래그]            여러 가지 종류의 디버깅 정보를 출력합니다.
  -e, --environment-overrides
                              환경변수가 메이크파일 내용에 우선합니다.
  -E <문자열>, --eval=<문자열>
                              <문자열>을 메이크파일 내용으로 해석합니다.
  -f <파일>, --file=<파일>, --makefile=<파일>
                              <파일>을 메이크파일로 읽습니다.
  -h, --help                  이 메시지를 출력하고 끝냅니다.
  -i, --ignore-errors         명령에서 발생하는 오류를 무시합니다.
  -I <디렉터리>, --include-dir=<디렉터리>
                              포함할 메이크파일을 <디렉터리>에서 찾습니다.
  -j [N], --jobs[=N]          동시에 N개의 작업 허용, 인자 없으면 무한대로 허용.
  -k, --keep-going            일부 타겟을 만들 수 없더라도 계속 진행합니다.
  -l [N], --load-average[=N], --max-load[=N]
                              로드가 N 아래로 내려가야 동시 작업 시작합니다.
  -L, --check-symlink-times   심볼릭 링크와 실제 중 더 최근 수정 시각을
                               사용합니다.
  -n, --just-print, --dry-run, --recon
                              실제로는 아무 명령도 실행하지 않고 표시만 합니다.
  -o <파일>, --old-file=<파일>, --assume-old=<파일>
                              <파일>이 아주 오래 되었다고 생각하고 다시 만들지
                              않습니다.
  -O[방식], --output-sync[=방식]
                              병렬 작업의 출력을 <방식>에 따라 맞춥니다.
  -p, --print-data-base       make의 내부 데이터베이스를 출력합니다.
  -q, --question              명령을 실행하지 않음. 종료 상태로 업데이트
                              여부를 알 수 있습니다.
  -r, --no-builtin-rules      내장 묵시적 규칙을 사용하지 않습니다.
  -R, --no-builtin-variables  내장 변수를 지정하지 못하게 합니다.
  -s, --silent, --quiet       명령어를 출력하지 않습니다.
  --no-silent                 명령어를 출력합니다. (--silent 모드 끄기)
  -S, --no-keep-going, --stop
                              -k 옵션을 끕니다.
  -t, --touch                 타겟을 다시 만들지 않고 touch만 합니다.
  --trace                     추적 정보를 표시합니다.
  -v, --version               make의 버전 번호를 출력하고 끝냅니다.
  -w, --print-directory       현재 디렉터리를 출력합니다.
  --no-print-directory        묵시적으로 켜져 있더라도 -w를 끕니다.
  -W <파일>, --what-if=<파일>, --new-file=<파일>, --assume-new=<파일>
                              <파일>을 무한히 계속 새로운 것으로 취급합니다.
  --warn-undefined-variables  정의되지 않은 변수를 참조할 때 경고를 냅니다.

이 프로그램은 x86_64-pc-linux-gnu에서 사용하도록 빌드되었습니다
문제점을 <bug-make@gnu.org>로 알려 주십시오.

```

#### Make



