### Описание
Правило ищет различные коды бронирования.

### Примеры найденных сущностей
* Хочу отменить заказ. **Номер бронирования SN1234**
* Ранее был забронирован билет. **Код брони KB4321**

### Инструкция
* Создать новый проект в платформе [PolyAnalyst](https://www.megaputer.ru/produkti/)
* Загрузить в проект файл [**Данные_Код_бронирования.txt**](Данные_Код_бронирования.txt)
* Добавить узел **Извлечение сущностей**, в настройках узла:
	 * отключить стандартные сущности
	 * создать новую пользовательскую сущность
	 * открыть **Редактор правил** новой сущности и скопировать в него правило [**Код_бронирования.txt**](Код_бронирования.txt):
	```
    rule: Код_бронирования
    {
    	query: {phrase(0, synonym(phrase(Код or номер, бронирования or брони or билета or заказа, optional(билета))) or "pnr code",
    	optional(orn("№", ":", "-")), 
    	{length(6, 6, char(alnum|alpha))}:alnum)}:m
    	
    		result: Результат = $m        
    		attribute: Код  = $alnum
    		
    		rule: второй_код
    		{
    			query: phrase(0, $m, и or char(comma), {length(6, 6, char(alnum|alpha))}:alnum2)
    			
    			result: Результат = $alnum2       
    				attribute: Код  = $alnum2
    		}
    }
	```
	 * выполнить узел

### Структура сущности
* **Результат** - общий вывод найденной сущности
* **Код** - код бронирования

### Отчет узла
Отчет узла **Извлечение сущностей** выглядит следующим образом:

| Результат | Код  | 
| ------ | ------|
| Номер бронирования WE4678| WE4678 |

В тексте подсвечиваются следующие фрагменты:
> Хочу узнать статус заказа. **Номер бронирования WE4678**.
