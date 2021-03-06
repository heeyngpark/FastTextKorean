# FastText for Korean

## 설치

**아래 메뉴얼은 AWS EC2 r4.large 인스턴스에서 테스트되었습니다.**

Embedding 에 대한 기초지식은 [여기](https://ratsgo.github.io/embedding/) 가 좋습니다. 인터넷 전체를 구글링하는것보다 여기 블로그를 찾는 것이 낫습니다.

### 개발환경 구성

```sh
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python-setuptools
sudo apt install python3-pip

pip3 install soynlp
pip3 install soyspacing
pip3 install sentencepiece
pip3 install bert
pip3 install bert-tensorflow
pip3 install tensorflow
pip3 install hgtk

# sudo apt install make
# sudo apt-get install build-essential

sudo apt install unzip

sudo apt install cmake
```

### fasttext 설치

fasttext 를 컴파일합니다.

``` sh
cd ~/
git clone https://github.com/facebookresearch/fastText.git
cd fastText
make
```

python 모듈을 설치합니다.

```sh
pip3 install .
```

```sh
./fasttext -help
```

### 데이타 전처리

#### 전처리 끝난 데이타 다운받기

[여기](https://github.com/ratsgo/embedding) 를 클론한다.

```sh
cd ~/
git clone https://github.com/ratsgo/embedding.git
cd embedding
```

[구글 드라이브](https://drive.google.com/file/d/1kUecR7xO7bsHFmUI6AExtY5u2XXlObOG/view) 에서 전처리가 끝난 데이타를 다운받습니다.

```sh
mkdir -p ~/bin
vi ~/bin/gdrive_download

-----------------------------------------
#!/usr/bin/env bash

# gdrive_download
#
# script to download Google Drive files from command line
# not guaranteed to work indefinitely
# taken from Stack Overflow answer:
# http://stackoverflow.com/a/38937732/7002068

gURL=$1
# match more than 26 word characters
ggID=$(echo "$gURL" | egrep -o '(\w|-){26,}')

ggURL='https://drive.google.com/uc?export=download'

curl -sc /tmp/gcokie "${ggURL}&id=${ggID}" >/dev/null
getcode="$(awk '/_warning_/ {print $NF}' /tmp/gcokie)"

cmd='curl --insecure -C - -LOJb /tmp/gcokie "${ggURL}&confirm=${getcode}&id=${ggID}"'
echo -e "Downloading from "$gURL"...\n"
eval $cmd
-----------------------------------------

chmod 700 ~/bin/gdrive_download

~/bin/gdrive_download https://drive.google.com/file/d/1kUecR7xO7bsHFmUI6AExtY5u2XXlObOG
```

모델을 학습하고, 데이타를 형태소분석 합니다.

```sh
mkdir mywork
cd mywork
mv ../processed.zip ./
unzip processed.zip

# train
python3 ../preprocess/unsupervised_nlputils.py --preprocess_mode compute_soy_word_score \
    --input_path ./processed/corrected_ratings_corpus.txt \
    --model_path ./soyword.model

# tokenize
python3 ../preprocess/unsupervised_nlputils.py --preprocess_mode soy_tokenize \
    --input_path ./processed/corrected_ratings_corpus.txt \
    --model_path ./soyword.model \
    --output_path ./ratings_tokenized_soy.txt

head -5 ./ratings_tokenized_soy.txt
어릴때 보고 지금 다시 봐도 재밌 어ㅋㅋ
디자인을 배우 는 학생 으로, 외국 디자이너와 그들 이 일군 전통을 통해 발전 해가는 문화 산업이 부러웠는데. 사실 우리나라 에서도 그 어려운 시절에 끝까지 열정 을 지킨 노라노 같은 전통이있어 저와 같은 사람들이 꿈을 꾸고 이뤄나갈 수 있다 는 것에 감사합니다.
폴리스스토리 시리즈는 1부터 뉴까지 버릴께 하나 도 없음. . 최고 .
와.. 연기 가 진짜 개쩔구나.. 지루 할거라고 생각 했는데 몰입 해서 봤다. . 그래 이런 게 진짜 영화 지
```

### khaiii 설치

[문서](https://github.com/kakao/khaiii/wiki/%EB%B9%8C%EB%93%9C-%EB%B0%8F-%EC%84%A4%EC%B9%98) 를 참조하여 khaiii 를 설치합니다.

```sh
cd ~/
git clone https://github.com/kakao/khaiii.git
cd khaiii
cmake --version

mkdir build
cd build/
cmake ..

make all
make resource
make large_resource
sudo make install
khaiii --help
```

테스트하기

```sh
vi input.txt
-----------------------------------------
동해물과 백두산이 마르고 닳도록 하느님이 보우하사 우리나라 만세
무궁화 삼천리 화려강산 대한 사람 대한으로 길이 보전하세
-----------------------------------------

khaiii --input input.txt
```

아래 명령으로 python 연동모듈을 설치할 수 있습니다.

```sh
make package_python
cd package_python
pip3 install .
```

### mecab-ko 설치

```sh
# mecab-ko
cd ~/
wget https://bitbucket.org/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz
tar xvfz mecab-0.996-ko-0.9.2.tar.gz
cd mecab-0.996-ko-0.9.2
./configure --prefix=/usr
make
make check
sudo make install

# mecab-ko-dic
sudo ldconfig
ldconfig -p | grep /usr/local/lib
cd ~/
wget https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/mecab-ko-dic-2.1.1-20180720.tar.gz
tar xvfz mecab-ko-dic-2.1.1-20180720.tar.gz
cd mecab-ko-dic-2.1.1-20180720
./configure --prefix=/usr
make
sudo make install

# 하드코딩으로 박혀있는 디렉토리 수정
sed -i -e 's/\/usr\/local/\/usr/g' tools/add-userdic.sh

# mecab-python
pip3 install python-mecab-ko
```

## 사용법

### fasttext 사용법

#### 데이타 준비

```sh
cd ~/fastText
mkdir mywork
cp ~/embedding/mywork/processed/corrected_ratings_corpus.txt mywork/
```

```sh
cd mywork
```

#### 형태소분석 없는 데이타로 학습

```sh
# using cbow
../fasttext cbow -input corrected_ratings_corpus.txt -output model_cbow

# using skipgram
../fasttext skipgram -input corrected_ratings_corpus.txt -output model_skipgram

# nearest neighbors
echo “디즈니” | ../fasttext nn model_skipgram.bin
Query word? 디즈니 0.968521
디즈니는 0.956993
디즈니와 0.934998
디즈니의 0.920893
클레이 0.8961
디즈니가 0.889897
함정 0.871816
레전드. 0.864659
쌍벽을 0.86439
걸작중 0.859125
```

Text Classification 을 위한 학습데이타를 다운받는다.

```sh
wget https://dl.fbaipublicfiles.com/fasttext/data/cooking.stackexchange.tar.gz
tar xvzf cooking.stackexchange.tar.gz
head cooking.stackexchange.txt
__label__sauce __label__cheese How much does potato starch affect a cheese sauce recipe?
__label__food-safety __label__acidity Dangerous pathogens capable of growing in acidic environments
__label__cast-iron __label__stove How do I cover up the white spots on my cast iron stove?
__label__restaurant Michelin Three Star Restaurant; but if the chef is not there
__label__knife-skills __label__dicing Without knife skills, how can I quickly and accurately dice vegetables?
__label__storage-method __label__equipment __label__bread What\'s the purpose of a bread box?
__label__baking __label__food-safety __label__substitutions __label__peanuts how to seperate peanut oil from roasted peanuts at home?
__label__chocolate American equivalent for British chocolate terms
__label__baking __label__oven __label__convection Fan bake vs bake
__label__sauce __label__storage-lifetime __label__acidity __label__mayonnaise Regulation and balancing of readymade packed mayonnaise and other sauces

# 데이타셋 분리
head -n 12404 cooking.stackexchange.txt > cooking.train
tail -n 3000 cooking.stackexchange.txt > cooking.test
```

```sh
../fasttext supervised -input cooking.train -output model_cooking
```

```sh
../fasttext predict model_cooking.bin -
Which baking dish is best to bake a banana bread ?
__label__baking
^C

../fasttext predict model_cooking.bin - 5
Why not put knives in the dishwasher?
__label__food-safety __label__baking __label__bread __label__equipment __label__substitutions
^C
```

파라미터 정리

| parameter         | description                                      | default   |
|-------------------|--------------------------------------------------|-----------|
| input             | training file path                               | mandatory |
| output            | output file path                                 | mandatory |
| verbose           | verbosity level                                  | 2         |
| minCount          | minimal number of word occurences                | 5         |
| minCountLabel     | minimal number of label occurences               | 0         |
| wordNgrams        | max length of word ngram                         | 1         |
| bucket            | number of buckets                                | 2000000   |
| minn              | min length of char ngram                         | 3         |
| maxn              | max length of char ngram                         | 6         |
| t                 | sampling threshold                               | 0.0001    |
| label             | labels prefix                                    | []        |
| lr                | learning rate                                    | 0.05      |
| lrUpdateRate      | change the rate of updates for the learning rate | 100       |
| dim               | size of word vectors                             | 100       |
| ws                | size of the context window                       | 5         |
| epoch             | number of epochs                                 | 5         |
| neg               | number of negatives sampled                      | 5         |
| loss              | loss function {ns, hs, softmax}                  | ns        |
| thread            | number of threads                                | 12        |
| pretrainedVectors | pretrained word vectors for supervised learning  | []        |
| saveOutput        | whether output params should be saved            | 0         |
| cutoff            | number of words and ngrams to retain             | 0         |
| retrain           | finetune embeddings if a cutoff is applied       | 0         |
| qnorm             | quantizing the norm separately                   | 0         |
| qout              | quantizing the classifier                        | 0         |
| dsub              | size of each sub-vector                          | 2         |

#### 형태소분석 진행한 데이타로 학습(soy_tokenize)

```sh
cd ~/fastText/mywork/
cp ~/embedding/mywork/ratings_tokenized_soy.txt ./
```

```sh
cd mywork

#### 형태소분석 진행한 데이타로 학습
../fasttext skipgram -input ratings_tokenized_soy.txt -output model_skipgram

# nearest neighbors
echo “디즈니” | ../fasttext nn model_skipgram.bin
Query word? 디즈니 0.994486
픽사 0.707323
애니메이션 0.70667
애니중 0.700826
애니 0.695701
애니의 0.689524
웍스 0.678675
애니를 0.675855
에니메이션 0.672339
2D 0.671045
```

#### 형태소분석 진행한 데이타로 학습(khaiii)

```sh
vi unsupervised_nlputils.py
```

```python
import sys, math, argparse, re
from khaiii import KhaiiiApi
import mecab

def khaiii_tokenize(corpus_fname, output_fname):
    api = KhaiiiApi()

    with open(corpus_fname, 'r', encoding='utf-8') as f1, \
            open(output_fname, 'w', encoding='utf-8') as f2:
        for line in f1:
            sentence = line.replace('\n', '').strip()
            tokens = api.analyze(sentence)
            tokenized_sent = ''
            for token in tokens:
                tokenized_sent += ' '.join([str(m) for m in token.morphs]) + ' '
            f2.writelines(tokenized_sent.strip() + '\n')


def mecab_tokenize(corpus_fname, output_fname):
    mcab = mecab.MeCab()

    with open(corpus_fname, 'r', encoding='utf-8') as f1, \
            open(output_fname, 'w', encoding='utf-8') as f2:
        for line in f1:
            sentence = line.replace('\n', '').strip()
            tokens = mcab.morphs(sentence)
            tokenized_sent = ' '.join(tokens)
            f2.writelines(tokenized_sent + '\n')


if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--preprocess_mode', type=str, help='preprocess mode')
    parser.add_argument('--input_path', type=str, help='Location of input files')
    parser.add_argument('--output_path', type=str, help='Location of output files')
    args = parser.parse_args()

    if args.preprocess_mode == "khaiii_tokenize":
        khaiii_tokenize(args.input_path, args.output_path)
    elif args.preprocess_mode == "mecab_tokenize":
        mecab_tokenize(args.input_path, args.output_path)
```

```sh
python3 ./unsupervised_nlputils.py --preprocess_mode khaiii_tokenize \
    --input_path ./corrected_ratings_corpus.txt \
    --output_path ./ratings_tokenized_khaiii.txt

head -5 ./ratings_tokenized_khaiii.txt
어리/VA ㄹ/ETM 때/NNG 보/VV 고/EC 지금/MAG 다/NNG 시/MAG 보/VV 아/EC 도/JX 재미있/VA 어요/EC ㅋㅋ/NNG
디자인/NNG 을/JKO 배우/VV 는/ETM 학생/NNG 으로/JKB ,/SP 외국/NNG 디자이/NNG 너/NP 와/JKB 그/NP 들/XSN 이/JKS 일/VV 군/NNG 전통/NNG 을/JKO 통하/VV 여/EC 발전/NNG 하/XSV 여/EC 가/VX 는/ETM 문화/NNG 산업/NNG 이/JKS 부럽/VA 었/EP 는데/EC ./SF 사실/MAG 우리나라/NNG 에서/JKB 도/JX 그/MM 어렵/VA ㄴ/ETM 시절/NNG 에/JKB 끝/NNG 까지/JX 열정/NNG 을/JKO 지키/VV ㄴ/ETM 노라노/NNG 같/VA 은/ETM 전통/NNG 이/JKS 있/VV 어/EC 저/NP 와/JKB 같/VA 은/ETM 사람/NNG 들/XSN 이/JKS 꿈/NNG 을/JKO 꾸/VV 고/EC 이루/VV 어/EC 나가/VX ㄹ/ETM 수/NNB 있/VV 다는/ETM 것/NNB 에/JKB 감사/NNG 하/XSV ㅂ니다/EF ./SF
폴리스스토리/NNG 시리즈/NNG 는/JX 1/SN 부터/JX 뉴/NNG 까지/JX 버리/VV ㄹ께/EC 하나/NR 도/JX 없/VA 음/ETN ../SE 최고/NNG ./SF
와/IC ./SF ./SE 연기/NNG 가/JKS 진짜/MAG 개쩌/VV ㄹ구나/EF ../SE 지루/XR 하/XSA ㄹ/ETM 거/EC 이/VCP 라고/EC 생각/NNG 하/XSV 였/EP 는데/EC 몰입/NNG 하/XSV 여서/EC 보/VV 았/EP 다/EF ../SE 그래/IC 이런/MM 것/NNB 이/JKS 진짜/NNG 영화지/NNG
안개/NNG 자욱/XR 하/XSA ㄴ/ETM 밤하늘/NNG 에/JKB 뜨/VV 어/EC 있/VX 는/ETM 초승달/NNG 같/VA 은/ETM 영화/NNG ./SF

../fasttext skipgram -input ratings_tokenized_khaiii.txt -output model_skipgram

echo “디즈니/NNP” | ../fasttext nn model_skipgram.bin
Query word? 디즈니/NNP 0.991276
애니/NNP 0.845127
즈니/NNG 0.832196
한국애니/NNP 0.809425
일본애니/NNP 0.806225
지브리/NNP 0.77295
드림웍스/NNP 0.756196
지니/NNP 0.745549
베니/NNP 0.740033
쟈니/NNP 0.730029
```

#### 형태소분석 진행한 데이타로 학습(mecab-ko)

```sh
python3 ./unsupervised_nlputils.py --preprocess_mode mecab_tokenize \
    --input_path ./corrected_ratings_corpus.txt \
    --output_path ./ratings_tokenized_mecab.txt

../fasttext skipgram -input ratings_tokenized_mecab.txt -output model_skipgram

echo “디즈니” | ../fasttext nn model_skipgram.bin
Query word? 디즈니 0.996115
픽사 0.77835
드림웍스 0.761766
타잔 0.749565
애니메 0.719357
애니메이션 0.694092
애니 0.691668
월트 0.690366
클레이 0.678788
지브리 0.677055
```

## 형태소분석 결과 비교

```sh
# 형태소분석 없는 데이타
echo “디즈니” | ../fasttext nn model_skipgram.bin
Query word? 디즈니 0.968521
디즈니는 0.956993
디즈니와 0.934998
디즈니의 0.920893
클레이 0.8961
디즈니가 0.889897
함정 0.871816
레전드. 0.864659
쌍벽을 0.86439
걸작중 0.859125
```

`형태소분석 없는 데이타` : 형태소 분석을 하지 않으면 `디즈니` 와 `디즈니는`, `디즈니와` 가 각각 다른 단어로 인식되어 버리는군요.

```sh
# soy_tokenize
echo “디즈니” | ../fasttext nn model_skipgram.bin
Query word? 디즈니 0.994486
픽사 0.707323
애니메이션 0.70667
애니중 0.700826
애니 0.695701
애니의 0.689524
웍스 0.678675
애니를 0.675855
에니메이션 0.672339
2D 0.671045
```

`soy` : `soy` 는 띄어쓰기 통계기반으로 형태소를 판단하는데 꽤 괜찮은 결과를 보이네요.

```sh
# khaiii
echo “디즈니/NNP” | ../fasttext nn model_skipgram.bin
Query word? 디즈니/NNP 0.991276
애니/NNP 0.845127
즈니/NNG 0.832196
한국애니/NNP 0.809425
일본애니/NNP 0.806225
지브리/NNP 0.77295
드림웍스/NNP 0.756196
지니/NNP 0.745549
베니/NNP 0.740033
쟈니/NNP 0.730029
```

`khaiii` : 사용자사전을 많이 등록해야만 올바른 결과가 나오는 듯 합니다.

```sh
# mecab-ko
echo “디즈니” | ../fasttext nn model_skipgram.bin
Query word? 디즈니 0.996115
픽사 0.77835
드림웍스 0.761766
타잔 0.749565
애니메 0.719357
애니메이션 0.694092
애니 0.691668
월트 0.690366
클레이 0.678788
지브리 0.677055
```

`mecab-ko` : 사전파일만 16M 가 넘어서 그런지 꽤 좋은 결과가 나오는군요. 그 와중에 `지브리` 가 나오네요.

## mecab-ko 오분석 처리

mecab-ko 를 형태소 분석기로 정하고 실제 데이타를 분석해 보니 오류가 많이 있습니다.

`파우치`, `셋트` 등이 `파우 치`, `셋 트` 로 분석되어 신조어/외래어에 취약한 모습을 보입니다.

```sh
cd ~/fastText/mywork/
vi get_newword.py
```

```python
import sys, math, argparse, re
import mecab
import hgtk

def word_count(corpus_fname):
    with open(corpus_fname, 'r', encoding='utf-8') as f:
        sentences = f.read()
        words = re.findall("[가-힣]+", sentences)

        # print(words)
        d = {}
        for word in words:
            d[word] = d.get(word, 0) + 1

        # print(d)
        word_freq = []
        for key, value in d.items():
            word_freq.append((value, key))

        # print(word_freq)
        word_freq.sort(reverse=True)
        return word_freq


def check_morphs(lst, corpus_fname, output_fname, log_fname):
    mcab = mecab.MeCab()

    with open(corpus_fname, 'r', encoding='utf-8') as f1, \
         open(output_fname, 'w', encoding='utf-8') as f2, \
         open(log_fname, 'w', encoding='utf-8') as f3:
        sentences = f1.read()

        for item in lst:
            cnt, word = item

            if cnt < 10:
                continue
            tokens = mcab.morphs(word)
            if len(tokens) == 1:
                continue

            words = re.findall(' '.join(tokens), sentences)
            if len(words) < (cnt * 0.05):
                # 형태소 분리된 단어의 빈도수가 분리안된 단어의 빈수도의 5% 미만이면 형태소 분리오류
                (cho, jung, jong) = hgtk.letter.decompose(word[-1])
                if 'ㄱ' <= jong <= 'ㅎ':
                    dic_line = "{},,,,NNP,*,{},{},*,*,*,*,*".format(word, 'T', word)
                else:
                    dic_line = "{},,,,NNP,*,{},{},*,*,*,*,*".format(word, 'F', word)
                # print("{}\t{}\t{}\t{}\t{}".format(word, ' '.join(tokens), cnt, len(words), jong))
                f2.writelines(dic_line + '\n')
                f3.writelines("{}\t{}\t{}\t{}".format(word, ' '.join(tokens), cnt, len(words)) + '\n')


if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--input_path', type=str, help='Location of input files')
    parser.add_argument('--output_path', type=str, help='Location of output files')
    parser.add_argument('--log_path', type=str, help='Location of log files')
    args = parser.parse_args()

    lst = word_count(args.input_path)
    # print(lst)

    check_morphs(lst, args.input_path, args.output_path, args.log_path)
```

```sh
# item_info.txt 은 상품명을 모아놓은 텍스트파일로 직접 구하셔야 합니다.
python3 get_newword.py \
    --input_path item_info.txt \
    --output_path output.txt \
    --log_path log.txt

head output.txt
파우치,,,,NNP,*,F,파우치,*,*,*,*,*
에코백,,,,NNP,*,T,에코백,*,*,*,*,*
크로스백,,,,NNP,*,T,크로스백,*,*,*,*,*
캔들,,,,NNP,*,T,캔들,*,*,*,*,*
백팩,,,,NNP,*,T,백팩,*,*,*,*,*
키링,,,,NNP,*,T,키링,*,*,*,*,*
린넨,,,,NNP,*,T,린넨,*,*,*,*,*
카드지갑,,,,NNP,*,T,카드지갑,*,*,*,*,*
발매트,,,,NNP,*,F,발매트,*,*,*,*,*
정리보관대,,,,NNP,*,F,정리보관대,*,*,*,*,*

cat output.txt >> ~/mecab-ko-dic-2.1.1-20180720/user-dic/nnp.csv

cd ~/mecab-ko-dic-2.1.1-20180720/
./tools/add-userdic.sh
make clean
make
sudo make install
```

**사용자사전 추출을 자동화하기 위해 몇시간 고민해 보았지만, 어차피 사람이 눈으로 확인을 해야 한다는 결론이 나오는군요.**

다른 방식으로 접근해 봤는데요. 키워드는 고객검색어를 이용하는게 가장 낫다는 생각이 드네요.

```sh
python3 ~/embedding/preprocess/unsupervised_nlputils.py \
    --preprocess_mode compute_soy_word_score \
    --input_path ./item_info.txt \
    --model_path ./soyword.model

python3 ~/embedding/preprocess/unsupervised_nlputils.py \
    --preprocess_mode soy_tokenize \
    --input_path ./item_info.txt \
    --model_path ./soyword.model \
    --output_path ./itemname_tokenized_soy.txt

python3 get_frequent_word.py \
    --input_path itemname_tokenized_soy.txt \
    --output_path frequent.txt \
    --log_path log.txt

vi frequent.txt

cp frequent.txt ~/mecab-ko-dic-2.1.1-20180720/user-dic/frequent.csv
cd ~/mecab-ko-dic-2.1.1-20180720/
./tools/add-userdic.sh

# 우선순위 올리기
sed -i -E 's/[0-9]+,NNP/1000,NNP/g' user-frequent.csv

make clean
make
sudo make install
```

## 후기

형태소분석이 많이 중요하군요.

fasttext 는 후기 평점 예측, 고객센터 자동응답, 그리고 멀티 라벨링도 지원하기에 상품 속성 추출에도 적용할만 합니다.

보다 많은 자료는 [여기](https://github.com/facebookresearch/fastText/tree/master/docs) 에서 확인할 수 있습니다.
