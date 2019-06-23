###################
查询构造器类
###################

CodeIgniter 提供了查询构造器类，查询构造器允许你使用较少的代码来在数据库中
获取、新增或更新数据。有时只需要一两行代码就能完成数据库操作。CodeIgniter
并不需要为每个数据表提供一个类，而是使用了一种更简单的接口。

除了简单，使用查询构造器的另一个好处是可以让你创建数据库独立的应用程序，
这是因为查询语句是由每个独立的数据库适配器生成的。另外，由于系统会自动对数据
进行转义，所以它还能提供更安全的查询。

.. note:: 如果你想要编写你自己的查询语句，你可以在数据库配置文件中禁用这个类，
	这样数据库核心类库和适配器将使用更少的资源。

.. contents::
    :local:
    :depth: 1

**************
查询
**************

下面的方法用来构建 **SELECT** 语句。

**$this->db->get()**

该方法执行 SELECT 语句并返回查询结果，可以得到一个表的所有数据::

	$query = $this->db->get('mytable');  // Produces: SELECT * FROM mytable

第二和第三个参数用于设置 LIMIT 子句::

	$query = $this->db->get('mytable', 10, 20);

	// Executes: SELECT * FROM mytable LIMIT 20, 10
	// (in MySQL. Other databases have slightly different syntax)

你应该已经注意到了，上面的方法的结果都赋值给了一个 $query 变量，通过这个变量，
我们可以得到查询的结果::

	$query = $this->db->get('mytable');

	foreach ($query->result() as $row)
	{
		echo $row->title;
	}

参考 :doc:`生成查询结果 <results>` 页面获取关于生成结果的更多信息。

**$this->db->get_compiled_select()**

该方法和 **$this->db->get()** 方法一样编译 SELECT 查询并返回查询的 SQL 语句，
但是，该方法并不执行它。

例子::

	$sql = $this->db->get_compiled_select('mytable');
	echo $sql;

	// Prints string: SELECT * FROM mytable

第二个参数用于设置是否重置查询（默认会重置，和使用 `$this->db->get()` 方法时一样）::

	echo $this->db->limit(10,20)->get_compiled_select('mytable', FALSE);

	// Prints string: SELECT * FROM mytable LIMIT 20, 10
	// (in MySQL. Other databases have slightly different syntax)

	echo $this->db->select('title, content, date')->get_compiled_select();

	// Prints string: SELECT title, content, date FROM mytable LIMIT 20, 10

上面的例子中，最值得注意的是，第二个查询并没有用到 **$this->db->from()** 方法，
也没有为查询指定表名参数，但是它生成的 SQL 语句中有 FROM mytable 子句。
这是因为查询并没有被重置（使用 **$this->db->get()** 方法查询会被执行并被重置，
使用 **$this->db->reset_query()** 方法直接重置）。

**$this->db->get_where()**

这个方法基本上和上面的方法一样，但它提供了第二个参数可以让你添加一个 WHERE 子句，
而不是使用 `db->where()` 方法::

	$query = $this->db->get_where('mytable', array('id' => $id), $limit, $offset);

阅读下面的 `db->where()` 方法获取更多信息。

.. note:: get_where() 方法的前身为 getwhere(), 已废除

**$this->db->select()**

该方法用于编写查询语句中的 SELECT 子句::

	$this->db->select('title, content, date');
	$query = $this->db->get('mytable');

	// Executes: SELECT title, content, date FROM mytable

.. note:: 如果你要查询表的所有列，可以不用写这个函数，CodeIgniter 会自动查询所有列（SELECT \*）。

``$this->db->select()`` 方法的第二个参数可选，如果设置为 FALSE，CodeIgniter 将不保护你的
表名和字段名，这在当你编写复合查询语句时很有用，不会破坏你编写的语句。

::

	$this->db->select('(SELECT SUM(payments.amount) FROM payments WHERE payments.invoice_id=4') AS amount_paid', FALSE);
	$query = $this->db->get('mytable');

**$this->db->select_max()**

该方法用于编写查询语句中的 ``SELECT MAX(field)`` 部分，你可以使用第二个参数（可选）重命名结果字段。

::

	$this->db->select_max('age');
	$query = $this->db->get('members');  // Produces: SELECT MAX(age) as age FROM members

	$this->db->select_max('age', 'member_age');
	$query = $this->db->get('members'); // Produces: SELECT MAX(age) as member_age FROM members


**$this->db->select_min()**

该方法用于编写查询语句中的 ``SELECT MIN(field)`` 部分，和 select_max() 方法一样，
你可以使用第二个参数（可选）重命名结果字段。

::

	$this->db->select_min('age');
	$query = $this->db->get('members'); // Produces: SELECT MIN(age) as age FROM members


**$this->db->select_avg()**

该方法用于编写查询语句中的 ``SELECT AVG(field)`` 部分，和 select_max() 方法一样，
你可以使用第二个参数（可选）重命名结果字段。

::

	$this->db->select_avg('age');
	$query = $this->db->get('members'); // Produces: SELECT AVG(age) as age FROM members


**$this->db->select_sum()**

该方法用于编写查询语句中的 ``SELECT SUM(field)`` 部分，和 select_max() 方法一样，
你可以使用第二个参数（可选）重命名结果字段。

::

	$this->db->select_sum('age');
	$query = $this->db->get('members'); // Produces: SELECT SUM(age) as age FROM members

**$this->db->from()**

该方法用于编写查询语句中的 FROM 子句::

	$this->db->select('title, content, date');
	$this->db->from('mytable');
	$query = $this->db->get();  // Produces: SELECT title, content, date FROM mytable

