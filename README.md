# soynlp

한국어 분석을 위한 pure python code 입니다. 학습데이터를 이용하지 않으면서 데이터에 존재하는 단어를 찾거나, 문장을 단어열로 분해, 혹은 품사 판별을 할 수 있는 비지도학습 접근법을 지향합니다.

## Guide

### Usage guide

soynlp 에서 제공하는 WordExtractor 나 NounExtractor 는 여러 개의 문서로부터 학습한 통계 정보를 이용하여 작동합니다.
비지도학습 기반 접근법들은 통계적 패턴을 이용하여 단어를 추출하기 때문에 하나의 문장 혹은 문서에서 보다는 어느 정도 규모가 있는 동일한 집단의 문서 (homogeneous documents) 에서 잘 작동합니다.
영화 댓글들이나 하루의 뉴스 기사처럼 같은 단어를 이용하는 집합의 문서만 모아서 Extractors 를 학습하시면 좋습니다.
이질적인 집단의 문서들은 하나로 모아 학습하면 단어가 잘 추출되지 않습니다.


## Setup

    pip install soynlp

## Python version

- Python 3.5+ 를 지원합니다. 3.x 에서 주로 작업을 하기 때문에 3.x 로 이용하시길 권장합니다.
- Python 2.x 는 모든 기능에 대해서 테스트가 끝나지 않았습니다. 

## Requires

- numpy >= 1.12.1
- psutil >= 5.0.1

## Word Extraction 

2016 년 10월의 연예기사 뉴스에는 '트와이스', '아이오아이' 와 같은 단어가 존재합니다. 하지만 말뭉치를 기반으로 학습된 품사 판별기 / 형태소 분석기는 이런 단어를 본 적이 없습니다. 늘 새로운 단어가 만들어지기 때문에 학습하지 못한 단어를 제대로 인식하지 못하는 미등록단어 문제 (out of vocabulry, OOV) 가 발생합니다. 하지만 이 시기에 작성된 여러 개의 연예 뉴스 기사를 읽다보면 '트와이스', '아이오아이' 같은 단어가 등장함을 알 수 있고, 사람은 이를 학습할 수 있습니다. 문서집합에서 자주 등장하는 연속된 단어열을 단어라 정의한다면, 우리는 통계를 이용하여 이를 추출할 수 있습니다. 통계 기반으로 단어(의 경계)를 학습하는 방법은 다양합니다. soynlp는 그 중, Cohesion score, Branching Entropy, Accessor Variety 를 제공합니다. 

    from soynlp.word import WordExtractor

    word_extractor = WordExtractor(min_count=100,
        min_cohesion_forward=0.05, 
        min_right_branching_entropy=0.0)
    word_extractor.train(sentences) # list of str or like
    words = word_extractor.extract()

words 는 Scores 라는 namedtuple 을 value 로 지니는 dict 입니다. 

    words['아이오아이']

    Scores(cohesion_forward=0.30063636035733476,
           cohesion_backward=0,
           left_branching_entropy=0,
           right_branching_entropy=0,
           left_accessor_variety=0,
           right_accessor_variety=0,
           leftside_frequency=270,
           rightside_frequency=0)

2016-10-26 의 뉴스 기사로부터 학습한 단어 점수 (cohesion * branching entropy) 기준으로 정렬한 예시입니다. 

    단어   (빈도수, cohesion, branching entropy)

    촬영     (2222, 1.000, 1.823)
    서울     (25507, 0.657, 2.241)
    들어     (3906, 0.534, 2.262)
    롯데     (1973, 0.999, 1.542)
    한국     (9904, 0.286, 2.729)
    북한     (4954, 0.766, 1.729)
    투자     (4549, 0.630, 1.889)
    떨어     (1453, 0.817, 1.515)
    진행     (8123, 0.516, 1.970)
    얘기     (1157, 0.970, 1.328)
    운영     (4537, 0.592, 1.768)
    프로그램  (2738, 0.719, 1.527)
    클린턴   (2361, 0.751, 1.420)
    뛰어     (927, 0.831, 1.298)
    드라마   (2375, 0.609, 1.606)
    우리     (7458, 0.470, 1.827)
    준비     (1736, 0.639, 1.513)
    루이     (1284, 0.743, 1.354)
    트럼프   (3565, 0.712, 1.355)
    생각     (3963, 0.335, 2.024)
    팬들     (999, 0.626, 1.341)
    산업     (2203, 0.403, 1.769)
    10      (18164, 0.256, 2.210)
    확인     (3575, 0.306, 2.016)
    필요     (3428, 0.635, 1.279)
    문제     (4737, 0.364, 1.808)
    혐의     (2357, 0.962, 0.830)
    평가     (2749, 0.362, 1.787)
    20      (59317, 0.667, 1.171)
    스포츠    (3422, 0.428, 1.604)

