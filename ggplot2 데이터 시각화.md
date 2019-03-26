# ggplot2 데이터 시각화

### *ggplot2 활용 R로 데이터 시각화*

#### 자~! 이제 시작이야~! 내꿈을 ~! 내꿈을 위한여행~! 피카츄~! T4IR 화이팅! 

`ggplot2` 팩키지에 포함된 함수만 다룰 예정이다. R에는 시각화 관련 강력한 내장 기능을 포함하고 있지만, 금번 실습에는 `ggplot2` 팩키지만 다룰 예정이다. 이를 통해 구조화되고 정제된 데이터에 대한 고급 정보를 제공하는 시각화 산출물 생성이 가능하다.

> ### 학습목표
>
> 이번 학습단원을 마치게 되면, 학습자는 다음 작업을 수행할 수 있는 경험치를 얻게 된다.:
>
> - `ggplot2` 팩키지를 사용해서 데이터를 효과적으로 시각화하는 방법을 익히게 된다.
> - 복잡성 높은 시각화 산출물을 단계별 접근법을 통해 생성하는 방법을 익히게 된다.
> - 산점도, 상자그림, 시계열 그래프를 생성할 수 있게 된다.
> - `facet_wrap`, `facet_grid` 명령어를 사용해서 요인 변수로 쪼개진 데이터에 대해 시각화해서 집합으로 묶어 보일 수 있다.
> - 목적에 맞춰 맞춤형 그래프 스타일을 적용한 시각적 산출물을 생성할 수 있다.

시각화에 필요한 팩키지를 적재한다.

```R
# 시각화 팩키지
library(ggplot2)
# 가장 최신 데이터프레임 조작 팩키지
library(dplyr)
## Warning: package 'dplyr' was built under R version 3.2.5
## 
## Attaching package: 'dplyr'
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

## `ggplot2` 활용 시각화

`ggplot2` 팩키지를 활용해서 동일한 그래프를 생성하게 된다.

`ggplot2`는 시각화 그래프 팩키지로 데이터프레임에 존재하는 데이터에서 복잡한 그래프 생성을 간단히 만들 수 있게 해준다. 기본디폴트 설정이 잘 갖춰져서 최소한의 설정과 약간의 튜닝으로 출판수준 품질 시각화 산출물을 빠른 시간내에 만들어 내는데 도움이 된다.

`ggplot`는 신규 요소를 하나씩 하나씩 덧씌우는 방식으로 그래프를 생성해 나간다.

`ggplot` 그래프를 생성하려면 다음 순서로 진행한다:

- `data` 인자를 사용해서 특정 데이터프레임과 그래프 플롯을 단단히 묶는다.

```r
ggplot(data = surveys_complete)
```

- 미학요소(aesthetics, `aes`)를 정의해서 데이터의 변수와 그래프 플롯의 축 혹은 크기, 형태, 색생등을 매핑한다.

```r
ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length))
```

- `geoms` 을 추가한다 – 그래프 플롯에 있는 데이터의 그래픽 표현(점, 선, 막대). 그래프 플롯에 `geom`을 추가하려면 `+` 연산자를 사용한다:

```R
ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length)) +
  geom_point()
```

![image](https://user-images.githubusercontent.com/46669551/54423315-b9687880-4753-11e9-9228-b97275ab174e.png)

`ggplot2` 팩키지에 `+` 연산자가 유용한데 이유는 이미 존재하는 `ggplot` 객체를 변형시킬 수 있기 때문이다. 이런 사실이 의미를 갖는 이유는 쉽게 “템플릿” 그래프 플롯을 생성하고 나서 다른 그래프 유형을 편리하게 탐색할 수 있게 해준다. 따라서 앞선 그래프 플롯은 다음과 같은 코드로 생성될 수 있다.

```R
# 그래프 플롯 생성
surveys_plot <- ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length))