.. note:: 正如前面所说，查询中的 FROM 部分可以在方法 $this->db->get() 中指定，所以，你可以
	 选择任意一种你喜欢的方式。

**$this->db->join()**

该方法用于编写查询语句中的 JOIN 子句::

	$this->db->select('*');
	$this->db->from('blogs');
	$this->db->join('comments', 'comments.id = blogs.id');
	$query = $this->db->get();

	// Produces:
	// SELECT * FROM blogs JOIN comments ON comments.id = blogs.id

如果你的查询中有多个连接，你可以多次调用这个方法。

你可以传入第三个参数指定连接的类型，有这样几种选择：left，right，outer，inner，left
outer 和 right outer 。

::

	$this->db->join('comments', 'comments.id = blogs.id', 'left');
	// Produces: LEFT JOIN comments ON comments.id = blogs.id

*************************
搜索
*************************

**$this->db->where()**

该方法提供了4种方式让你编写查询语句中的 WHERE 子句：

.. note:: 所有的数据将会自动转义，生成安全的查询语句。

#. **简单的 key/value 方式:**

	::

		$this->db->where('name', $name); // Produces: WHERE name = 'Joe'

	注意自动为你加上了等号。

	如果你多次调用该方法，那么多个 WHERE 条件将会使用 AND 连接起来：

	::

		$this->db->where('name', $name);
		$this->db->where('title', $title);
		$this->db->where('status', $status);
		// WHERE name = 'Joe' AND title = 'boss' AND status = 'active'

#. **自定义 key/value 方式:**

	为了控制比较，你可以在第一个参数中包含一个比较运算符：

	::

		$this->db->where('name !=', $name);
		$this->db->where('id <', $id); // Produces: WHERE name != 'Joe' AND id < 45

#. **关联数组方式:**

	::

		$array = array('name' => $name, 'title' => $title, 'status' => $status);
		$this->db->where($array);
		// Produces: WHERE name = 'Joe' AND title = 'boss' AND status = 'active'

	你也可以在这个方法里包含你自己的比较运算符：

	::

		$array = array('name !=' => $name, 'id <' => $id, 'date >' => $date);
		$this->db->where($array);

#. **自定义字符串:**

	你可以完全手工编写 WHERE 子句::

		$where = "name='Joe' AND status='boss' OR status='active'";
		$this->db->where($where);


``$this->db->where()`` 方法有一个可选的第三个参数，如果设置为 FALSE，CodeIgniter
将不保护你的表名和字段名。

::

	$this->db->where('MATCH (field) AGAINST ("value")', NULL, FALSE);

**$this->db->or_where()**

这个方法和上面的方法一样，只是多个 WHERE 条件之间使用 OR 进行连接::

	$this->db->where('name !=', $name);
	$this->db->or_where('id >', $id);  // Produces: WHERE name != 'Joe' OR id > 50

.. note:: or_where() 方法的前身为 orwhere(), 已废除

**$this->db->where_in()**

该方法用于生成 WHERE IN 子句，多个子句之间使用 AND 连接

::

	$names = array('Frank', 'Todd', 'James');
	$this->db->where_in('username', $names);
	// Produces: WHERE username IN ('Frank', 'Todd', 'James')


**$this->db->or_where_in()**

该方法用于生成 WHERE IN 子句，多个子句之间使用 OR 连接

::

	$names = array('Frank', 'Todd', 'James');
	$this->db->or_where_in('username', $names);
	// Produces: OR username IN ('Frank', 'Todd', 'James')

**$this->db->where_not_in()**

该方法用于生成 WHERE NOT IN 子句，多个子句之间使用 AND 连接

::

	$names = array('Frank', 'Todd', 'James');
	$this->db->where_not_in('username', $names);
	// Produces: WHERE username NOT IN ('Frank', 'Todd', 'James')


**$this->db->or_where_not_in()**

该方法用于生成 WHERE NOT IN 子句，多个子句之间使用 OR 连接

::

	$names = array('Frank', 'Todd', 'James');
	$this->db->or_where_not_in('username', $names);
	// Produces: OR username NOT IN ('Frank', 'Todd', 'James')

************************
模糊搜索
************************

**$this->db->like()**

该方法用于生成 LIKE 子句，在进行搜索时非常有用。

.. note:: 所有数据将会自动被转义。

#. **简单 key/value 方式:**

	::

		$this->db->like('title', 'match');
		// Produces: WHERE `title` LIKE '%match%' ESCAPE '!'

	如果你多次调用该方法，那么多个 WHERE 条件将会使用 AND 连接起来::

		$this->db->like('title', 'match');
		$this->db->like('body', 'match');
		// WHERE `title` LIKE '%match%' ESCAPE '!' AND  `body` LIKE '%match% ESCAPE '!'

	可以传入第三个可选的参数来控制 LIKE 通配符（%）的位置，可用选项有：'before'，'after' 和
	'both' (默认为 'both')。

	::

		$this->db->like('title', 'match', 'before');	// Produces: WHERE `title` LIKE '%match' ESCAPE '!'
		$this->db->like('title', 'match', 'after');	// Produces: WHERE `title` LIKE 'match%' ESCAPE '!'
		$this->db->like('title', 'match', 'both');	// Produces: WHERE `title` LIKE '%match%' ESCAPE '!'

#. **关联数组方式:**

	::

		$array = array('title' => $match, 'page1' => $match, 'page2' => $match);
		$this->db->like($array);
		// WHERE `title` LIKE '%match%' ESCAPE '!' AND  `page1` LIKE '%match%' ESCAPE '!' AND  `page2` LIKE '%match%' ESCAPE '!'