자세한 내용은 [word extraction tutorial][wordextraction_lecture] 에 있습니다. 
현재 버전에서 제공하는 기능은 다음과 같습니다. 

## Tokenizer

WordExtractor 로부터 단어 점수를 학습하였다면, 이를 이용하여 단어의 경계를 따라 문장을 단어열로 분해할 수 있습니다. soynlp 는 세 가지 토크나이저를 제공합니다. 띄어쓰기가 잘 되어 있다면 LTokenizer 를 이용할 수 있습니다. 한국어 어절의 구조를 "명사 + 조사" 처럼 "L + [R]" 로 생각합니다. 

### LTokenizer

L parts 에는 명사/동사/형용사/부사가 위치할 수 있습니다. 어절에서 L 만 잘 인식한다면 나머지 부분이 R parts 가 됩니다. LTokenizer 에는 L parts 의 단어 점수를 입력합니다. 

    from soynlp.tokenizer import LTokenizer

    scores = {'데이':0.5, '데이터':0.5, '데이터마이닝':0.5, '공부':0.5, '공부중':0.45}
    tokenizer = LTokenizer(scores=scores)

    sent = '데이터마이닝을 공부한다'

    print(tokenizer.tokenize(sent, flatten=False))
    #[['데이터마이닝', '을'], ['공부', '중이다']]

    print(tokenizer.tokenize(sent))
    # ['데이터마이닝', '을', '공부', '중이다']

### MaxScoreTokenizer

띄어쓰기가 제대로 지켜지지 않은 데이터라면, 문장의 띄어쓰기 기준으로 나뉘어진 단위가 L + [R] 구조라 가정할 수 없습니다. 하지만 사람은 띄어쓰기가 지켜지지 않은 문장에서 익숙한 단어부터 눈에 들어옵니다. 이 과정을 모델로 옮긴 MaxScoreTokenizer 역시 단어 점수를 이용합니다. 

    from soynlp.tokenizer import MaxScoreTokenizer

    scores = {'파스': 0.3, '파스타': 0.7, '좋아요': 0.2, '좋아':0.5}
    tokenizer = MaxScoreTokenizer(scores=scores)

    print(tokenizer.tokenize('난파스타가좋아요'))
    # ['난', '파스타', '가', '좋아', '요']

    print(tokenizer.tokenize('난파스타가 좋아요'), flatten=False)
    # [[('난', 0, 1, 0.0, 1), ('파스타', 1, 4, 0.7, 3),  ('가', 4, 5, 0.0, 1)],
    #  [('좋아', 0, 2, 0.5, 2), ('요', 2, 3, 0.0, 1)]]

LTokenizer 와 MaxScoreTokenizer 에 들어갈 dict[str]=float 의 scores dictionary 는 WordExtractor 로부터 학습된 단어 점수들을 이용하면 됩니다. 혹은 이미 알고 있는 단어들이 있다면, 다른 어떤 단어보다도 더 큰 점수를 부여하면 그 단어는 토크나이저가 하나의 단어로 잘라냅니다. 

### RegexTokenizer

규칙 기반으로도 단어열을 만들 수 있습니다. 언어가 바뀌는 부분에서 우리는 단어의 경계를 인식합니다. 예를 들어 "아이고ㅋㅋㅜㅜ진짜?" 는 [아이고, ㅋㅋ, ㅜㅜ, 진짜, ?]로 쉽게 단어열을 나눕니다. 

    from soynlp.tokenizer import RegexTokenizer

    tokenizer = RegexTokenizer()

    print(tokenizer.tokenize('이렇게연속된문장은잘리지않습니다만'))
    # ['이렇게연속된문장은잘리지않습니다만']

    print(tokenizer.tokenize('숫자123이영어abc에섞여있으면ㅋㅋ잘리겠죠'))
    # ['숫자', '123', '이영어', 'abc', '에섞여있으면', 'ㅋㅋ', '잘리겠죠']

