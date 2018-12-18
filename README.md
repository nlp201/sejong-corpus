#### sejong-corpus ####

국립국어원 세종 말뭉치를 다운로드하는 스크립트입니다.

본 스크립트는 다음과 같은 이유로 제작되었습니다.

1) 세종 말뭉치 관련 파일들이 1400개가 넘습니다. 또한 전체 크기는 2GB에 해당합니다. 
2) 게시판을 통해 하나씩 받는데 어려움이 있습니다.
3) 말뭉치에 사소한 오류들이 있습니다. 또한, 세종 말뭉치는 개작 및 재배포를 허용하지 않는
라이센스를 가지고 있습니다. 따라서, 연구자들이 말뭉치를 받아 오류를 고쳐도 재배포가 허용되지
않습니다.

말뭉치 파일을 받을 때 항상 보이는, 자료에 대한 라이센스가 있습니다.  본 스크립트는 그 라이센스
노출을 우회하는 결과를 가져오므로, 사용자는 해당 라이센스를 읽고 동의한다는 것을 가정합니다.
다운로드 전 동의해야 하는 라이센스의 사본은 LICENSE.txt 입니다.

### 사용법 ###

#### 다운로드 및 UTF-8으로 변환 ####

```
$ make
```

게시판의 게시물 목록을 추출하고, 각 게시물에서 첨부파일을 받는 과정입니다. 총 4 STEP

#### STEP 0 ####
이 단계에서는 필요한 툴의 존재를 확인합니다. 없는 경우 적당하게 설치를 유도합니다. 만약 본
스크립트가 실행되지 않는 환경에서라면 소스(00.prepare.sh) 를 보시고 필요한 패키지를 수동으로
설치해주시기 바랍니다.

#### STEP 1 ####
이 단계는 게시물 목록을 가져오는 단계입니다.

#### STEP 2 ####
이 단계는 각 게시물의 첨부파일을 다운로드합니다.

#### STEP 3 ####
이 단계는 첨부파일의 형식(UTF16)을 UTF8으로 바꿉니다. 또한, 연구자의 수작업 패치를 가하는
단계입니다.

```
$ make dic
```

말뭉치가 준비되었으면 (corpus-utf8 디렉토리에) 해당 말뭉치로부터 형태소 분석 결과만을 추출하고
추출된 형태소 및 태깅된 품사로부터 역으로 사전을 구축하는 단계입니다. (dictionary 디렉토리에)

#### STEP 4 ####
이 단계는 각 파일 중 형태소 분석 결과를 포함한 파일에서 유형을 따라 파일을 형태소 및 품사 부분을
추출합니다. 추출된 파일들은 최종 통합되어 하나의 파일을 만듭니다.

#### STEP 5 ####
통합된 거대 파일을 정렬한 뒤 인접한 중복행을 제거하는 방식으로 유일한 표제어를 추출하는 단계입니다.

#### STEP 6 ####
유일한 표제어 집합을 만든다음 각 품사별로 쪼개어 품사(POS)별로 나누어 저장합니다