**$this->db->or_like()**

这个方法和上面的方法一样，只是多个 WHERE 条件之间使用 OR 进行连接::

	$this->db->like('title', 'match'); $this->db->or_like('body', $match);
	// WHERE `title` LIKE '%match%' ESCAPE '!' OR  `body` LIKE '%match%' ESCAPE '!'

.. note:: ``or_like()`` 方法的前身为 ``orlike()``, 已废除

**$this->db->not_like()**

这个方法和 ``like()`` 方法一样，只是生成 NOT LIKE 子句::

	$this->db->not_like('title', 'match');	// WHERE `title` NOT LIKE '%match% ESCAPE '!'

**$this->db->or_not_like()**

这个方法和 ``not_like()`` 方法一样，只是多个 WHERE 条件之间使用 OR 进行连接::

	$this->db->like('title', 'match');
	$this->db->or_not_like('body', 'match');
	// WHERE `title` LIKE '%match% OR  `body` NOT LIKE '%match%' ESCAPE '!'

**$this->db->group_by()**

该方法用于生成 GROUP BY 子句::

	$this->db->group_by("title"); // Produces: GROUP BY title

你也可以通过一个数组传入多个值::

	$this->db->group_by(array("title", "date"));  // Produces: GROUP BY title, date

.. note:: group_by() 方法前身为 groupby(), 已废除

**$this->db->distinct()**

该方法用于向查询中添加 DISTINCT 关键字：

::

	$this->db->distinct();
	$this->db->get('table'); // Produces: SELECT DISTINCT * FROM table

**$this->db->having()**

该方法用于生成 HAVING 子句，有下面两种不同的语法::

	$this->db->having('user_id = 45');  // Produces: HAVING user_id = 45
	$this->db->having('user_id',  45);  // Produces: HAVING user_id = 45

你也可以通过一个数组传入多个值::

	$this->db->having(array('title =' => 'My Title', 'id <' => $id));
	// Produces: HAVING title = 'My Title', id < 45

如果 CodeIgniter 自动转义你的查询，为了避免转义，你可以将第三个参数设置为 FALSE 。

::

	$this->db->having('user_id',  45);  // Produces: HAVING `user_id` = 45 in some databases such as MySQL
	$this->db->having('user_id',  45, FALSE);  // Produces: HAVING user_id = 45


**$this->db->or_having()**

该方法和 ``having()`` 方法一样，只是多个条件之间使用 OR 进行连接。

****************
排序
****************

**$this->db->order_by()**

该方法用于生成 ORDER BY 子句。

第一个参数为你想要排序的字段名，第二个参数用于设置排序的方向，
可选项有： ASC（升序），DESC（降序）和 RANDOM （随机）。

::

	$this->db->order_by('title', 'DESC');
	// Produces: ORDER BY `title` DESC

第一个参数也可以是你自己的排序字符串::

	$this->db->order_by('title DESC, name ASC');
	// Produces: ORDER BY `title` DESC, `name` ASC

如果需要根据多个字段进行排序，可以多次调用该方法。

::

	$this->db->order_by('title', 'DESC');
	$this->db->order_by('name', 'ASC');
	// Produces: ORDER BY `title` DESC, `name` ASC

如果你选择了 **RANDOM** （随机排序），第一个参数会被忽略，但是你可以传入一个
数字值，作为随机数的 seed。

::

	$this->db->order_by('title', 'RANDOM');
	// Produces: ORDER BY RAND()

	$this->db->order_by(42, 'RANDOM');
	// Produces: ORDER BY RAND(42)

.. note:: order_by() 方法的前身为 orderby(), 已废除

.. note:: Oracle 暂时还不支持随机排序，会默认使用升序

****************************
分页与计数
****************************

**$this->db->limit()**

该方法用于限制你的查询返回结果的数量::

	$this->db->limit(10);  // Produces: LIMIT 10

第二个参数可以用来设置偏移。

::

	// Produces: LIMIT 20, 10 (in MySQL.  Other databases have slightly different syntax)
	$this->db->limit(10, 20);

**$this->db->count_all_results()**

该方法用于获取特定查询返回结果的数量，也可以使用查询构造器的这些方法：
``where()``，``or_where()``，``like()``，``or_like()`` 等等。举例::

	echo $this->db->count_all_results('my_table');  // Produces an integer, like 25
	$this->db->like('title', 'match');
	$this->db->from('my_table');
	echo $this->db->count_all_results(); // Produces an integer, like 17

但是，这个方法会重置你在 ``select()`` 方法里设置的所有值，如果你希望保留它们，可以将
第二个参数设置为 FALSE ::

	echo $this->db->count_all_results('my_table', FALSE);

**$this->db->count_all()**

该方法用于获取某个表的总行数，第一个参数为表名::

	echo $this->db->count_all('my_table');  // Produces an integer, like 25

**************
查询条件组
**************

查询条件组可以让你生成用括号括起来的一组 WHERE 条件，这能创造出非常复杂的 WHERE 子句，
支持嵌套的条件组。例如::

	$this->db->select('*')->from('my_table')
		->group_start()
			->where('a', 'a')
			->or_group_start()
				->where('b', 'b')
				->where('c', 'c')
			->group_end()
		->group_end()
		->where('d', 'd')
	->get();

	// Generates:
	// SELECT * FROM (`my_table`) WHERE ( `a` = 'a' OR ( `b` = 'b' AND `c` = 'c' ) ) AND `d` = 'd'

.. note:: 条件组必须要配对，确保每个 group_start() 方法都有一个 group_end() 方法与之配对。

