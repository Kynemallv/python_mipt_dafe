
# Домашнее задание 1: Алгоритмы и все, что рядом

В этом семестре мы займемся созданием своего "игрушечного" фреймворка для научных вычислений и анализа данных. Наш фреймворк будет включать утилиты для предобработки данных, алгоритмы решения задач [классификации](http://www.machinelearning.ru/wiki/index.php?title=%D0%9A%D0%BB%D0%B0%D1%81%D1%81%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D1%8F) и [регрессии](http://www.machinelearning.ru/wiki/index.php?title=%D0%A0%D0%B5%D0%B3%D1%80%D0%B5%D1%81%D1%81%D0%B8%D1%8F), функционал для визуализации и оценки качества алгоритмов.

Данное домашнее задание знаменует завершение модуля NumPy и посвящено разработке функционала для предобработки данных, сами алгоритмы классификации и регрессии, а также функционал оценки качества наших алгоритмов.

## Описания

### Предобработка

В качестве предобработки мы будем использовать разбиение наших данных на тренировочную и тестувую выборки. Причны для подобного разбиения были изложены в [семинаре 5](../../lessons/sem2_lesson05/sem5_313/sem5_numpy_practice.ipynb). Вы можете взять функцию разбиения, реализованную в этом семинаре, за основу. Реализованная вами функция должна соответствовать всем характеристикам, описанным в семинаре 5, а также включать в себя параметр `shuffle`, принимающий булево значение. Если параметр `shuffle` принимает значение `True`, вам необходимо вернуть перемешанные данные. Обратите внимание, что при перемешивании данных, вы должны сохранять соответствие описания объекта и его метки, а не просто хаотично перемешивать все массивы, не обращая внимание на взаимные соответсвия меток и точек.

### Алгоритмы

**Непераметрическая регрессия**

Вспомним алгоритм непараметрической регрессии, который мы разбирали на одном из семинаров предыдущего семестра.

В рамках непараметрической регрессии мы пытаемся приблизить значение $y(x)$ некоторой константой $\alpha$ в некоторой окрестности точки $x$. Задача сводится к нахождению данной константы для каждого $x \in X$.

Для того, чтобы отыскать оптимальное значения данной константы, решим следующую оптимизационную задачу:

$Q(\alpha, X^l) = \sum_{i = 1}^{l}{w_i(x)(\alpha - y_i)^2} \rightarrow min_{\alpha \in \mathbb{R}},$ где $w_i(x) = K(\frac{\rho(x, x_i)}{h})$ - вес i-ого квадратичного отклонения, $K(\frac{\rho(x, x_i)}{h})$ - ядро, невозрастающая, ограниченная, гладкая функция, $h$ - ширина окна сглаживания. Фактически мы минимизируем взвешанную сумму квадратов отклонений известных ответов от подобранной константы по $\alpha$.

$$
\frac{d}{d\alpha}Q(\alpha, X^l) = \sum_{i = 1}^{l}{w_i(x)(2\alpha - 2y_i)} = 0;  
\\ \sum_{i = 1}^{l}{\alpha w_i(x)} = \sum_{i = 1}^{l}{y_i w_i(x)} \rightarrow \alpha = \frac{\sum_{i = 1}^{l}{y_i w_i(x)}}{\sum_{i = 1}^{l}{w_i(x)}};
\\ a(x, X^l) = \frac{\sum_{i = 1}^{l}{y_i w_i(x)}}{\sum_{i = 1}^{l}{w_i(x)}} = \frac{\sum_{i = 1}^{l}{y_i K(\frac{\rho(x, x_i)}{h})}}{\sum_{i = 1}^{l}{K(\frac{\rho(x, x_i)}{h})}};
$$  

Таким образом мы получили аппроксимацию $a(x, X^l) = \frac{\sum_{i = 1}^{l}{y_i K(\frac{\rho(x, x_i)}{h})}}{\sum_{i = 1}^{l}{K(\frac{\rho(x, x_i)}{h})}}$, которая позволяет вычислять значение функции в данной точке по имеющимся данным.

В качестве ядра будем использовать так называемое [ядро Епанечникова](https://ru.wikipedia.org/wiki/%D0%AF%D0%B4%D1%80%D0%BE_(%D1%81%D1%82%D0%B0%D1%82%D0%B8%D1%81%D1%82%D0%B8%D0%BA%D0%B0)):
$$
\begin{equation*}
K(x) = 
 \begin{cases}
   \frac{3}{4}(1 - x^2), |x| \le 1\\
   0, |x| > 1
 \end{cases}
\end{equation*}
$$

Ещё одним тонким моментом, связанным с ядром, является ширина окна $h$. Ширина окна - некоторое число, которое можно подобрать руками, или реализовать адаптивный расчет. Т.к. адаптивный расчет является более надёжным методом, не требующим долгой настройки, мы пойдём именно по этому пути. 

Для адаптивного расчета ширины окна необходимо вычислить расстояния от элемента $x$, для которого мы хотим осуществить предсказание, до всех элементов $x_i \in X^l$ обучающей выборки. После чего необходимо упорядочить их по возрастанию расстояния до $x$. Из полученного вариационного ряда выберем k-ый элемент $x_k$. Расстояние $\rho(x, x_k)$ и будет нашей шириной окна, адаптивно вычисляемой для каждого элемента $x \in X$. Может показаться, что мы несильно упростили задачу, ведь теперь вместо выбора $h$ мы будем выбирать $k$. Но, благодаря подобным действиям мы получим лучшие результаты, ведь при данном подходе ширина окна будет своя для кажого $x$.

**Взвешенный KNN для решения задачи классификации**

Принцип, похожий на непараметрическую регрессию также можно использовать для решения задачи классификации. Данный подход называется "Взвешенный KNN". В общих чертах он заключается в следующем:

- Определим $\Rho = \{ \rho(\vec{x}, \vec{x_i}) | \vec{x_i} \in X^l, i = \overline{1,l} \}$ - множество расстояний от точки $\vec{x}$ до точек обучающей выборки;
- Определим $W = \{w_i =  K(\frac{\rho(x, x_i)}{h}) | \vec(x_i) \in X^l, i = \overline{1, l}\}$ - множество весов, рассчитанных для каждой точки обучающей выборки, все обозначения соответствуют обозначениям предыдущего пункта; 
- Упорядочим расстояния по возрастанию: $\rho(\vec{x}, \vec{x_1}) \le ... \le \rho(\vec{x}, \vec{x_l})$;
- Выберем $k$ точек, ближайших в смысле метрики $\rho$ к рассматриваемой точке $\vec{x}$;
- Вычислим величину $label_x = \argmax_{c \in C}{\sum_{i = 1}^k{[y_i = c]w_i}}$, где $C$ - множество классов (в нашем случае множество из двух цветов: красного и синего), $c$ - элемент множества классов, $y_i$ - метка точки $\vec{x_i}$, т.е. цвет точки $\vec{x_i}$, квадратные скобки - функция-индикатор, т.е. функция, которая принимает значение 1, если условие в скобках выполнено, иначе - 0;
- Вычисленная величина $label_x$ и является предсказанием.

### Оценка качества

В качестве оценки качества регрессии реализуем три популярные метрики:

- $MSE (Mean Squared Error) = \frac{1}{n}\sum_{i=1}^n{(y_i - y_i^*)^2}$ - среднеквадратичное отклонение значений, полученных с помощью нашего алгоритма, от исходных значений;

- $MAE (Mean Absolute Error) = \frac{1}{n}\sum_{i=1}^n{|y_i - y_i^*|}$ - среднее абсолютное отклонение значений, полученных с помощью нашего алгоритма, от исходных значений;

- $R^2 = 1 - \frac{\sum_{i=1}^n(y_i - y_i^*)^2}{\sum_{i=1}^n(y_i - \overline{y_i})^2}$ - коэффициент детерминации

В качестве метрики качества классификации реализуем метрику "точность":

- $accuracy = \frac{1}{n}\sum_{i=1}^n{[y_i = prediction_i]}$

## Задание

В данной домашней работе вам необходимо реализовать описанный функционал. 

- Разработайте файловую структуру фреймворка;
- Реализуйте функцию разбиения входных данных на обучающу и тестувую выборки;
- Реализуйте алгоритм непараметрической регресии. В качестве гиперпараметров алгоритм должен принимать k - целое число, индекс расстояния, используемого для адаптивного вычисления окна, а также `metric` - используемая метрика. Доступные значения `l1` и `l2`, т.е. манхэттенская метрика и обычное расстояние между точками. 
- Реализуйте алгоритм взвешенный KNN. В качестве гиперпараметров алгоритм должен принимать k - целое число, количество ближайших соседей, используемых для классификации, а также `metric` - используемая метрика. Доступные значения `l1` и `l2`, т.е. манхэттенская метрика и обычное расстояние между точками;
- Реализуйте описанные метрики качества;

## Требования

- Четкая структура проекта, целесообразность которой вы можете обосновать;
- Оформление кода: ваш код должен проходить проверку линтером flake8, использующим конфиг `.flake8`, который лежит в корне репозитория;
- Реализован весь описанный функционал;
- Все функции написаны с использованием векторизованных функций NumPy без использования циклов; 
- Семинарист должен иметь возможность убедиться в работоспособности вашего кода. Это требование можно выполнить написав скрипт `demo.py`, который будет решать некоторую искусственную задачу, используя весь написанный выми функционал. Также вы можете выполнить это требование, написав юнит-тесты к вашей программе, используя Pytest. Подробнее о работе с Pytest можно прочитать [тут](https://habr.com/ru/articles/448782/);

## Формат и срок сдачи

Срок сдачи работы - 07.04.2024. Работа будет сдаваться очно, перед сдачей работы, необходимо загрузить весь код в ваш репозиторий на GitHub и создать Pull Request в репозиторий курса.