## Noun Extractor

WordExtractor 는 통계를 이용하여 단어의 경계 점수를 학습하는 것일 뿐, 각 단어의 품사를 판단하지는 못합니다. 때로는 각 단어의 품사를 알아야 하는 경우가 있습니다. 또한 다른 품사보다도 명사에서 새로운 단어가 가장 많이 만들어집니다. 명사의 오른쪽에는 -은, -는, -라는, -하는 처럼 특정 글자들이 자주 등장합니다. 문서의 어절 (띄어쓰기 기준 유닛)에서 왼쪽에 위치한 substring 의 오른쪽에 어떤 글자들이 등장하는지 분포를 살펴보면 명사인지 아닌지 판단할 수 있습니다. soynlp 에서는 두 가지 종류의 명사 추출기를 제공합니다. 둘 모두 개발 단계이기 때문에 어떤 것이 더 우수하다 말하기는 어렵습니다만, NewsNounExtractor 가 좀 더 많은 기능을 포함하고 있습니다. 추후, 명사 추출기는 하나의 클래스로 정리될 예정입니다. 

### Noun Extractor ver 1 & News Noun Extractor

    from soynlp.noun import LRNounExtractor
    noun_extractor = LRNounExtractor()
    nouns = noun_extractor.train_extract(sentences) # list of str like

    from soynlp.noun import NewsNounExtractor
    noun_extractor = NewsNounExtractor()
    nouns = noun_extractor.train_extract(sentences) # list of str like

2016-10-20 의 뉴스로부터 학습한 명사의 예시입니다. 

    덴마크  웃돈  너무너무너무  가락동  매뉴얼  지도교수
    전망치  강구  언니들  신산업  기뢰전  노스
    할리우드  플라자  불법조업  월스트리트저널  2022년  불허
    고씨  어플  1987년  불씨  적기  레스
    스퀘어  충당금  건축물  뉴질랜드  사각  하나씩
    근대  투자주체별  4위  태권  네트웍스  모바일게임
    연동  런칭  만성  손질  제작법  현실화
    오해영  심사위원들  단점  부장조리  차관급  게시물
    인터폰  원화  단기간  편곡  무산  외국인들
    세무조사  석유화학  워킹  원피스  서장  공범

더 자세한 설명은 [튜토리얼][nounextraction-v1_usage]에 있습니다. 

### Noun Extractor ver 2

soynlp=0.0.46+ 에서는 명사 추출기 version 2 를 제공합니다. 이전 버전의 명사 추출의 정확성과 합성명사 인식 능력, 출력되는 정보의 오류를 수정한 버전입니다. 사용법은 version 1 과 비슷합니다.

    from soynlp.utils import DoublespaceLineCorpus
    from soynlp.noun import LRNounExtractor_v2

    corpus_path = '2016-10-20-news'
    sents = DoublespaceLineCorpus(corpus_path, iter_sent=True)

    noun_extractor = LRNounExtractor_v2(verbose=True)
    nouns = noun_extractor.train_extract(sents)

추출된 nouns 는 {str:namedtuple} 형식입니다. 

    print(nouns['뉴스']) # NounScore(frequency=4319, score=1.0)

_compounds_components 에는 복합명사를 구성하는 단일명사들의 정보가 저장되어 있습니다. '대한민국', '녹색성장'과 같이 실제로는 복합형태소이지만, 단일 명사로 이용되는 경우는 단일 명사로 인식합니다.

    list(noun_extractor._compounds_components.items())[:5]

    # [('잠수함발사탄도미사일', ('잠수함', '발사', '탄도미사일')),
    #  ('미사일대응능력위원회', ('미사일', '대응', '능력', '위원회')),
    #  ('글로벌녹색성장연구소', ('글로벌', '녹색성장', '연구소')),
    #  ('시카고옵션거래소', ('시카고', '옵션', '거래소')),
    #  ('대한민국특수임무유공', ('대한민국', '특수', '임무', '유공')),