**$this->db->group_start()**

开始一个新的条件组，为查询中的 WHERE 条件添加一个左括号。

**$this->db->or_group_start()**

开始一个新的条件组，为查询中的 WHERE 条件添加一个左括号，并在前面加上 OR 。

**$this->db->not_group_start()**

开始一个新的条件组，为查询中的 WHERE 条件添加一个左括号，并在前面加上 NOT 。

**$this->db->or_not_group_start()**

开始一个新的条件组，为查询中的 WHERE 条件添加一个左括号，并在前面加上 OR NOT 。

**$this->db->group_end()**

结束当前的条件组，为查询中的 WHERE 条件添加一个右括号。

**************
插入数据
**************

**$this->db->insert()**

该方法根据你提供的数据生成一条 INSERT 语句并执行，它的参数是一个**数组**
或一个**对象**，下面是使用数组的例子::

	$data = array(
		'title' => 'My title',
		'name' => 'My Name',
		'date' => 'My date'
	);

	$this->db->insert('mytable', $data);
	// Produces: INSERT INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')

第一个参数为要插入的表名，第二个参数为要插入的数据，是个关联数组。

下面是使用对象的例子::

	/*
	class Myclass {
		public $title = 'My Title';
		public $content = 'My Content';
		public $date = 'My Date';
	}
	*/

	$object = new Myclass;
	$this->db->insert('mytable', $object);
	// Produces: INSERT INTO mytable (title, content, date) VALUES ('My Title', 'My Content', 'My Date')

第一个参数为要插入的表名，第二个参数为要插入的数据，是个对象。

.. note:: 所有数据会被自动转义，生成安全的查询语句。

**$this->db->get_compiled_insert()**

该方法和 $this->db->insert() 方法一样根据你提供的数据生成一条 INSERT 语句，但是并不执行。

例如::

	$data = array(
		'title' => 'My title',
		'name'  => 'My Name',
		'date'  => 'My date'
	);

	$sql = $this->db->set($data)->get_compiled_insert('mytable');
	echo $sql;

	// Produces string: INSERT INTO mytable (`title`, `name`, `date`) VALUES ('My title', 'My name', 'My date')

第二个参数用于设置是否重置查询（默认情况下会重置，正如 $this->db->insert() 方法一样）::

	echo $this->db->set('title', 'My Title')->get_compiled_insert('mytable', FALSE);

	// Produces string: INSERT INTO mytable (`title`) VALUES ('My Title')

	echo $this->db->set('content', 'My Content')->get_compiled_insert();

	// Produces string: INSERT INTO mytable (`title`, `content`) VALUES ('My Title', 'My Content')

上面的例子中，最值得注意的是，第二个查询并没有用到 **$this->db->from()** 方法，
也没有为查询指定表名参数，但是它生成的 SQL 语句中有 INTO mytable 子句。
这是因为查询并没有被重置（使用 **$this->db->insert()** 方法会被执行并被重置，
使用 **$this->db->reset_query()** 方法直接重置）。

.. note:: 这个方法不支持批量插入。

**$this->db->insert_batch()**

该方法根据你提供的数据生成一条 INSERT 语句并执行，它的参数是一个**数组**
或一个**对象**，下面是使用数组的例子::

	$data = array(
		array(
			'title' => 'My title',
			'name' => 'My Name',
			'date' => 'My date'
		),
		array(
			'title' => 'Another title',
			'name' => 'Another Name',
			'date' => 'Another date'
		)
	);

	$this->db->insert_batch('mytable', $data);
	// Produces: INSERT INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date'),  ('Another title', 'Another name', 'Another date')

第一个参数为要插入的表名，第二个参数为要插入的数据，是个二维数组。

.. note:: 所有数据会被自动转义，生成安全的查询语句。

*************
更新数据
*************

**$this->db->replace()**

该方法用于执行一条 REPLACE 语句，REPLACE 语句根据表的**主键**和**唯一索引**
来执行，类似于标准的 DELETE + INSERT 。
使用这个方法，你不用再手工去实现 ``select()``，``update()``，``delete()``
以及 ``insert()`` 这些方法的不同组合，为你节约大量时间。

例如::

	$data = array(
		'title' => 'My title',
		'name'  => 'My Name',
		'date'  => 'My date'
	);

	$this->db->replace('table', $data);

	// Executes: REPLACE INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')

上面的例子中，我们假设 *title* 字段是我们的主键，那么如果我们数据库里有一行
的 *title* 列的值为 'My title'，这一行将会被删除并被我们的新数据所取代。

也可以使用 ``set()`` 方法，而且所有字段都被自动转义，正如 ``insert()`` 方法一样。

**$this->db->set()**

该方法用于设置新增或更新的数据。

**该方法可以取代直接传递数据数组到 insert 或 update 方法：**

::

	$this->db->set('name', $name);
	$this->db->insert('mytable');  // Produces: INSERT INTO mytable (`name`) VALUES ('{$name}')

如果你多次调用该方法，它会正确组装出 INSERT 或 UPDATE 语句来::

	$this->db->set('name', $name);
	$this->db->set('title', $title);
	$this->db->set('status', $status);
	$this->db->insert('mytable');

**set()** 方法也接受可选的第三个参数（``$escape``），如果设置为 FALSE，数据将不会自动转义。为了说明两者之间的区别，这里有一个带转义的 ``set()`` 方法和不带转义的例子。