### 수정 ###
corpus/*.txt 파일은 UTF18 원본이므로 수정하지 않습니다. 말뭉치에 문제가 있는 경우 corpus-utf8/*.txt
파일을 직접 수정합니다.

```
make diff
```

위 명령을 수행하면 corpus-utf8.orig 에 다시 한번 원본 파일을 복원합니다. 그리고 작업자가 수정한
파일이 있는 corpus-utf8 디렉토리의 파일들과 비교하여 바뀐 파일들의 차이만을 patches 디렉토리에
저장합니다. 

만약 patches/*.patch 파일을 수동으로 corpus-utf8 파일들에 patch 하고 싶다면

```
$ cat patches/* | patch -N -d corpus-utf8 -p1
```

위와 같이 합니다. -N 옵션이 있으므로 이미 적용된 패치는 무시하므로 여러번 실행하여도 같은 결과를
유지합니다.

### 산출물 ###
* logs/list.idx : 국립국어원 언어정보나눔터 게시판 글 번호
* html/*:게시판 원문
* download/*:게시판 첨부파일
* corpus/*:첨부파일에서 말뭉치 추출
* corpus-utf8/*:말뭉치를 UTF8으로 변환
* corpus-utf8.orig/*:말뭉치를 UTF8으로 변환한 원본 (make diff 중 생김)
* logs/download.log: 첨부파일 다운로드 기록
* logs/words.dic: 단어/품사분석 추출 원본
* logs/words-uniq.dic: 단어/품사의 중복 제거
* dictionary/(POS).dic 품사별 사전

### 제공되는 도구 ###
## 00.prepare.sh ##
* 필요한 유틸리티 혹은 인터프리터의 설치를 확인합니다

## 10.list.sh ##
* 언어정보나눔터 웹페이지에 접속하여 필요한 파일들의 리스트를 받아옵니다.
* logs/list.idx 파일에 글의 시쿽스가 저장됩니다.

## 20.schedule.sh ##
* 다운로드를 병렬로 진행하기 위한 스케쥴러입니다.
* logs/list.idx 를 각 다운로더에 분배합니다.
* make 명령에 동시에 접속할 수를 M이라는 변수로 지정할 수 있습니다. 기본은 4 입니다.
```
make M=10
```

## 21.getcontent.sh ##
* 02.schedule.sh 에 의해 실행되는 게시글 다운로더입니다.
* logs/list.idx 의 시작위치와 갯수를 입력받아 실행합니다. (Makefile 참조)

## 22.download.sh ##
* 03.getcontent.sh에 의해 실행되는 첨부파일 다운로더입니다.
* 웹서버는 첨부파일이 한 개인 경우 txt 파일로, 여러개인경우 zip 파일로 묶여 제공합니다.
* 받은 파일은 corpus/XXXXXX.txt 파일로 저장 혹은 압축이 풀립니다.
* 모든 파일은 XML 형식으로 제공되며, 원문, 형태소분석, 단어의미분석 등 여러 형식이 섞여 있습니다.

## 30.convert.sh
* 받은 파일은 utf16 포맷입니다. 이 파일을 utf8 형식으로 변환할 대상을 찾습니다.

## 31.convert-file.sh
* 원본, 출력 파일 이름을 받아 실제 변환을 수행합니다.
* UTF16을 UTF8으로 변환합니다.
* 05.convert-xml-tag.py 파일을 이용하여 xml escaping을 합니다.

## 33.patch.sh
* 각 파일에 해당하는 patch 파일들을 찾아 패치를 수행합니다
* 패치 파일들의 이름은 patches/(CORPUSFILENAME)-(NNN).patch 형식으로 되어 있으며, 원본의 오류를
수정하는 내용만 들어 있습니다.

## 32.convert-xml-tag.py
* XML 파일 본문에 CDATA 영역에 해당하는 내용이 XML 규약에 따라 escaping되어 있지 않습니다.
* 말뭉치를 구성하는 태그들을 제외한 태그형식의 '<', '>' 문자를 '&lt;', '&gt;' 로 변환합니다.

## 40.extract.sh
* 받은 파일 중에서 형태소분석파일들을 대상으로 형태소 분석 결과만 추출하고 자/모 보정을 합니다
* 출력되는 결과는 logs/words.dic 에 통합하여 저장됩니다.

## 41.extract.py
* 40.extract.sh에 의해 실행되며 파일에서 형태소 분석 부분만 추출합니다

## 42.jamo-conv.py
* 40.extract.sh에 의해 실행되며, 형태소 분석 결과중에서 특정 연구 결과는 종성을 그대로 종성 코드로
저장한 경우가 있습니다. 이를 초성으로 변환합니다.

## 60.build_dic.py
* 하나로 합쳐있는 형태소 분석 결과 뭉치에서 각 형태소별로 dictionary/(POS).dic 파일로 분할정리합니다.

## 90.diff.sh
* 원본 파일과(corpus-utf8.orig/*.txt) 작업자가 보정한(corpus-utf8/*.txt) 파일을 비교하여
patches/*.patch 파일을 만듭니다.

