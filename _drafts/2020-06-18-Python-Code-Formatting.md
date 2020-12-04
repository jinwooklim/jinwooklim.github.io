---
title:  "Clean Code in Python: Code Formatting and Tools"
excerpt: "Python에서의 Code Formatting과 Tools에 대해 알아보자"

categories:
  - Development
tags:
  - Python
comments : true
---

# Intro
---
글 작성을 하기 전에, 이 글은 **"Clean Code in Python: Refactor Your Legacy Code Base"**를 참조하였음을 알려드립니다.



이 글에서는 클린 코드를 작성하기 위한 네 가지 내용을 말하려고 합니다.

1. 클린 코드는 Formatting 이상의 훨씬 중요한 것을 의미한다.

2. 표준 Formatting을 유지하는 것이 유지보수성의 핵심 유의사항이다.

3. Python에서는, 파이썬이 제공하는 기능을 사용하여, 자체 문서화된 코드를 작성할 수 있다.

4. 코드의 레이아웃을 일정하게 유지하여, 팀원들이 문제에만 집중할 수 있게 도구를 설정할 수 있다.  



&nbsp;
# 클린 코드의 의미
---

클린 코드라는 한 가지 유일하고, 정해진 정의는 없습니다. 또한 이를 측정할 방법도 없습니다. 따라서 코드가 얼마나 좋은지 나쁜지, 유지보수가 가능한지를 알려줄 수 있는 도구는 없는 것입니다. 물론, 문법을 체크하는 **checker**, 취약한 부분을 찾아내는 **linter**, 코드를 분석하기 위해 static analyzer를 사용할 수 있습니다.

하지만, 이런 방법만으로는 클린 코드를 작성하기에 부족한 점이 있습니다. 왜냐하면, 프로그래밍 언어는 컴퓨터에게 사람의 생각을 전달하는 것은 물론, 다른 사람(엔지니어)에게 전달하는 목적도 가지고 있기 때문입니다. 즉, 내가 작성한 코드가 클린 코드인지 아닌지는 다른 엔지니어가 코드를 읽고 유지 관리할수 있는지 여부에 달려 있는 것입니다.

따라서, 클린 코드가 무엇인지는 스스로 좋은 코드와 나쁜 코드의 차이점을 확인하고, 자신만의 정의를 하는 것이 좋습니다.



&nbsp;
# 클린 코드의 중요성
---

클린 코드는 maintainability(유지보수성) 향상, technical debt(기술 부채) 감소, agile development를 통한 효과적인 진행 그리고 성공적인 프로젝트 관리에 이어지므로 중요하다.

* techincal debt: 개발팀이 코드를 수정하고 리팩토링하기 위해 멈추는 것은 technical debt에 대한 비용을 지불하는 것임. (debt, 즉 부채이므로 이자가 생기며, 나중에 큰 문제가 된다.)

  

### 클린 코드에서 code formatting의 역할  
클린 코드란 무엇일까 ?

**클린 코드 != PEP-8, 프로젝트 가이드라인, 표준 지침** 이다.

클린 코드는 코딩 표준, formatting, linting 도구, 다른 검사 도구들을 사용한 코드 레이아웃 설정 등 그 이상을 말한다. 즉, PEP-8을 만족한다고 해서 코드가 클린 코드의 요건을 모두 충족하는 것은 아니다. PEP-8은 코드를 올바르게 formatting 하는데에 효율성을 높여주는 것이다.

  

### 프로젝트 코딩 스타일 가이드 준수  
코딩 가이드라인은 코드의 품질 표준을 지키기 위해 프로젝트에서 따라야하는 최소한의 요구사항이다.

좋은 코드 레이아웃에서 필요한 특성은 일관성이다. 코드가 일관되게 구조화 되어 있으면 가독성이 높아진다.

특히, 파이썬에서는 PEP-8을 따르거나 이를 부분적으로 확장한 것(ex: 한 줄의 길이)을 따르는 것이 좋다. (다른 언어들과 달리 파이썬의 구문을 고려하여 작성되었기 때문)

**PEP-8의 특징**