::

	$this->db->set('field', 'field+1', FALSE);
	$this->db->where('id', 2);
	$this->db->update('mytable'); // gives UPDATE mytable SET field = field+1 WHERE id = 2

	$this->db->set('field', 'field+1');
	$this->db->where('id', 2);
	$this->db->update('mytable'); // gives UPDATE `mytable` SET `field` = 'field+1' WHERE `id` = 2

你也可以传一个关联数组作为参数::

	$array = array(
		'name' => $name,
		'title' => $title,
		'status' => $status
	);

	$this->db->set($array);
	$this->db->insert('mytable');

或者一个对象::

	/*
	class Myclass {
		public $title = 'My Title';
		public $content = 'My Content';
		public $date = 'My Date';
	}
	*/

	$object = new Myclass;
	$this->db->set($object);
	$this->db->insert('mytable');

**$this->db->update()**

该方法根据你提供的数据生成一条 UPDATE 语句并执行，它的参数是一个 **数组** 或一个 **对象** ，下面是使用数组的例子::

	$data = array(
		'title' => $title,
		'name' => $name,
		'date' => $date
	);

	$this->db->where('id', $id);
	$this->db->update('mytable', $data);
	// Produces:
	//
	//	UPDATE mytable
	//	SET title = '{$title}', name = '{$name}', date = '{$date}'
	//	WHERE id = $id

或者你可以使用一个对象::

	/*
	class Myclass {
		public $title = 'My Title';
		public $content = 'My Content';
		public $date = 'My Date';
	}
	*/

	$object = new Myclass;
	$this->db->where('id', $id);
	$this->db->update('mytable', $object);
	// Produces:
	//
	// UPDATE `mytable`
	// SET `title` = '{$title}', `name` = '{$name}', `date` = '{$date}'
	// WHERE id = `$id`

.. note:: 所有数据会被自动转义，生成安全的查询语句。

你应该注意到 $this->db->where() 方法的使用，它可以为你设置 WHERE 子句。
你也可以直接使用字符串形式设置 WHERE 子句::

	$this->db->update('mytable', $data, "id = 4");

或者使用一个数组::

	$this->db->update('mytable', $data, array('id' => $id));

当执行 UPDATE 操作时，你还可以使用上面介绍的 $this->db->set() 方法。

**$this->db->update_batch()**

该方法根据你提供的数据生成一条 UPDATE 语句并执行，它的参数是一个**数组**
或一个**对象**，下面是使用数组的例子::

	$data = array(
	   array(
	      'title' => 'My title' ,
	      'name' => 'My Name 2' ,
	      'date' => 'My date 2'
	   ),
	   array(
	      'title' => 'Another title' ,
	      'name' => 'Another Name 2' ,
	      'date' => 'Another date 2'
	   )
	);

	$this->db->update_batch('mytable', $data, 'title');

	// Produces:
	// UPDATE `mytable` SET `name` = CASE
	// WHEN `title` = 'My title' THEN 'My Name 2'
	// WHEN `title` = 'Another title' THEN 'Another Name 2'
	// ELSE `name` END,
	// `date` = CASE
	// WHEN `title` = 'My title' THEN 'My date 2'
	// WHEN `title` = 'Another title' THEN 'Another date 2'
	// ELSE `date` END
	// WHERE `title` IN ('My title','Another title')

第一个参数为要更新的表名，第二个参数为要更新的数据，是个二维数组，第三个
参数是 WHERE 语句的键。

.. note:: 所有数据会被自动转义，生成安全的查询语句。

.. note:: 取决于该方法的内部实现，在这个方法之后调用 ``affected_rows()`` 方法
	返回的结果可能会不正确。但是你可以直接使用该方法的返回值，代表了受影响的行数。

**$this->db->get_compiled_update()**

该方法和 ``$this->db->get_compiled_insert()`` 方法完全一样，除了生成的 SQL 语句是
UPDATE 而不是 INSERT。

查看 `$this->db->get_compiled_insert()` 方法的文档获取更多信息。

.. note:: 该方法不支持批量更新。

*************
删除数据
*************

**$this->db->delete()**

该方法生成 DELETE 语句并执行。

::

	$this->db->delete('mytable', array('id' => $id));  // Produces: // DELETE FROM mytable  // WHERE id = $id

第一个参数为表名，第二个参数为 WHERE 条件。你也可以不用第二个参数，
使用 where() 或者 or_where() 函数来替代它::

	$this->db->where('id', $id);
	$this->db->delete('mytable');

	// Produces:
	// DELETE FROM mytable
	// WHERE id = $id

如果你想要从多个表中删除数据，你也可以将由多个表名构成的数组传给 delete() 方法。

::

	$tables = array('table1', 'table2', 'table3');
	$this->db->where('id', '5');
	$this->db->delete($tables);

如果你想要删除一个表中的所有数据，可以使用 truncate() 或 empty_table() 方法。

**$this->db->empty_table()**

该方法生成 DELETE 语句并执行::

	  $this->db->empty_table('mytable'); // Produces: DELETE FROM mytable

**$this->db->truncate()**

该方法生成 TRUNCATE 语句并执行。

::

	$this->db->from('mytable');
	$this->db->truncate();

	// or

	$this->db->truncate('mytable');

	// Produce:
	// TRUNCATE mytable

.. note:: 如果 TRUNCATE 语句不可用，truncate() 方法将执行 "DELETE FROM table"。

**$this->db->get_compiled_delete()**

该方法和 ``$this->db->get_compiled_insert()`` 方法完全一样，除了生成的 SQL 语句是
DELETE 而不是 INSERT。

查看 `$this->db->get_compiled_insert()` 方法的文档获取更多信息。

***************
链式方法
***************