# 그래프 화면 출력
surveys_plot + geom_point()
```

주목:

- `ggplot()` 함수내부에 넣은 어떤 것이든 모든 추가한 `geom` 계층에서도 볼 수 있다. (즉, 가장 보편적인 그래프 플롯 설정이 된다). `aes()` 내부에 설정한 `x`, `y`축도 포함된다.
- 물론 `ggplot()` 함수에서 전역으로 정의한 미학요소와 독립적으로 해당 `geom`에 미학요소를 설정할 수도 있다.

## 주고받으며 인터랙티브하게 그래프 플롯 생성

`ggplot`으로 그래프 플롯을 생성하는 과정은 일반적으로 반복적인 과정을 경험하게 된다. 데이터셋을 정의하는 것부터 시작해서, 축을 설정하고 `geom`을 고르게 된다.

```R
ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length)) +
    geom_point()
```

![image](https://user-images.githubusercontent.com/46669551/54423361-dbfa9180-4753-11e9-8bcb-c35b432032f8.png)

그리고 나서, 먼저 그린 그래프에서 더 많은 정보가 추출되도록 그래프를 변형한다. 예를 들어, 그래프가 겹치는 것을 방지하고자 투명도(alpha)를 추가한다.

```R
ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length)) +
    geom_point(alpha = 0.1)
```

![image](https://user-images.githubusercontent.com/46669551/54423399-f3d21580-4753-11e9-9d93-8f9d63cf9fc8.png)

다른 색상을 모든 점에 입힐 수도 있다.

```R
ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length)) +
    geom_point(alpha = 0.1, color = "blue")
```

![image](https://user-images.githubusercontent.com/46669551/54423435-02203180-4754-11e9-8499-5c4b2bcd9691.png)

혹은 각 종족마다 다른 색상을 입힐 수도 있다:

```R
ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length)) +
    geom_point(alpha = 0.1, aes(color=species_id))
```

![image](https://user-images.githubusercontent.com/46669551/54423471-16642e80-4754-11e9-8db9-0b1a274a4768.png)

## 상자그림(Boxplot)

각 종마다 무게에 대한 분포를 시각화한다.

```R
ggplot(data = surveys_complete, aes(x = species_id, y = hindfoot_length)) +
    geom_boxplot()
```

![image](https://user-images.githubusercontent.com/46669551/54423500-2845d180-4754-11e9-9eec-74cc3692a7ca.png)

상자그림에 점을 추가해서, 무게를 측정한 개체수와 분포에 대해 더 좋은 이해를 얻을 수 있다:

```R
ggplot(data = surveys_complete, aes(x = species_id, y = hindfoot_length)) +
    geom_boxplot(alpha = 0) +
    geom_jitter(alpha = 0.3, color = "tomato")
```

![image](https://user-images.githubusercontent.com/46669551/54423541-3ac00b00-4754-11e9-8520-18c28914b089.png)

어떻게 하면 상자그림 지터 계층 뒤에 위치시킬 수 있을까? 어떤 조치를 취해야만 상자그림을 점들 위에 위치시켜서 숨겨지지 않게 만들 수 있을까요.

> ### 도전과제
>
> 상자그림은 유용한 요약이지만, 분포의 *형태(shape)*가 숨겨진다. 예를 들어 이봉(bimodal) 분포를 갖는 경우, 상자그림에는 관측되지 않는다. 상자그림 대안으로 바이올린 그림(콩그림, beanplot이라고도 부름)이 있는데, 점 분포가 형태로 그려진다.
>
> - 상자그림을 바이올린 그림으로 교체한다; `geom_violin()` 참조
>
> 많은 자료유형에서, 관측점 *척도(scale)*를 고려하는 것이 중요하다. 예를 들어, 축의 척도를 변경시키게 되면 그래프 공간에 관측점 분포를 더 잘 표현할 수 있다. 축의 척도를 변경하는 것은 다른 구성요소를 추가/변경해서 가능하다. (즉, 명령어를 차근 차근 추가해 나감)
>
> - log10 척초로 체중을 표현시킨다; `scale_y_log10()` 참조.
> - `hindfoot_length`에 대한 상자그림을 생성하라.
> - 표본이 추출된 것(`plot_id`)에 따라 상자그림에 데이터점을 추가한다.

힌트: `plot_id`에 대한 자료형 클래스를 확인한다. `plot_id` 자료형을 정수형에서 요인형으로 변경하는 것을 생각해 본다. 왜 이런 자료형 변경이 R로 하여금 그래프를 생성방법을 바뀌게 한다고 생각하는가?

## 시계열 데이터 시각화

각 종족별로 매년 개체수를 계산해본다. 이 작업을 수행하려면, 먼저 데이터를 집단으로 묶고나서 각 집단마다 레코드(개체수)를 개수한다.

```R
yearly_counts <- surveys_complete %>%
                 group_by(year, species_id) %>%
                 tally
