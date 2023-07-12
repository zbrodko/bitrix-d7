# Bitrix D7 на примере инфоблоков
 <h3>Подключаем модуль</h3>
<pre>
\Bitrix\Main\Loader::includeModule('iblock');
</pre>
<h3> Делаем запрос в таблицу элементов инфоблока.</h3>
<pre>
$dbItems = \Bitrix\Iblock\ElementTable::getList(array(
	'order' => array('SORT' => 'ASC'), // сортировка
	'select' => array('ID', 'NAME', 'IBLOCK_ID', 'SORT', 'TAGS'), // выбираемые поля, без свойств. Свойства можно получать на старом ядре \CIBlockElement::getProperty
	'filter' => array('IBLOCK_ID' => 4), // фильтр только по полям элемента, свойства (PROPERTY) использовать нельзя
	'group' => array('TAGS'), // группировка по полю, order должен быть пустой
	'limit' => 1000, // целое число, ограничение выбираемого кол-ва
	'offset' => 0, // целое число, указывающее номер первого столбца в результате
	'count_total' => 1, // дает возможность получить кол-во элементов через метод getCount()
	'runtime' => array(), // массив полей сущности, создающихся динамически
	'data_doubling' => false, // разрешает получение нескольких одинаковых записей
	'cache' => array( // Кеш запроса. Сброс можно сделать методом \Bitrix\Iblock\ElementTable::getEntity()->cleanCache();
		'ttl' => 3600, // Время жизни кеша
		'cache_joins' => true // Кешировать ли выборки с JOIN
	),
));
</pre>
<h3>Что можно сделать с $dbItems?</h3>
<pre>
$dbItems->fetch(); // или $dbItems->fetchRaw() получение одной записи, можно перебрать в цикле while ($arItem = $dbItems->fetch())
$dbItems->fetchAll(); // получение всех записей
$dbItems->getCount(); // кол-во найденных записей без учета limit, доступно если при запросе было указано count_total = 1
$dbItems->getSelectedRowsCount(); // кол-во полученных записей с учетом limit
</pre>
<h3>В какие таблицы инфоблоков можно делать запросы</h3>
<pre>
\Bitrix\Iblock\TypeTable::getList(); // типы инфоблоков
\Bitrix\Iblock\IblockTable::getList(); // инфоблоки
\Bitrix\Iblock\PropertyTable::getList(); // свойства инфоблоков
\Bitrix\Iblock\PropertyEnumerationTable::getList(); // значения свойств, например списков
\Bitrix\Iblock\PropertyFeatureTable::getList(); // доп. параметры свойств (например "Показывать на детальной странице элемента")
\Bitrix\Iblock\SectionTable::getList(); // Разделы инфоблоков
\Bitrix\Iblock\ElementTable::getList(); // Элементы инфоблоков 
\Bitrix\Iblock\ElementPropertyTable::getList() // Значения свойств элементов
\Bitrix\Iblock\InheritedPropertyTable::getList(); // Наследуемые свойства (seo шаблоны)
</pre>
<h3>Помимо getList можно использовать другие методы</h3>
<pre>
checkFields(Result $result, $primary, array $data) // метод проверяет поля данных перед записью в БД.
getById($id) // получение элемента по ID
getByPrimary($primary, array $parameters = array()) // метод возвращает выборку по первичному ключу сущности и по опциональным параметрам \Bitrix\Main\Entity\DataManager::getList.
getConnectionName() // метод возвращает имя соединения для сущности. 12.0.9
getCount($filter = array(), array $cache = array()) // метод выполняет COUNT запрос к сущности и возвращает результат. 12.0.10
getEntity() // метод возвращает объект сущности.
getList(array $parameters = array()) // получение элементов, подробнее было выше
getMap() // метод возвращает описание карты сущностей. 12.0.7
getRow(array $parameters) // метод возвращает один столбец (или null) по параметрам для \Bitrix\Main\Entity\DataManager::getList.
getRowById($id) // метод возвращает один столбец (или null) по первичному ключу сущности. 14.0.0
getTableName() // метод возвращает имя таблицы БД для сущности. 12.0.7
query() // метод создаёт и возвращает объект запроса для сущности.
enableCrypto($field, $table = null, $mode = true) // метод устанавливает флаг поддержки шифрования для поля. 17.5.14
cryptoEnabled($field, $table = null) // метод возвращает true если шифрование разрешено для поля. 17.5.14
addMulti($rows, $ignoreEvents = false)
updateMulti($primaries, $data, $ignoreEvents = false)
// Следующий методы заблокированы у инфоблоков
add(array $data) // добавление элемента
delete($primary) // удаление элемента по ID
update($primary, array $data) // обновление элемента по ID
</pre>
<h2>Примеры</h2>
<h3>Инфоблок и его свойства</h3>
<pre>
$arIblock = \Bitrix\Iblock\IblockTable::getList(array(
	'filter' => array('CODE' => 'news'),
))->fetch();

$arProps = \Bitrix\Iblock\PropertyTable::getList(array(
	'select' => array('*'),
	'filter' => array('IBLOCK_ID' => $arIblock['ID'])
))->fetchAll();
</pre>
<h3>Значения определенного свойства типа список</h3>
<pre>
$dbEnums = \Bitrix\Iblock\PropertyEnumerationTable::getList(array(
	'order' => array('SORT' => 'asc'),
	'select' => array('*'),
	'filter' => array('PROPERTY_ID' => $arIblockProp['ID'])
));
while($arEnum = $dbEnums->fetch()) {
	$arIblockProp['ENUM_LIST'][$arEnum['ID']] = $arEnum;
}
</pre>
<h3>Получения пользовательских полей раздела</h3>
<pre>
$entity = \Bitrix\Iblock\Model\Section::compileEntityByIblock($iblockId);
$dbSect = $entity::getList(array(
	"select" => ["UF_POP_BRANDS", 'UF_R_NAME'], // пользовательские поля
	"filter" => ['ID' => $sectionId],
	"cache"  => ['ttl' => 36000],
));
if ($arSect = $dbSect->fetch()) {
	var_dump($arSect["UF_POP_BRANDS"]);
}
</pre>
</h3>
