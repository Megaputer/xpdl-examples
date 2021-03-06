### Описание
Правило ищет упоминания налогов и акцизов.

### Примеры найденных сущностей
* в следующем году будет увеличен **земельный налог**
* поступления в бюджет по **налогу на прибыль организаций** составили 10 млн рублей

### Инструкция
* Создать новый проект в платформе [PolyAnalyst](https://www.megaputer.ru/produkti/)
* Загрузить в проект файл [**Данные_Налог.txt**](Данные_Налог.txt)
* Добавить узел **Извлечение сущностей**, в настройках узла:
	 * отключить стандартные сущности
	 * создать новую пользовательскую сущность
	 * открыть **Редактор правил** новой сущности и скопировать в него правило [**Налог.txt**](Налог.txt):
	```
	rule: налог
	{
		//Ищем ключевые слова НАЛОГ или АКЦИЗ
		query: {налог or акциз}:keyword
		
		//Далее ищем названия налогов и акцизов в разных вариантах написания
		
		//Пример: налог на имущество физических лиц
		rule: налог_на
		{
			query: {phrase($keyword, на,								
							optional(lemma(noun)), 	    				
							optional(lemma(adjective)),   				
							lemma(noun)								
						)}:nalog

			result: Налог = title(norm($nalog))      
		}
		
		//Пример: транспортный налог
		rule: прилагательное_налог
		{
			query: {phrase(orn(земельный, транспортный, водный), $keyword)}:nalog	

			result: Налог = title(norm($nalog))      
		}
		
		//Пример: акциз по подакцизным товарам
		rule: акциз_по
		{
			query: {phrase($keyword, по, подакцизные, товары)}:nalog 	

			result: Налог = title(norm($nalog))      
		}
	}
	```
	 * выполнить узел

### Структура сущности
* **Налог** - название налога или акциза

### Отчет узла
Отчет узла **Извлечение сущностей** выглядит следующим образом:

| Налог | 
| ------ | 
| Транспортный налог|
| Акциз на импорт автомобилей| 
 
В тексте подсвечиваются следующие фрагменты:
>В Госдуму РФ был внесен законопроект об отмене **транспортного налога** в 2021 г.
 Узбекистан с 1 августа отменяет **акциз на импорт автомобилей**.