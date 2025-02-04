`pytest -h`
##### Что такое тестирование и зачем оно нужно?
Чаще всего говорят о выявлении ошибок, но одной из главных целей является проверка соответствия программы требованиям заказчика.
##### Какие виды тестов вы знаете? Что каждый из видов тестов подразумевает под собой?
- системное тестирование - тестирование всего функционала
- нагрузочное тестирование
- юнит тестирование - то, что пишут сами разрабы
##### Какие библиотеки для написания тестов используются? Что такое фикстура в pytest? Для чего нужен файл conftest.py? Если я ожидаю во время выполнения функцию ошибку, как я могу это протестировать?
Чаще всего (всегда) использую pytest: он хорошо интегрирован с инструментами языка и в целом с библиотеками и фреймворками.
Фикстуры - обеспечение повторяемости тестов. Т.е. постоянная передача аргументов в функции, удаляя тем самым дублирование. Можно трактовать как предопределенное состояние.
Этот файл нужен для того, чтобы отдельно вынести фикстуры, и не импортируя их передавать в аргументы тестов.
##### Что такое мок? Как мокать вызовы функций: простой вызов, вызов с аргументами, количество вызовов.
Подмена реального объекта или его действий для конкретных тестов.
```python
def func():
	return 2

def test_sum(number, mocker):
	mocker.patch("test.func", return_value=10)
	assert 5 + 5 == func()
```