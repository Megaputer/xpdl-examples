
### Описание
Правило ищет различные мероприятия, а также места их проведения и организаторов/участиников.

### Примеры найденных сущностей
* в **Томске в Ярмарке образовательных услуг принял участие Юргинский технологический институт**
* **компания «БЕЛЛОНА» провела семинар «ЭКО-ЮРИСТ»** в **Брянске**

### Инструкция
* Создать новый проект в платформе [PolyAnalyst](https://www.megaputer.ru/produkti/)
* Загрузить в проект файл [**Данные_Мероприятия_Усложненные.txt**](Данные_Мероприятия_Усложненные.txt)
* Добавить узел **Извлечение сущностей**, в настройках узла:
	* отключить все стандартные сущности, кроме сущностей **Companies** (Компании), **Organizations** (Организации) и **GeoAdministrative** (Географические объекты)
	* выполнить узел
* Добавить еще один узел **Извлечение сущностей**, в настройках узла:
	 * отключить стандартные сущности
	 * создать новую пользовательскую сущность
	 * открыть **Редактор правил** новой сущности и скопировать в него правило [**Мероприятия_Усложненные.txt**](Мероприятия_Усложненные.txt):
	 ```
	macro:event() = orn(акция, мероприятие, "мастер-класс", празднование, фестиваль)

	//Поиск текстов, где встречаются ключевые компоненты: Компании, Организации, Географические объекты
	rule: events

	{query: {entity(Companies|Organizations)}:org
			 or {entity(GeoAdministrative, Category:=country)}:country
			 or {entity(GeoAdministrative, Category:=region)}:region		 
			 or {entity(GeoAdministrative, Category:=city)}:city

		
		//Поиск мероприятий, упомянутых после компании-участника или компании-организатора в одной и той же части предложения
		//Пример: компания «БЕЛЛОНА» провела семинар «ЭКО-ЮРИСТ» в Брянске
		rule: after_org
		{query: {phrase(5,  $org
							, phrase(3, phrase(0, not([его]|[её]), проводит|проводил|"принимает участие в"|"принимает участие во"|"был проведен", not(качестве))  										/* Поиск ключевых глаголов и глагольных фраз */
							, optional($city|$country|$region)																									  										/* Поиск места, где было проведено мероприятие */
							, optional(phrase("в", term(ee_months), optional(phrase(length(4,4, char(digit)), год))))																					/* Поиск даты мероприятия */
							, optional($city|$country|$region)																																			/* Поиск места, где было проведено мероприятие */
							, optional(phrase("для"|[с], term(ee_professions)|"студентов и сотрудников"|"выпускников"))																					/* Поиск людей, для которых было проведено мероприятие */
							, optional(phrase(0, optional(lemma(adjective)), мероприятие|событие, "-"))																									/* Поиск дополнительного контекста */	
							, not(char(comma)|"исследование"|скрининг)																																	/* Исключение некорректных контекстов */
							, {																																											/* Поиск мероприятий в различных написаниях: */																		
							  (phrase(0,  repeat(1,6,lemma(noun|adjective|NumeralOrdinal)|[по]|term(ee_roman_num)),																						//- Пятая международная конференция по вопросам образования
										  optional(phrase(0, [по]|[среди]|[на]|[в]|[для]|"-", optional(вопросам|проблемам), repeat(1,6, lemma(noun|verb_participle|adjective))/orn(время, встреча))
										  or
										  phrase(0, char(dq|sq),repeat(1,6, lemma(pronoun|noun|adjective|verb|preposition)|":"|[и]), char(dq|sq))))														//- Пятая международная конференция "Вопросы образования"											
							  /orn("@", phrase(length(4,4, char(digit)), год), www, неделя, скрининг, создание, [выходные], опрос, исследование, "во время", entity(GeoAdministrative))					/* Исключение некорректных результатов */
							  )
							  or
							  phrase(0, char(dq|sq),repeat(1,5, lemma(pronoun|noun|adjective|verb|preposition)|[среди]|":"), char(dq|sq))																//- Семинар «ЭКО-ЮРИСТ»
							  or
							  phrase(чемпионат, $region, optional(phrase(0, среди, repeat(1,3, lemma(noun)))))																							//- Чемпионат Москвы среди студентов																					
							  }:event		
							, optional($city|$country|$region))																																			/* Поиск места, где было проведено мероприятие */
						 )
			 	}:m
			
			//Исключение некорректных результатов
			rule_except: negative 	
			{query: include($m:event, term(ee_months, ee_professions)|lemma(работа, опрос, тема, память, коллега)|"ребята"|[co]|"д."|phrase(lemma(adjc|numeral), год)|"наши ученые"|"образовательное учреждение"|"первое"|"по вопросам"|lemma(adjc|numeral)|length(1,1, char(alpha))|phrase(lemma(preposition|adjc), lemma(adjc)))
					//в российско-американском эксперименте 'SIRI...
					or phrase(0, char(sq), $m:event, not(char(sq)))
					or phrase(2, "будут проведены на", $m:event)
					or ($m:event&orn("ремонт в здании", "при поддержке", "провел в конце", "опрос населения", "на протяжении", "трансфер на площадку", путин, выпускники, победители))

				result: Результат = $m
					attribute: Компания = toentity(Companies|Organizations, $org, field:=Name)
					attribute: Мероприятие = norm($event)
					attribute: Город = toentity(GeoAdministrative, $city, field:=Name)
					attribute: Регион = toentity(GeoAdministrative, $region, field:=Name)
					attribute: Страна = toentity(GeoAdministrative, $country, field:=Name)	
								
			}	
		}
		
		//Поиск мероприятий и компаний-участников или компаний-организаторов, которые находятся в разных частях предложения
		//Пример: скоро откроется первый фестиваль Fashion Laboratory, который проводит Сибирский центр дизайна
		rule: in_clause
		{query: {phrase(5, phrase(0, {phrase(phrase(0, optional(школа|"мастер-класс"| фестиваль), optional(lemma(noun))), lemma(noun))}:event, char(comma)), 
						"которые проведут"|" которой приняли участие", 
						$org, allow_punct:=no)}:m
				
			//Поиск места проведения мероприятия
			//Пример: в Томске откроется первый фестиваль Fashion Laboratory, который проводит Сибирский центр дизайна
			rule: geo
			{query: {sentence($m, phrase(0, not(администрации), $city|$country|$region))		
					or $m}:m1
			
			//Исключение некорректных результатов
			rule_except: negative		
			{query: phrase(0, lemma(genitive, $m1:event), char(comma), [которые])
			
				result: Результат = $m1
					attribute: Компания = toentity(Companies|Organizations, $org, field:=Name)
					attribute: Мероприятие = norm($event)
					attribute: Город = toentity(GeoAdministrative, $city, field:=Name)
					attribute: Регион = toentity(GeoAdministrative, $region, field:=Name)
					attribute: Страна = toentity(GeoAdministrative, $country, field:=Name)		
			}
			}
		}
		
		//Поиск мероприятий, упомянутых до компании-участника или компании-организатора в одной и той же части предложения
		//Пример: в Ярмарке образовательных услуг принял участие Юргинский технологический институт
		rule: before_org
		{query: {phrase(5, phrase(0, "В"|"состоится"|пройти, not(области|сфере|центре)), not("состоится"|пройти|[в]),  						/* Поиск ключевых глаголов */ 
						   {phrase(0, repeat(1,5,lemma(noun|adjective|NumeralOrdinal)), 													/* Поиск мероприятия в различных написаниях: */	
									  optional(phrase(0, [по]|[за]|[на]|[над]|[в]|[для],repeat(1,4, lemma(noun|adjective)|"самый"))			// - Конференция по вопросам образования
									  or
									  phrase(0, char(dq|sq),repeat(1,4, lemma(pronoun|noun|adjective|verb|preposition)), char(dq|sq))))		// - Конференция "Вопросы образования"
							  /orn("@", www, неделя, создание, день, исследование, разработка, entity(GeoAdministrative))}:event			/* Исключение некорректных результатов */
						, optional(phrase("на тему", char(dq), repeat(3,12, char(alpha|col)), char(dq)))
						, not("состоится"|пройти|[в])																						/* Исключение некорректных контекстов */
						, "примет участие", $org)																							/* Поиск компании-участника */
						
				//Пример: форум по цифровизации пройдет в МГУ	
				or phrase({orn(фестиваль, курс, "Курс-интенсив", интентсив, конференция, семинар, форум, "цикл лекций", проект, phrase(школа, not(журналистики)))}:event, 		/* Поиск ключевых слов */
																																												/* Поиск мероприятия в различных написаниях: */	
						  {phrase(0, optional([по]|[за]|[на]|[над]|[в]|[для]),repeat(1,4, lemma(noun|adjective)|case(upper, [ПО])))												// - Конференция по вопросам образования
						   or
						   phrase(0, char(dq|sq),repeat(1,4, lemma(pronoun|noun|adjective|verb|preposition)), char(dq|sq))}:event												// - Конференция "Вопросы образования"
						   ,  phrase(orn(пройдет, стартует, состоится),"на базе"|[в], $org)|phrase($org, проводит))																/* Поиск компании-организатора */
				}:m
			
			//Поиск места проведения мероприятия
			//Пример: в Томске в Ярмарке образовательных услуг принял участие Юргинский технологический институт
			rule: geo
			{query: {sentence($m, phrase(7,  not("дистанционные участники из"|департамент),$city|$country|$region))
					 or $m}:m1
					 
				result: Результат = $m1
					attribute: Компания = toentity(Companies|Organizations, $org, field:=Name)
					attribute: Мероприятие = norm($event)
					attribute: Город = toentity(GeoAdministrative, $m1:city, field:=Name)
					attribute: Регион = toentity(GeoAdministrative, $m1:region, field:=Name)
					attribute: Страна = toentity(GeoAdministrative, $m1:country, field:=Name)		
			}
		}
	}
 
	```
	 * выполнить узел

### Структура сущности
* **Результат** - общий вывод найденной сущности
* **Компания** - компания-организатор или компания-участник мероприятия
* **Мероприятие** - название мероприятия
* **Город** - город проведения мероприятия
* **Регион** - регион проведения мероприятия
* **Страна** - страна проведения мероприятия

### Отчет узла
Отчет узла **Извлечение сущностей** выглядит следующим образом:

| Результат| Компания| Мероприятие| Город| Регион| Страна| 
| ------ | ------ |------ |------ |------ |------ |
| Балашовского института приняли участие в международной конференции "Биологическое разнообразие" Керчи | Балашовский институт| международная конференция "Биологическое разнообразие" | Керчь | 

В тексте подсвечиваются следующие фрагменты:
> Преподаватели **Балашовского института приняли участие в международной конференции «Биологическое разнообразие»** в **Керчи**.