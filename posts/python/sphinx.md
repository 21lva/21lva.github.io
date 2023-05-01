---
layout: post
title:  "Sphinx"
date:   2019-08-10 02:02:59
author: Inhyuk
category: Python
tags: Python
cover:  "/assets/instacode.png"
name: sphinx.md
---

Sphinx
======

- Python 코드내에 들어간 docstring을 자동으로 문서화해주는 도구.
- 문서화 : 함수에 대한 정보를 작성(하는 일, 입력 값, 출력 값 등)

시작
---


```
#프로젝트 구조

sphinx-example
├── main.py
└── package1
    ├── Math.py
    ├── __init__.py
    └── hello.py
```

- 위와 같은 간단한 파이썬 프로젝트를 이용한다

```
#설치
pip install sphinx

#시작
cd sphinx-example
sphinx-quickstart doc

#빌드
./make html
```

- seperate 옵션에서 **y** 를 선택하면 **source** 에 rst등의 설정파일들이, **build** 에 html과 같은 출력 파일이 따로 저장된다. **n** 을 선택하면 최상위 루트(현재는 doc)에 source파일들이 저장되고 **_build**에 출력 파일들이 저장된다.

```
#sphinx 설치 후 구조

sphinx-example
├── doc
│   ├── Makefile
│   ├── build
│   ├── make.bat
│   └── source
│       ├── _static
│       ├── _templates
│       ├── conf.py
│       └── index.rst
├── main.py
└── package1
    ├── Math.py
    ├── __init__.py
    └── hello.py
```

- - -

설정
---

- **source** 의 **conf.py** 를 수정해서 설정한다


```
# 버전 연동

import os
import sys
sys.path.insert(0, os.path.abspath('../..'))

from main import __version__ as VERSION


project = 'Ex'
copyright = '2021, ih'
author = 'ih'

release = VERSION

# autodoc 사용
extensions = [
    'sphinx.ext.autodoc'
]
```

- **autodoc** 를 사용하기 위해서는 **extensions** 를 알맞게 수정해야 한다
  - autodoc: 파이썬 코드로부터 자동으로 문서를 생성해준다


- - -

생성
-----

```
# rst 파일 생성
sphinx-apidoc -e -o doc/source .
```

- **-e** 옵션: source와 build를 분리해서 출력한다

```
# rst 생성 후 프로젝트 구조

sphinx-example
├── doc
│   ├── Makefile
│   ├── build
│   ├── make.bat
│   └── source
│       ├── _static
│       ├── _templates
│       ├── conf.py
│       ├── index.rst
│       ├── main.rst
│       ├── modules.rst
│       ├── package1.Math.rst
│       ├── package1.hello.rst
│       └── package1.rst
├── main.py
└── package1
    ├── Math.py
    ├── __init__.py
    └── hello.py
```

- index.rst에 modules.rst에 대한 정보가 없다면 다음과 같이 수정해준다

```
.. toctree::
   :maxdepth: 2
   :caption: Contents:

   modules
```

- - -

테스트
----

- 테스트를 해보려면 doc 폴더로 이동하여 빌드한다

```
cd doc
make html
```


![화면]({{site.baseurl}}/post_img/{{page.name}}/sphinx1)

- - -

테마
-----

- [테마 목록](https://sphinx-themes.org/)
- built in 테마는 conf.py를 수정해서 사용가능하다.

```
# -- Options for HTML output -------------------------------------------------

# The theme to use for HTML and HTML Help pages.  See the documentation for
# a list of builtin themes.
#
html_theme = 'classic'
```

- - -

문법
------

#### 설명

- **param:** 와 **return:** 을 이용하면 입력과 출력값을 보여줄 수 있다.

```
def say_hello(x:str):
    """
    :param x: 문자열 변수
    :return void: 없어!
    """
    print("hello "+x)
```


- - -

참고 자료
=======

- https://tech.ssut.me/start-python-documentation-using-sphinx/
- https://soma0sd.tistory.com/132
- http://www.hanul93.com/python-sphinx/