通过将多个方法连接在一起，链式方法可以大大的简化你的语法。感受一下这个例子::

	$query = $this->db->select('title')
			->where('id', $id)
			->limit(10, 20)
			->get('mytable');

.. _ar-caching:

*********************
查询构造器缓存
*********************

尽管不是 "真正的" 缓存，查询构造器允许你将查询的某个特定部分保存（或 "缓存"）起来，
以便在你的脚本执行之后重用。一般情况下，当查询构造器的一次调用结束后，所有已存储的信息
都会被重置，以便下一次调用。如果开启缓存，你就可以使信息避免被重置，方便你进行重用。

缓存调用是累加的。如果你调用了两次有缓存的 select()，然后再调用两次没有缓存的 select()，
这会导致 select() 被调用4次。

有三个可用的缓存方法方法:

**$this->db->start_cache()**

如需开启缓存必须先调用此方法，所有支持的查询类型（见下文）都会被存储起来供以后使用。

**$this->db->stop_cache()**

此方法用于停止缓存。

**$this->db->flush_cache()**

此方法用于清空缓存。

这里是一个使用缓存的例子::

	$this->db->start_cache();
	$this->db->select('field1');
	$this->db->stop_cache();
	$this->db->get('tablename');
	//Generates: SELECT `field1` FROM (`tablename`)

	$this->db->select('field2');
	$this->db->get('tablename');
	//Generates:  SELECT `field1`, `field2` FROM (`tablename`)

	$this->db->flush_cache();
	$this->db->select('field2');
	$this->db->get('tablename');
	//Generates:  SELECT `field2` FROM (`tablename`)


.. note:: 支持缓存的语句有: select, from, join, where, like, group_by, having, order_by


***********************
重置查询构造器
***********************

**$this->db->reset_query()**

该方法无需执行就能重置查询构造器中的查询，$this->db->get() 和 $this->db->insert()
方法也可以用于重置查询，但是必须要先执行它。和这两个方法一样，使用`查询构造器缓存`_
缓存下来的查询不会被重置。

当你在使用查询构造器生成 SQL 语句（如：``$this->db->get_compiled_select()``），
之后再执行它。这种情况下，不重置查询缓存将非常有用::

	// Note that the second parameter of the get_compiled_select method is FALSE
	$sql = $this->db->select(array('field1','field2'))
					->where('field3',5)
					->get_compiled_select('mytable', FALSE);

	// ...
	// Do something crazy with the SQL code... like add it to a cron script for
	// later execution or something...
	// ...

	$data = $this->db->get()->result_array();

	// Would execute and return an array of results of the following query:
	// SELECT field1, field1 from mytable where field3 = 5;

.. note:: 如果你正在使用查询构造器缓存功能，连续两次调用 ``get_compiled_select()`` 方法
	并且不重置你的查询，这将会导致缓存被合并两次。举例来说，例如你正在缓存 ``select()``
	方法，那么会查询两个相同的字段。

***************
Class Reference
***************

