# tc_tagger

- tc-tagger의 목적은 인스타그램 텍스트의 특징에 맞는 한국어 자연어처리 패키지를 만드는 것입니다.
- 인스타그램 텍스트는 다른 텍스트와 비교했을 때에 아래와 같은 특징을 가집니다.
    1. 해쉬태그가 중요한 역할을 합니다. 해쉬태그가 달려 있는 텍스트가 전체 텍스트에서 가장 중요한 정보들을 담고 있을 가능성이 높습니다.
    2. 이모티콘을 많이 포함합니다. 이모티콘은 포스트를 올린 사람의 감정상태를 나타내는 중요한 정보입니다.
    3. 다른 텍스트에서 문장부호를 활용하는 방식과는 다른 방식으로 문장부호를 많이 활용합니다. 보통 다른 텍스트들에서 '.', '?', '!' 과 같은 문장부호들은 문장의 종결 그리고 문장유형을 보여주는 의미로 쓰는 경우가 많습니다.  또한 이러한 문장부호들은 각 문장에서 한번만 쓰이는 경우가 많습니다. 그러나 인스타그램 텍스트에서는 이러한 문장부호들이 문장의 종결이나 문장유형을 드러내는 데에 활용된다기 보다는 자신의 감정 등을 드러내는 데에 활용됩니다. 또한 문장부호를 여러 개 활용하여 하나의 단위로 활용하는 경우가 많습니다.
- 따라서 기존 형태소 분석기들을 인스타그램 텍스트에 활용하려고 하면 아래와 같은 제한 사항들이 발생합니다.
    1. 해쉬태그 표시를 단순하게 하나의 부호로 처리하거나 해쉬태그 뒤의 텍스트들을 과분석하는 문제점이 발생합니다.
    2. 이모티콘을 처리하는 태그가 따로 존재하지 않습니다.
    3. 문장부호 여러 개가 조합되어 하나의 의미를 가지는 단위가 되어도 이를 쪼개서 분석합니다.
- 따라서 tc-tagger는 위의 제한 사항들을 어느 정도 해결할 수 있는 인스타그램에 적합한 새로운 형태소 분석기를 만드는 것을 목적으로 합니다.

## Guideline

### Install

```powershell
$ git clone https://github.com/HyeonminNam/tc_tagger.git
```

### Requires

```powershell
$ pip install Konlpy
$ pip install emoji
```

- Konlpy ≥ 0.5.0
    - 본 패키지는 Konlpy에 포함된 okt 클래스를 기반으로 제작되었습니다. okt 클래스는 Konlpy 0.5.0 버전 이후부터 포함되어 있기 때문에 0.5.0 버전 이후의 Konlpy를 설치해 주세요.
- emoji ≥ 0.6.0

### Python

- Python 3.7 버전을 기반으로 구현하였습니다.
- Python 3.x 버전 이상을 사용하시기를 권장합니다.

## Usage

### Preprocessing

- tc-tagger는 아래와 같이 인스타그램 텍스트에 특화된 전처리 기능을 제공합니다.
    - 이모티콘, 문장부호, Hashtag 정보를 텍스트에서 제거하는 기능
    - 텍스트에 포함된 Hashtag 정보(#), 사용자 아이디 정보(@)를 추출하는 기능
    - 인스타그램 텍스트의 띄어쓰기를 교정하는 기능(PyKospacing 패키지 활용)

### Tagger

- tc-tagger의 tagger는 아래와 같은 특징을 가집니다.
    - 이모티콘에 대한 독립된 태그('Emoji')를 제공합니다.
    - Hashtag 뒤의 단어들은 형태소 분석과 더불어서 Hashtag 정보임을 보여주는 태그를 붙여줍니다.
    - 인스타그램에서 자주 활용되는 신조어들을 사전에 반영하였습니다.
- 사용 예시

```python
from tc_tagger.tag import Threecow()
text1 = '럽스타 그자체❤❤\n#럽스타그램 #운동하는커플 #태닝'
text2 = '이지부스트 신은 연영과 학생'
threecow = TC_tagger()
for text in [text1, text2]:
    print(threecow.tagger(text))
    print(threecow.tokenizer(text))
```

- 출력 결과
    - 이모티콘은 각각이 모두 'Emoji'로 태깅되었음을 알 수 있습니다.
    - 하나의 해쉬태그당 하나의 tuple이 배당되었으며 해쉬태그 tuple 내에 다시 형태소 분석이 이루어지고 있음을 확인할 수 있습니다.
    - 인스타그램에서 자주 쓰이는 고유명사, 신조어를 하나의 단어로 태깅하는 것을 확인할 수 있습니다.

```python
# text1 tagger 출력값
[('럽스타', 'Noun'), ('그', 'Determiner'), ('자체', 'Noun'), ('❤_red_heart', 'Emoji'), ('❤_red_heart', 'Emoji'), ('\n', 'Foreign'), (('럽스타그램', 'Hashtag_Noun'),), (('운동', 'Hashtag_Noun'), ('하는', 'Hashtag_Verb'), ('커플', 'Hashtag_Noun')), (('태닝', 'Hashtag_Noun'),)]
# text1 tokenizer 출력값
['럽스타', '그', '자체', '❤_red_heart', '❤_red_heart', '\n', '럽스타그램', '운동', '하는', '커플', '태닝']
# text2 tagger 출력값
[('이지부스트', 'Noun'), ('신은', 'Verb'), ('연영과', 'Noun'), ('학생', 'Noun')]
# text2 tokenizer 출력값
['이지부스트', '신은', '연영과', '학생']
```