```

시간흐름에 따른 데이터는 `x`축에 시간, `y`축에 개체수를 두고 선그래프(line plot)로 시각화한다.

```R
ggplot(data = yearly_counts, aes(x = year, y = n)) +
     geom_line()
```

![image](https://user-images.githubusercontent.com/46669551/54423585-55927f80-4754-11e9-91d4-bcf0121fcd7a.png)

불행하게도, 이렇게 도식화하면 원하는 바가 산출되지 않는다. 이유는 모든 종족에 대해서 데이터를 시각화하기 때문이다. `ggplot`에 각 종족마다 따라 선그래프가 그려지도록 미학요소(`aes`) 함수에 `group = species_id`을 포함시킨다.

```R
ggplot(data = yearly_counts, aes(x = year, y = n, group = species_id)) +
    geom_line()
```

![image](https://user-images.githubusercontent.com/46669551/54423615-6642f580-4754-11e9-96b1-080644a2dfca.png)

그래프에 색상을 추가하면 종족을 구분하기 쉽다.

```R
ggplot(data = yearly_counts, aes(x = year, y = n, group = species_id, colour = species_id)) +
    geom_line()
```

![image](https://user-images.githubusercontent.com/46669551/54423640-74911180-4754-11e9-97a2-2062b48e4f59.png)

## 측면보기(Faceting)

`ggpot`에는 **측면보기(Faceting)**로 불리는 특수 기능이 내장되어 있는데, 이를 통해서 그래프 하나를 데이터셋에 포함된 요인에 따라 다수 그래프로 쪼갤 수 있다. 이 기능을 사용해서 각 종마다 시계열적 변화를 그림 한장으로 표현할 수 있다.

```R
ggplot(data = yearly_counts, aes(x = year, y = n, group = species_id, colour = species_id)) +
    geom_line() +
    facet_wrap(~ species_id)
```

![image](https://user-images.githubusercontent.com/46669551/54423666-85da1e00-4754-11e9-9eb2-35147721dc13.png)

이제 각 그래프마다 그려진 선을 측정된 성별로 분리해보자. 이 작업을 위해서 `year`, `species_id`, `sex`로 집단을 묶어 개체수를 합산한다:

```R
 yearly_sex_counts <- surveys_complete %>%
                      group_by(year, species_id, sex) %>%
                      tally
```

이런 작업을 통해 (하나의 그림안에) 추가적으로 성별에 따라 쪼갤 수 있다:

```R
 ggplot(data = yearly_sex_counts, aes(x = year, y = n, color = species_id, group = sex)) +
     geom_line() +
     facet_wrap(~ species_id)
```

![image](https://user-images.githubusercontent.com/46669551/54423688-968a9400-4754-11e9-9119-18cb9a18cfc1.png)

일반적으로 그래프 플롯은 프린터로 출력했을 경우 흰색 배경이 더 가독성이 좋다. `theme_bw()` 함수를 사용해서 배경을 흰색으로 설정한다. 부가적으로 격자 그리드(grid)도 제거한다.

```R
 ggplot(data = yearly_sex_counts, aes(x = year, y = n, color = species_id, group = sex)) +
     geom_line() +
     facet_wrap(~ species_id) +
     theme_bw() +
     theme(panel.grid.major.x = element_blank(), 
       panel.grid.minor.x = element_blank(),
       panel.grid.major.y = element_blank(),
       panel.grid.minor.y = element_blank())