* **Grepability**
  * `grep -nr "location=" .`
    `./core.py:13: location=current_location,`
  * 위와 같은 방법을 이용하여,  location 파라미터의 위치를 쉽게 찾을 수 있다.
  * `grep -nr "location =" .`
    `./core.py:10: current_location = get_location()`
  * 위와 같은 방법을 이용하여, current_location에 변수가 할당되는 것을 쉽게 찾을 수 있다.
* **Consistency**
  
  * 쉽게 읽을 수 있다.
* **Code quality**
  * 쉽게 버그를 찾을 수 있다.



&nbsp;
# Docstring과 Annotations
---

#### Documentation

**코드를 문서화 하는 것 != 주석을 추가하는 것**

comment(주석)을 다는 것보다, 문서화를 통해 데이터 타입이 무엇인지 설명하고, 예제를 작성하는 것이 더욱 바람직하다.

* 주석은 코드로 아이디어를 제대로 표현하지 못했음을 나타내는 것임
* 코드의 변경과 함께 주석이 변경되지 않는다면, 위험을 초래할 수 있다.

#### Annotation

특히, 파이썬의 경우 dynamically type(동적 타입)을 사용하기 때문에, 함수나 메소드를 거치면 변수나 객체의 값이 무엇인지 알기가 어렵습니다. 이럴 때에는 annotation을 통해 명시하게 되면, 개발을 함께 하는데 있어 도움이 됩니다. 또한, annotation을 사용하면 Mypy와 같은 도구를 사용해서 type hint와 같은 자동화된 검증을 실행할 수 있습니다.

  

### Docstring

Docstring은 소스코드에 포함된 문서입니다. docstring은 기본적으로 literal string이고, 코드의 일부분에 배치됩니다. 즉, docstring은 코드의 특정 부분(module, class, method or function)에 대한 문서화이다.

**사용 예**

* 함수 뒤에 `??` 붙여쓰기 (`dict.update??`)
* 객체의 경우 attribute에  `__doc__`을 이용

Docstring을 도와주는 tool로는 **Sphinx, autodoc, sphinx.ext.autodoc**이 있다.

위와 같은 문서화 도구를 사용하기 위해서는, 모든 팀원이 참여할 수 있게 해야한다. 또한 지속적으로 코드가 변경될때마다 수정해야하는 수작업을 해야한다.

  

### Annotation

PEP-3107에서 annotation을 소개하고 있다. 이는 코드 사용자에게 함수 인자로 어떤 값이 와야하는지 힌트를 주자는 것이다. 즉 type hinting 이다.

**사용 예**

* ```python
  class Point:
      def __init__(self, lat, long): 
      self.lat = lat
      self.long = long 
      
  def locate(latitude: float, longitude: float) -> Point:
      """Find an object in the map by its coordinates"""
  ```

* `locate.__annotations__`
  `{'latitude': float, 'longitue': float, 'return': __main__.Point}`

PEP-484를 이용하면, annotation을 통해 코드를 확인할 수 있다. 즉, type hinting의 기본 원리를 이용해 함수의 타입을 체크할 수 있다.

* **type hinting :** interpreter와 추가적인 도구를 사용해서, 코드 전체에 올바른 타입이 사용되었는지 확인하고, 호환된지 않는 타입이있을 때 사용자에게 힌트를 주는 것.

  


### 기본 도구를 설정하기

#### Mypy를 이용한 type hinting

* `pip install mypy`를 이용하여 설치할 수 있다.
* `mypy filename`을 이용하여 타입 검사 결과를 제공한다.
* `type_to_ignore = "something"  # type : ignore` 를 이용해서 해당 line을 무시할 수 있다.

  

#### Pylint를 사용한 코드 검사

pycodestyle, pylint, flake8, autopep8 등 도구가 있다.

* `pip install pylint` 를 이용하여 설치할 수 있다.

    
  

#### 자동 검사 설정

linux 개발 환경에서 빌드를 자동화 하는 방법은 makefile을 사용하는 것임.

**사용 예**

* ```
  typehint:
  mypy src/ tests/
  test:
  pytest tests/
  lint:
  pylint src/ tests/
  checklist: lint typehint test
  .PHONY: typehint test lint checklist
  ```

* `make checklist` 실행 하는 방법
  PEP-8 검사, type 검사, test 실행이 모두 이루어진다.

**Black**이라는 PEP-8 이상의 엄격한 포매팅 도구도 있다.