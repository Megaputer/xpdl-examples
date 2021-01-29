### Описание
Правило ищет номер и дату авиарейса.

### Примеры найденных сущностей
* **Рейс FV 1000** по направлению Москва - Рига **6 мая**
* **10 мая: рейс SU 2000** по направлению Рига - Москва

### Инструкция
* Создать новый проект в платформе [PolyAnalyst](https://www.megaputer.ru/produkti/)
* Загрузить в проект файл [**Данные_Номер_рейса.txt**](Данные_Номер_рейса.txt)
* Добавить узел **Извлечение сущностей**, в настройках узла:
	* отключить все стандартные сущности, кроме сущности **Dates** (Даты)
	* выполнить узел
* Добавить еще один узел **Извлечение сущностей**, в настройках узла:
	 * отключить стандартные сущности
	 * создать новую пользовательскую сущность
	 * открыть **Редактор правил** новой сущности и скопировать в него правило [**Номер_рейса.txt**](Номер_рейса.txt):
	 ```
    /* Макрос для поиска номера рейса. */
    macro: flight_number () = orn(phrase(0, length(2, 2, char(latin)), length(3, 4, char(digit))), 
    						      regex("[A-z][A-z]\-?\d+", wholeword:=yes))
    /* Макрос для исключения ложных номеров. */
    macro: wrong_flight () = orn(phrase("и", обратно), возвращаться)
    /* Макрос для исключения ложных дат. */
    macro: wrong_date () = orn(сообщить, перебронировать, запланированный, приобрести, уведомление)
    						  
    rule: flight_number
    {
        /* Ищем ключевые слова и номер рейса. */
    	query: {orn(phrase(3, optional(рейс|"номер рейса"), {macro(flight_number)}:number),
    		   phrase(3, рейс|"номер рейса", {length(3, 4, char(digit))}:number))}:m
    		   
    	result: Результат = $m        
    		attribute: Номер рейса = $number
    	
    	/* Ищем дату рейса, расположенную после номера. */
    	rule: date_after
    	{
    		query: {phrase(6, $m, not(entity(Dates, Day:>0)|macro(wrong_flight)|phrase(")", ",")), {entity(Dates, Day:>0)}:date)
    				or
    				phrase($m, not(phrase(")", ",")), {entity(Dates, Day:>0)}:date)}:m_date
    		
    		/* Исключаем ложные результаты. */
    		rule_except: not_date
    		{
    			query: phrase($m_date, phrase(5, $m, not(entity(Dates, Day:>0))))
    				   or
    				   phrase($m_date, entity(Dates, Day:>0))
    			
    			result: Результат = $m_date        
    				attribute: Номер рейса = $number
    				attribute: День = toentity(Dates, field:=Day, $date)
    				attribute: Месяц = toentity(Dates, field:=Month, $date)
    				attribute: Год = toentity(Dates, field:=Year, $date)
    		}
    	}
    	
    	/* Ищем дату рейса, расположенную до номера. */
    	rule: date_before
    	{
    		query: {phrase(5, {entity(Dates, Day:>0)}:date, not("и"), "(", $m)
    				or
    				phrase(7, {entity(Dates, Day:>0)}:date,
    				not(";"|macro(wrong_flight)|macro(wrong_date)|entity(Dates, Day:>0)), phrase(2, $m, not(entity(Dates, Day:>0))))
    				or
    				phrase({entity(Dates, Day:>0)}:date, ",", $m, allow_punct:=yes)}:m_date
    		
    		/* Исключаем ложные результаты. */
    		rule_except: not_date
    		{
    			query: phrase(3, $m, optional("-"), $m_date, allow_punct:=no)
    				   or
    				   phrase(entity(Dates, Day:>0), "/", $m_date)
    				   or
    				   ($m_date & phrase(char(digit), "и", char(digit), term(ee_months)))
    			
    			result: Результат = $m_date        
    				attribute: Номер рейса = $number
    				attribute: День = toentity(Dates, field:=Day, $date)
    				attribute: Месяц = toentity(Dates, field:=Month, $date)
    				attribute: Год = toentity(Dates, field:=Year, $date)
    		}
    	}	
    }
  
	```
	ИЛИ
  * импортировать файл [**Номер_рейса.xml**](Номер_рейса.xml), в котором дополнительно настроена постобработка (создание колонок **Номер рейса и дата**, **Дата перелета**, **Год** и нормализация номера рейса)
  * выполнить узел

### Структура сущности
* **Результат** - общий вывод найденной сущности
* **Номер рейса и дата** - номер авиарейса и дата
* **Номер рейса** - номер авиарейса
* **Дата перелета** - дата авиарейса
* **День** - дата рейса: день
* **Месяц** - дата рейса: месяц
* **Год** - дата рейса: год

### Отчет узла
Отчет узла **Извлечение сущностей** выглядит следующим образом:

| Результат| Номер рейса и дата |Номер рейса|Дата перелета |День| Месяц| Год| 
| ------ | ------ |------ |------ |------ |------ |------ |
| рейса SU 111 01 января 2021 | SU 111 (1.01.2021) | SU 111| 01.01.2021|1|01| 2021|

В тексте подсвечиваются следующие фрагменты:
> Уведомляем Вас об отмене **рейса SU 111** Лос-Анджелес Интернешнл - Шереметьево, **01 января 2021**.