.. php:class:: CI_DB_query_builder

	.. php:method:: reset_query()

		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Resets the current Query Builder state.  Useful when you want
		to build a query that can be cancelled under certain conditions.

	.. php:method:: start_cache()

		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Starts the Query Builder cache.

	.. php:method:: stop_cache()

		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Stops the Query Builder cache.

	.. php:method:: flush_cache()

		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Empties the Query Builder cache.

	.. php:method:: set_dbprefix([$prefix = ''])

		:param	string	$prefix: The new prefix to use
		:returns:	The DB prefix in use
		:rtype:	string

		Sets the database prefix, without having to reconnect.

	.. php:method:: dbprefix([$table = ''])

		:param	string	$table: The table name to prefix
		:returns:	The prefixed table name
		:rtype:	string

		Prepends a database prefix, if one exists in configuration.

	.. php:method:: count_all_results([$table = '', [$reset = TRUE]])

		:param	string	$table: Table name
		:param	bool	$reset: Whether to reset values for SELECTs
		:returns:	Number of rows in the query result
		:rtype:	int

		Generates a platform-specific query string that counts
		all records returned by an Query Builder query.

	.. php:method:: get([$table = ''[, $limit = NULL[, $offset = NULL]]])

		:param	string	$table: The table to query
		:param	int	$limit: The LIMIT clause
		:param	int	$offset: The OFFSET clause
		:returns:	CI_DB_result instance (method chaining)
		:rtype:	CI_DB_result

		Compiles and runs SELECT statement based on the already
		called Query Builder methods.

	.. php:method:: get_where([$table = ''[, $where = NULL[, $limit = NULL[, $offset = NULL]]]])

		:param	mixed	$table: The table(s) to fetch data from; string or array
		:param	string	$where: The WHERE clause
		:param	int	$limit: The LIMIT clause
		:param	int	$offset: The OFFSET clause
		:returns:	CI_DB_result instance (method chaining)
		:rtype:	CI_DB_result

		Same as ``get()``, but also allows the WHERE to be added directly.

	.. php:method:: select([$select = '*'[, $escape = NULL]])

		:param	string	$select: The SELECT portion of a query
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a SELECT clause to a query.

	.. php:method:: select_avg([$select = ''[, $alias = '']])

		:param	string	$select: Field to compute the average of
		:param	string	$alias: Alias for the resulting value name
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a SELECT AVG(field) clause to a query.

	.. php:method:: select_max([$select = ''[, $alias = '']])

		:param	string	$select: Field to compute the maximum of
		:param	string	$alias: Alias for the resulting value name
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a SELECT MAX(field) clause to a query.

	.. php:method:: select_min([$select = ''[, $alias = '']])

		:param	string	$select: Field to compute the minimum of
		:param	string	$alias: Alias for the resulting value name
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a SELECT MIN(field) clause to a query.

	.. php:method:: select_sum([$select = ''[, $alias = '']])

		:param	string	$select: Field to compute the sum of
		:param	string	$alias: Alias for the resulting value name
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a SELECT SUM(field) clause to a query.

	.. php:method:: distinct([$val = TRUE])

		:param	bool	$val: Desired value of the "distinct" flag
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Sets a flag which tells the query builder to add
		a DISTINCT clause to the SELECT portion of the query.

	.. php:method:: from($from)

		:param	mixed	$from: Table name(s); string or array
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Specifies the FROM clause of a query.

	.. php:method:: join($table, $cond[, $type = ''[, $escape = NULL]])

		:param	string	$table: Table name to join
		:param	string	$cond: The JOIN ON condition
		:param	string	$type: The JOIN type
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a JOIN clause to a query.

	.. php:method:: where($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: Name of field to compare, or associative array
		:param	mixed	$value: If a single key, compared to this value
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	DB_query_builder instance
		:rtype:	object

		Generates the WHERE portion of the query.
                Separates multiple calls with 'AND'.

	.. php:method:: or_where($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: Name of field to compare, or associative array
		:param	mixed	$value: If a single key, compared to this value
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	DB_query_builder instance
		:rtype:	object

		Generates the WHERE portion of the query.
                Separates multiple calls with 'OR'.

	.. php:method:: or_where_in([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	$key: The field to search
		:param	array	$values: The values searched on
		:param	bool	$escape: Whether to escape identifiers
		:returns:	DB_query_builder instance
		:rtype:	object

		Generates a WHERE field IN('item', 'item') SQL query,
                joined with 'OR' if appropriate.

	.. php:method:: or_where_not_in([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	$key: The field to search
		:param	array	$values: The values searched on
		:param	bool	$escape: Whether to escape identifiers
		:returns:	DB_query_builder instance
		:rtype:	object

		Generates a WHERE field NOT IN('item', 'item') SQL query,
                joined with 'OR' if appropriate.

	.. php:method:: where_in([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	$key: Name of field to examine
		:param	array	$values: Array of target values
		:param	bool	$escape: Whether to escape identifiers
		:returns:	DB_query_builder instance
		:rtype:	object

		Generates a WHERE field IN('item', 'item') SQL query,
                joined with 'AND' if appropriate.

	.. php:method:: where_not_in([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	$key: Name of field to examine
		:param	array	$values: Array of target values
		:param	bool	$escape: Whether to escape identifiers
		:returns:	DB_query_builder instance
		:rtype:	object

		Generates a WHERE field NOT IN('item', 'item') SQL query,
                joined with 'AND' if appropriate.

	.. php:method:: group_start()

		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Starts a group expression, using ANDs for the conditions inside it.

	.. php:method:: or_group_start()

		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Starts a group expression, using ORs for the conditions inside it.

	.. php:method:: not_group_start()

		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Starts a group expression, using AND NOTs for the conditions inside it.

	.. php:method:: or_not_group_start()

		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Starts a group expression, using OR NOTs for the conditions inside it.

	.. php:method:: group_end()

		:returns:	DB_query_builder instance
		:rtype:	object

		Ends a group expression.

	.. php:method:: like($field[, $match = ''[, $side = 'both'[, $escape = NULL]]])

		:param	string	$field: Field name
		:param	string	$match: Text portion to match
		:param	string	$side: Which side of the expression to put the '%' wildcard on
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a LIKE clause to a query, separating multiple calls with AND.

	.. php:method:: or_like($field[, $match = ''[, $side = 'both'[, $escape = NULL]]])

		:param	string	$field: Field name
		:param	string	$match: Text portion to match
		:param	string	$side: Which side of the expression to put the '%' wildcard on
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a LIKE clause to a query, separating multiple class with OR.

	.. php:method:: not_like($field[, $match = ''[, $side = 'both'[, $escape = NULL]]])

		:param	string	$field: Field name
		:param	string	$match: Text portion to match
		:param	string	$side: Which side of the expression to put the '%' wildcard on
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a NOT LIKE clause to a query, separating multiple calls with AND.

	.. php:method:: or_not_like($field[, $match = ''[, $side = 'both'[, $escape = NULL]]])

		:param	string	$field: Field name
		:param	string	$match: Text portion to match
		:param	string	$side: Which side of the expression to put the '%' wildcard on
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a NOT LIKE clause to a query, separating multiple calls with OR.

	.. php:method:: having($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: Identifier (string) or associative array of field/value pairs
		:param	string	$value: Value sought if $key is an identifier
		:param	string	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a HAVING clause to a query, separating multiple calls with AND.

	.. php:method:: or_having($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: Identifier (string) or associative array of field/value pairs
		:param	string	$value: Value sought if $key is an identifier
		:param	string	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a HAVING clause to a query, separating multiple calls with OR.

	.. php:method:: group_by($by[, $escape = NULL])

		:param	mixed	$by: Field(s) to group by; string or array
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds a GROUP BY clause to a query.

	.. php:method:: order_by($orderby[, $direction = ''[, $escape = NULL]])

		:param	string	$orderby: Field to order by
		:param	string	$direction: The order requested - ASC, DESC or random
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds an ORDER BY clause to a query.

	.. php:method:: limit($value[, $offset = 0])

		:param	int	$value: Number of rows to limit the results to
		:param	int	$offset: Number of rows to skip
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds LIMIT and OFFSET clauses to a query.

	.. php:method:: offset($offset)

		:param	int	$offset: Number of rows to skip
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds an OFFSET clause to a query.

	.. php:method:: set($key[, $value = ''[, $escape = NULL]])

		:param	mixed	$key: Field name, or an array of field/value pairs
		:param	string	$value: Field value, if $key is a single field
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds field/value pairs to be passed later to ``insert()``,
		``update()`` or ``replace()``.

	.. php:method:: insert([$table = ''[, $set = NULL[, $escape = NULL]]])

		:param	string	$table: Table name
		:param	array	$set: An associative array of field/value pairs
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	TRUE on success, FALSE on failure
		:rtype:	bool

		Compiles and executes an INSERT statement.

	.. php:method:: insert_batch($table[, $set = NULL[, $escape = NULL[, $batch_size = 100]]])

		:param	string	$table: Table name
		:param	array	$set: Data to insert
		:param	bool	$escape: Whether to escape values and identifiers
		:param	int	$batch_size: Count of rows to insert at once
		:returns:	Number of rows inserted or FALSE on failure
		:rtype:	mixed

		Compiles and executes batch ``INSERT`` statements.

		.. note:: When more than ``$batch_size`` rows are provided, multiple
			``INSERT`` queries will be executed, each trying to insert
			up to ``$batch_size`` rows.

	.. php:method:: set_insert_batch($key[, $value = ''[, $escape = NULL]])

		:param	mixed	$key: Field name or an array of field/value pairs
		:param	string	$value: Field value, if $key is a single field
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds field/value pairs to be inserted in a table later via ``insert_batch()``.

	.. php:method:: update([$table = ''[, $set = NULL[, $where = NULL[, $limit = NULL]]]])

		:param	string	$table: Table name
		:param	array	$set: An associative array of field/value pairs
		:param	string	$where: The WHERE clause
		:param	int	$limit: The LIMIT clause
		:returns:	TRUE on success, FALSE on failure
		:rtype:	bool

		Compiles and executes an UPDATE statement.

	.. php:method:: update_batch($table[, $set = NULL[, $value = NULL[, $batch_size = 100]]])

		:param	string	$table: Table name
		:param	array	$set: Field name, or an associative array of field/value pairs
		:param	string	$value: Field value, if $set is a single field
		:param	int	$batch_size: Count of conditions to group in a single query
		:returns:	Number of rows updated or FALSE on failure
		:rtype:	mixed

		Compiles and executes batch ``UPDATE`` statements.

		.. note:: When more than ``$batch_size`` field/value pairs are provided,
			multiple queries will be executed, each handling up to
			``$batch_size`` field/value pairs.

	.. php:method:: set_update_batch($key[, $value = ''[, $escape = NULL]])

		:param	mixed	$key: Field name or an array of field/value pairs
		:param	string	$value: Field value, if $key is a single field
		:param	bool	$escape: Whether to escape values and identifiers
		:returns:	CI_DB_query_builder instance (method chaining)
		:rtype:	CI_DB_query_builder

		Adds field/value pairs to be updated in a table later via ``update_batch()``.

	.. php:method:: replace([$table = ''[, $set = NULL]])

		:param	string	$table: Table name
		:param	array	$set: An associative array of field/value pairs
		:returns:	TRUE on success, FALSE on failure
		:rtype:	bool

		Compiles and executes a REPLACE statement.

	.. php:method:: delete([$table = ''[, $where = ''[, $limit = NULL[, $reset_data = TRUE]]]])

		:param	mixed	$table: The table(s) to delete from; string or array
		:param	string	$where: The WHERE clause
		:param	int	$limit: The LIMIT clause
		:param	bool	$reset_data: TRUE to reset the query "write" clause
		:returns:	CI_DB_query_builder instance (method chaining) or FALSE on failure
		:rtype:	mixed

		Compiles and executes a DELETE query.

	.. php:method:: truncate([$table = ''])

		:param	string	$table: Table name
		:returns:	TRUE on success, FALSE on failure
		:rtype:	bool

		Executes a TRUNCATE statement on a table.

		.. note:: If the database platform in use doesn't support TRUNCATE,
			a DELETE statement will be used instead.

	.. php:method:: empty_table([$table = ''])

		:param	string	$table: Table name
		:returns:	TRUE on success, FALSE on failure
		:rtype:	bool

		Deletes all records from a table via a DELETE statement.

	.. php:method:: get_compiled_select([$table = ''[, $reset = TRUE]])

		:param	string	$table: Table name
		:param	bool	$reset: Whether to reset the current QB values or not
		:returns:	The compiled SQL statement as a string
		:rtype:	string

		Compiles a SELECT statement and returns it as a string.

	.. php:method:: get_compiled_insert([$table = ''[, $reset = TRUE]])

		:param	string	$table: Table name
		:param	bool	$reset: Whether to reset the current QB values or not
		:returns:	The compiled SQL statement as a string
		:rtype:	string

		Compiles an INSERT statement and returns it as a string.

	.. php:method:: get_compiled_update([$table = ''[, $reset = TRUE]])

		:param	string	$table: Table name
		:param	bool	$reset: Whether to reset the current QB values or not
		:returns:	The compiled SQL statement as a string
		:rtype:	string

		Compiles an UPDATE statement and returns it as a string.

	.. php:method:: get_compiled_delete([$table = ''[, $reset = TRUE]])

		:param	string	$table: Table name
		:param	bool	$reset: Whether to reset the current QB values or not
		:returns:	The compiled SQL statement as a string
		:rtype:	string

		Compiles a DELETE statement and returns it as a string.