```

![image](https://user-images.githubusercontent.com/46669551/54423728-b326cc00-4754-11e9-9324-8d573905d643.png)

그래프 플롯을 더 읽기 좋도록 만들려면, 종족에 따른 것이 아니라 성별에 따라 색상을 달리한다. (종족은 이미 별도 작은 그래프 플롯으로 구분되어 있어 추가로 더 구별할 필요는 없다).

```R
ggplot(data = yearly_sex_counts, aes(x = year, y = n, color = sex, group = sex)) +
    geom_line() +
    facet_wrap(~ species_id) +
    theme_bw()
```

![image](https://user-images.githubusercontent.com/46669551/54423755-c5086f00-4754-11e9-82fa-a3c400f59fd6.png)

## 도전 과제

> 방금 전에 학습한 것을 바탕으로 연도별로 각 종별로 평균 무게가 어떻게 변화되는지 시각화하여 도식화하시오.

`facet_wrap` 을 사용해서 임의 차원을 추출해서 한 페이지에 맞춰 깔끔하게 도식화할 수 있다. 다른 한편으로 `facet_grid`를 사용해서 명시적으로 어떤 하위 그래프 플롯을 정렬할지 공식 표기법을 사용해서 지정할 수 있다. (공식표기법: `행 ~ 열`; `.` 기호는 자리차지 연산자로 열 혹은 행만 표현한다.)

각 종족별로 숫컷과 암컷 체중이 시간에 따라 어떻게 변화했는지 비교할 수 있도록 앞선 그래프 플롯을 변경해보자.

```R
## 열 하나, 행별 측면보기(facet)
yearly_sex_weight <- surveys_complete %>%
    group_by(year, sex, species_id) %>%
    summarize(avg_weight = mean(weight))
ggplot(data = yearly_sex_weight, aes(x=year, y=avg_weight, color = species_id, group = species_id)) +
    geom_line() +
    facet_grid(sex ~ .)
```

![image](https://user-images.githubusercontent.com/46669551/54423801-d782a880-4754-11e9-9296-0c1823d0904e.png)

```R
# 행 하나, 열별 측면보기(facet)
ggplot(data = yearly_sex_weight, aes(x=year, y=avg_weight, color = species_id, group = species_id)) +
    geom_line() +
    facet_grid(. ~ sex)
```

![image](https://user-images.githubusercontent.com/46669551/54423827-e9fce200-4754-11e9-804a-22040ca5522d.png)

## 사용자 기능정의(Customization)

`ggplot2` 커닝 쪽지 ([https://www.rstudio.com/wp-content/uploads/2015/08/ggplot2-cheatsheet.pdf)를](https://www.rstudio.com/wp-content/uploads/2015/08/ggplot2-cheatsheet.pdf)살펴보고 나서, 앞서 시각화한 그래프를 개선시킬 수 있는 방법을 생각해 본다. Etherpad에 아이디어를 기술해서 적어본다.

이제, 작업한 그래프에 그래프 제목을 추가하고, 축 제목을 ‘year’, ‘n’ 보다 좀더 유의미하고 많은 정보를 제공하는 제목을 붙여본다:

```R
ggplot(data = yearly_sex_counts, aes(x = year, y = n, color = sex, group = sex)) +
    geom_line() +
    facet_wrap(~ species_id) +
    labs(title = 'Observed species in time',
         x = 'Year of observation',
         y = 'Number of species') +
    theme_bw()