LRGraph 는 학습된 corpus 에 등장한 어절의 L-R 구조를 저장하고 있습니다. get_r 과 get_l 을 이용하여 이를 확인할 수 있습니다.

    noun_extractor.lrgraph.get_r('아이오아이')

    # [('', 123),
    #  ('의', 47),
    #  ('는', 40),
    #  ('와', 18),
    #  ('가', 18),
    #  ('에', 7),
    #  ('에게', 6),
    #  ('까지', 2),
    #  ('랑', 2),
    #  ('부터', 1)]

더 자세한 설명은 [튜토리얼 2][nounextractor-v2_usage]에 있습니다.

## Part of Speech Tagger

단어 사전이 잘 구축되어 있다면, 이를 이용하여 사전 기반 품사 판별기를 만들 수 있습니다. 단, 형태소분석을 하는 것이 아니기 때문에 '하는', '하다', '하고'는 모두 동사에 해당합니다. Lemmatizer 는 현재 개발/정리 중입니다. 

    pos_dict = {
        'Adverb': {'너무', '매우'}, 
        'Noun': {'너무너무너무', '아이오아이', '아이', '노래', '오', '이', '고양'},
        'Josa': {'는', '의', '이다', '입니다', '이', '이는', '를', '라', '라는'},
        'Verb': {'하는', '하다', '하고'},
        'Adjective': {'예쁜', '예쁘다'},
        'Exclamation': {'우와'}    
    }

    from soynlp.pos import Dictionary
    from soynlp.pos import LRTemplateMatcher
    from soynlp.pos import LREvaluator
    from soynlp.pos import SimpleTagger
    from soynlp.pos import UnknowLRPostprocessor

    dictionary = Dictionary(pos_dict)
    generator = LRTemplateMatcher(dictionary)    
    evaluator = LREvaluator()
    postprocessor = UnknowLRPostprocessor()
    tagger = SimpleTagger(generator, evaluator, postprocessor)

    sent = '너무너무너무는아이오아이의노래입니다!!'
    print(tagger.tag(sent))
    # [('너무너무너무', 'Noun'),
    #  ('는', 'Josa'),
    #  ('아이오아이', 'Noun'),
    #  ('의', 'Josa'),
    #  ('노래', 'Noun'),
    #  ('입니다', 'Josa'),
    #  ('!!', None)]

더 자세한 사용법은 [사용법 튜토리얼][tagger_usage] 에 기술되어 있으며, [개발과정 노트][tagger_lecture]는 여기에 기술되어 있습니다. 

## Vetorizer

토크나이저를 학습하거나, 혹은 학습된 토크나이저를 이용하여 문서를 sparse matrix 로 만듭니다. minimum / maximum of term frequency / document frequency 를 조절할 수 있습니다. Verbose mode 에서는 현재의 벡터라이징 상황을 print 합니다. 

    vectorizer = BaseVectorizer(
        tokenizer=tokenizer,
        min_tf=0,
        max_tf=10000,
        min_df=0,
        max_df=1.0,
        stopwords=None,
        lowercase=True,
        verbose=True
    )

    corpus.iter_sent = False
    x = vectorizer.fit_transform(corpus)

문서의 크기가 크거나, 곧바로 sparse matrix 를 이용할 것이 아니라면 이를 메모리에 올리지 않고 그대로 파일로 저장할 수 있습니다. fit_to_file() 혹은 to_file() 함수는 하나의 문서에 대한 term frequency vector 를 얻는대로 파일에 기록합니다. BaseVectorizer 에서 이용할 수 있는 parameters 는 동일합니다.

    vectorizer = BaseVectorizer(min_tf=1, tokenizer=tokenizer)
    corpus.iter_sent = False

    matrix_path = 'YOURS'
    vectorizer.fit_to_file(corpus, matrix_path)

하나의 문서를 sparse matrix 가 아닌 list of int 로 출력이 가능합니다. 이 때 vectorizer.vocabulary_ 에 학습되지 않은 단어는 encoding 이 되지 않습니다.

    vectorizer.encode_a_doc_to_bow('오늘 뉴스는 이것이 전부다')
    # {3: 1, 258: 1, 428: 1, 1814: 1}

