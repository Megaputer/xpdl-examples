
### Описание
Правило ищет тип договора.

### Примеры найденных сущностей
* **Договор возмездного оказания услуг** № 124/A
* **Договор об оказании услуг** № 14T07

### Инструкция
* Создать новый проект в платформе [PolyAnalyst](https://www.megaputer.ru/produkti/)
* Загрузить в проект файл [**Данные_Тип_Договора.txt**](Данные_Тип_Договора.txt)
* Добавить узел **Извлечение сущностей**, в настройках узла:
	 * отключить стандартные сущности
	 * создать новую пользовательскую сущность
	 * открыть **Редактор правил** новой сущности и скопировать в него правило [**Тип_Договора.txt**](Тип_Договора.txt):
	```
	rule: тип_договора
	{
		// Тип договора указывается в самом начале документа, поэтому используется функция position(). Позиция распространяется на все эелементы искомой фразы, т.е. упоминание типа договора будет находится не далее 8 слов от начала документа.
	    query: {position(8, phrase(договор, optional(orn(о, об, на)), 
		{repeat(1,5, lemma(adjective|noun))}:type))}:contract // от 1 до 5 повторов прилагательных и/или существительных: ПОДРЯДА, ВОЗМЕЗДНОГО ОКАЗАНИЯ УСЛУГ
		
	    result: Результат = $contract
	        attribute: Тип договора = $type

	}
	```
	 * выполнить узел

### Структура сущности
* **Результат** - общий вывод найденной сущности
* **Тип** - тип договора

### Отчет узла
Отчет узла **Извлечение сущностей** выглядит следующим образом:

| Результат |Тип |
| ------ |------ |
| Договор возмездного оказания услуг|возмездного оказания услуг |

В тексте подсвечиваются следующие фрагменты:
> **Договор возмездного оказания услуг** № 124/A