```

![image](https://user-images.githubusercontent.com/46669551/54423912-13b60900-4755-11e9-9478-1e384d01f48d.png)

축에 좀더 도움이 되는 명칭이 붙었지만, 글꼴 크기를 키워 가독성을 높일 수 있다. 여기까지 왔으면, 글꼴도 변경해 본다:

```R
ggplot(data = yearly_sex_counts, aes(x = year, y = n, color = sex, group = sex)) +
    geom_line() +
    facet_wrap(~ species_id) +
    labs(title = 'Observed species in time',
        x = 'Year of observation',
        y = 'Number of species') +
    theme_bw() +
    theme(text=element_text(size=16, family="Arial"))
```

![image](https://user-images.githubusercontent.com/46669551/54423937-2597ac00-4755-11e9-93c9-c16c9717a8c7.png)

여기까지 조작한 뒤에도 `x`-축 값을 읽기에는 어려움이 있는게 보인다. 라벨 방향을 변경시켜 서로 겹치지 않도록 수직/수평으로 조정한다. 90도 회전시키거나, 대각방향으로 적절히 돌려 나름 최선의 방향을 탐색해 본다.

```R
ggplot(data = yearly_sex_counts, aes(x = year, y = n, color = sex, group = sex)) +
    geom_line() +
    facet_wrap(~ species_id) +
    labs(title = 'Observed species in time',
        x = 'Year of observation',
        y = 'Number of species') +
    theme_bw() +
    theme(axis.text.x = element_text(colour="grey20", size=12, angle=90, hjust=.5, vjust=.5),
                        axis.text.y = element_text(colour="grey20", size=12),
          text=element_text(size=16, family="Arial"))
```

![image](https://user-images.githubusercontent.com/46669551/54423966-37794f00-4755-11e9-8495-18fbc717449f.png)

지금까지 작업한 것에 만족해서 이를 기본디폴트 설정으로 삼고자 한다면, 객체로 저장해서 다른 그래프에 아래와 같이 쉽게 적용시킬 수 있다:

```R
arial_grey_theme <- theme(axis.text.x = element_text(colour="grey20", size=12, angle=90, hjust=.5, vjust=.5),
                          axis.text.y = element_text(colour="grey20", size=12),
                          text=element_text(size=16, family="Arial"))
ggplot(surveys_complete, aes(x = species_id, y = hindfoot_length)) +
    geom_boxplot() +
    arial_grey_theme
```

![image](https://user-images.githubusercontent.com/46669551/54423999-4bbd4c00-4755-11e9-9865-7cd4371a54b3.png)

지금까지 작업한 모든 지식을 바탕으로, 5분 시간을 갖고 예제로 생성시킨 그래프 플롯 혹은 스스로 작업한 그래프를 개선시켜 보자. RStudio ggplot2 컨닝쪽지를 사용한다.

아래에 몇가지 그래프를 개선시킬 아이디어가 나와 있다:

- 선의 굵기를 변경하면 어떨까?
- 범례 명칭을 바꾸는 것은 어떨까? 라벨을 바꾸는 것은 어떨까?
- 다른 색상 팔레트를 사용하는 것은 어떨까? (<http://www.cookbook-r.com/Graphs/Colors_(ggplot2)/> 참조)

그래프를 생성한 다음에, 원하는 그림파일 형식으로 파일로 떨궈 저장한다. 적절한 인자(`width`, `height`, `dpi`)를 조정해서 그래프 플롯의 차원(해상도)도 쉽게 변경할 수 있다:

```R
my_plot <- ggplot(data = yearly_sex_counts, aes(x = year, y = n, color = sex, group = sex)) +
    geom_line() +
    facet_wrap(~ species_id) +
    labs(title = 'Observed species in time',
        x = 'Year of observation',
        y = 'Number of species') +
    theme_bw() +
    theme(axis.text.x = element_text(colour="grey20", size=12, angle=90, hjust=.5, vjust=.5),
                        axis.text.y = element_text(colour="grey20", size=12),
          text=element_text(size=16, family="Arial"))
ggsave("name_of_file.png", my_plot, width=15, height=10)
```