list of int 는 list of str 로 decoding 이 가능합니다.

    vectorizer.decode_from_bow({3: 1, 258: 1, 428: 1, 1814: 1})
    # {'뉴스': 1, '는': 1, '오늘': 1, '이것이': 1}

dict 형식의 bag of words 로도 encoding 이 가능합니다. 

    vectorizer.encode_a_doc_to_list('오늘의 뉴스는 매우 심각합니다')
    # [258, 4, 428, 3, 333]

dict 형식의 bag of words 는 decoding 이 가능합니다.

    vectorizer.decode_from_list([258, 4, 428, 3, 333])
    ['오늘', '의', '뉴스', '는', '매우']

## Normalizer

대화 데이터, 댓글 데이터에 등장하는 반복되는 이모티콘의 정리 및 한글, 혹은 텍스트만 남기기 위한 함수를 제공합니다. 

    from soynlp.normalizer import *

    emoticon_normalize('ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ쿠ㅜㅜㅜㅜㅜㅜ', n_repeats=3)
    # 'ㅋㅋㅋㅜㅜㅜ'

    repeat_normalize('와하하하하하하하하하핫', n_repeats=2)
    # '와하하핫'

    only_hangle('가나다ㅏㅑㅓㅋㅋ쿠ㅜㅜㅜabcd123!!아핫')
    # '가나다ㅏㅑㅓㅋㅋ쿠ㅜㅜㅜ 아핫'

    only_hangle_number('가나다ㅏㅑㅓㅋㅋ쿠ㅜㅜㅜabcd123!!아핫')
    # '가나다ㅏㅑㅓㅋㅋ쿠ㅜㅜㅜ 123 아핫'

    only_text('가나다ㅏㅑㅓㅋㅋ쿠ㅜㅜㅜabcd123!!아핫')
    # '가나다ㅏㅑㅓㅋㅋ쿠ㅜㅜㅜabcd123!!아핫'


## notes

### Slides

- [slide files][unkornlp_pdf]에 알고리즘들의 원리 및 설명을 적어뒀습니다. 데이터야놀자에서 발표했던 자료입니다.
- [textmining tutorial][textmining-tutorial] 을 만들고 있습니다. soynlp project 에서 구현 중인 알고리즘들의 설명 및 텍스트 마이닝에 이용되는 머신 러닝 방법들을 설명하는 slides 입니다. 

### Blogs

- [github io blog][lovitio] 에서 [slides][textmining-tutorial] 에 있는 내용들의 텍스트 설명 글들을 올리고 있습니다. Slides 의 내용에 대해 더 자세하게 보고 싶으실 때 읽으시길 권합니다. 


## 함께 이용하면 좋은 라이브러리들

### soyspacing

띄어쓰기 오류가 있을 경우 이를 제거하면 텍스트 분석이 쉬워질 수 있습니다. 분석하려는 데이터를 기반으로 띄어쓰기 엔진을 학습하고, 이를 이용하여 띄어쓰기 오류를 교정합니다. 

- https://github.com/lovit/soyspacing
- pip install soyspacing

### KR-WordRank

토크나이저나 단어 추출기를 학습할 필요없이, HITS algorithm 을 이용하여 substring graph 에서 키워드를 추출합니다. 

- https://github.com/lovit/KR-WordRank
- pip install krwordrank

### soykeyword

키워드 추출기입니다. Logistic Regression 을 이용하는 모델과 통계 기반 모델, 두 종류의 키워드 추출기를 제공합니다. scipy.sparse 의 sparse matrix 형식과 텍스트 파일 형식을 지원합니다. 

- https://github.com/lovit/soykeyword
- pip install soykeyword


[wordextraction_lecture]: tutorials/wordextractor_lecture.ipynb
[nounextraction-v1_usage]: tutorials/nounextraction-v1_usage.ipynb
[nounextractor-v2_usage]: tutorials/nounextractor-v2_usage.ipynb
[tagger_usage]: tutorials/tagger_usage.ipynb
[tagger_lecture]: tutorials/tagger_lecture.ipynb
[unkornlp_pdf]: notes/unskonlp.pdf
[textmining-tutorial]: https://github.com/lovit/textmining-tutorial
[lovitio]: https://lovit.github.io/
