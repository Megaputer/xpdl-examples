
### Описание
Правило ищет срок действия договора.

### Примеры найденных сущностей
* Настоящий договор **действует до 31.12.2045**
* **Срок действия** договора **10 лет**

### Инструкция
* Создать новый проект на платформе [PolyAnalyst](https://www.megaputer.ru/produkti/)
* Загрузить в проект файл [**Данные_срок_действия.txt**](Данные_срок_действия.txt)
* Добавить узел **Извлечение сущностей**, в настройках узла:
	* отключить все стандартные сущности, кроме сущности **Dates** (Даты)
	* выполнить узел
* Добавить еще один узел **Извлечение сущностей**, в настройках узла:
	 * отключить стандартные сущности
	 * создать новую пользовательскую сущность
	 * открыть **Редактор правил** новой сущности и скопировать в него правило [**Срок_действия.txt**](Срок_действия.txt):
	 ```
	rule: срок действия
	{
		  query: {phrase(3, действует or "срок действия", 
					
							optional("в течение"),
							
							{phrase(number(), год or месяц or неделя or день)}:длит 
							or 
							phrase(до, {entity(Dates)}:оконч)
							
						)}:m

		  result: Результат = title($m)
				attribute: Длительность = $длит
				attribute: Дата окончания = toentity(Dates, $оконч, field:=DateTime)

	}  
	```
	 * выполнить узел

### Структура сущности
* **Результат** - общий вывод найденной сущности
* **Длительность** - срок действия договора
*  **Дата окончания** - дата окончания действия договора

### Отчет узла
Отчет узла **Извлечение сущностей** выглядит следующим образом:

| Результат| Длительность| Дата окончания  |
| ------ | ------ | ------ |
| Действует до 01.11.2015 | |Ноябрь 1, 2015|
| Срок действия 25 лет | 25 лет ||

В тексте подсвечиваются следующие фрагменты:
> Настоящий Договор вступает в силу с момента его государственной регистрации и **действует до 01.11.2015**.
**Срок действия** Настоящего Договора **25 лет**.
