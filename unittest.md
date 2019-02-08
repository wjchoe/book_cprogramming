# 인터페이스과 플러그인을 분리하는 프로그래밍

## 유닛테스트를 위한 프레임웍 만들기

인터페이스와 플러그인의 활용을 가장 분명하면서도 쉽게 설명해주는 예제중 하나가 유닛테스트 프레임웍입니다.
cstring이라는 라이브러리를 만들었으면 내가 생각한대로 동작하는지 테스트를 해야겠지요.
간단한 main 함수를 만들어서 몇가지 동작을 테스트해볼 수 있을것입니다.

하지만 라이브러리 규모가 커지면서 10개 혹은 100개의 라이브러리를 만들었을때도 각 라이브러리마다 혹은 함수마다 main 함수를 만들 수 있을까요?
만들수는 있겠지만, main함수가 100개있는 프로젝트를 생각해보세요.
소스 구조가 엄청나게 복잡해질 것입니다.

유닛테스트 프레임웍은 각각의 유닛테스트를 등록할 수 있도록 인터페이스를 제공하고, 모아진 유닛테스트들을 한번에 실행해주는 기능을 가집니다.
사용자는 인터페이스에맞게 등록만하고, 프로젝트를 빌드하면 자동으로 유닛테스트 전체를 실행하는 실행파일이 만들어지게됩니다.

아래 그림은 유닛테스트가 대략 어떤 형태를 가지게되는지 보여줍니다.

![unittest interface and dummy plugin](/unittest_dummy.png)

유닛테스트는 2개의 파일로 이루어져있습니다.
1. unittest.c: 프레임웍 구현체, 개별적인 유닛테스트를 등록하는 인터페이스 제공
2. unittest_main.c: 등록된 유닛테스트를 실행

unittest.c에서 2개의 인터페이스를 제공합니다.
1. struct unittest: 유닛테스트의 실행에 필요한 3개의 함수 포인터와 1개의 데이터(void 타입 포인터)가 인터페이스입니다.
1. DEFINE_UNITTEST: 유닛테스트의 이름과 struct unittest를 정의한 객체를 유닛테스트 프레임웍에 저장하는 함수입니다.

unittest_dummy.c 파일은 가상의 dummy라는 라이브러리를 가정해서 dummy라는 라이브러리의 유닛테스트를 등록하는 파일입니다.
유닛테스트를 어떻게 사용하는지를 보여주기 위해 만들었습니다.

먼저 unittest_dummy.c 파일을 보고 어떻게 유닛테스트 프레임웍을 사용하는건지 알아보겠습니다.
```
#include <stdio.h>
#include <stdlib.h>
#include "unittest.h"


struct priv_data {
	int val;
};

static struct priv_data pdata;


int test_dummy_init(void *priv)
{
	struct priv_data *data = priv;
	printf("\ttest_dummy_init\n");
	data->val = 0xa5a5;
	return 0;
}

int test_dummy_final(void *priv)
{
	struct priv_data *data = priv;
	data->val = 0x0;
	printf("\ttest_dummy_final\n");
	return 0;
}

int test_dummy_run(void *priv)
{
	struct priv_data *data = priv;
	printf("\ttest_dummy_run: val=%x\n", data->val);
	return 0;
}

struct unittest test_dummy = {
	.init = test_dummy_init,
	.final = test_dummy_final,
	.run = test_dummy_run,
	.priv = &pdata,
};

DEFINE_UNITTEST(dummy, test_dummy)
```

먼저 struct unittest test_dummy