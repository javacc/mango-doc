增删改查NoSQL
=============

为提高开发效率，mango框架提供了一整套NoSQL的方案，开发人员无需书写SQL即可完成绝大多数的数据库增删改查操作。

CrudDao接口
___________

为实现增删改查NoSQL，mango框架对开发人员提供了 `CrudDao <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/crud/CrudDao.java>`_ 接口，我们只需简单继承CrudDao接口，不需要书写任何SQL，即可获得常用的增删改查方法。

CrudDao定义如下：

.. code-block:: java

	public interface CrudDao<T, ID> extends NamedCrudDao<T, ID> {

	  void add(T entity);

	  long addAndReturnGeneratedId(T entity);

	  void add(Iterable<T> entities);

	  T getOne(ID primaryKey);

	  Optional<T> findOne(ID primaryKey);

	  List<T> findMany(Iterable<ID> primaryKeys);

	  long count();

	  int update(T entity);

	  int[] update(Iterable<T> entities);

	  int delete(ID primaryKey);

	  List<T> findAll();

	  PageResult<T> findAll(Page page);

	  List<T> findAll(Sort sort);

	}


订单表增删改查实例
__________________

t_order表

.. code-block:: sql

	CREATE TABLE `t_order` (
	  `id` int(11) NOT NULL AUTO_INCREMENT, 
	  `uid` int(11) NOT NULL,
	  `status` int(11) NOT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Order对象

.. code-block:: java

	public class Order {

	  @ID
	  @AutoGenerated
	  private int id;

	  private int uid;

	  private int status;

	  // 省略get与set方法

	}

`@ID <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/ID.java>`_ 注解需放在业务主键所对应的属性上。

`@AutoGenerated <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/AutoGenerated.java>`_ 注解需放在自增主键所对应的属性上。

上面的实例中，业务主键与自增主键均为id，所以将@ID注解与@AutoGenerated放在了Order类的id属性上。

测试代码：

.. code-block:: java

	public class OrderDaoMain {

	  public static void main(String[] args) {
	    String driverClassName = "com.mysql.jdbc.Driver";
	    String url = "jdbc:mysql://localhost:3306/mango_example";
	    String username = "root"; // 这里请使用您自己的用户名
	    String password = "root"; // 这里请使用您自己的密码
	    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	    Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

	    OrderDao dao = mango.create(OrderDao.class);
	    Order order = new Order();
	    order.setUid(100);
	    order.setStatus(1);
	    dao.add(order);
	    int id = (int) dao.addAndReturnGeneratedId(order);
	    order.setId(id);
	    System.out.println(dao.getOne(id));
	    order.setStatus(2);
	    dao.update(order);
	    System.out.println(dao.getOne(id));
	  }

	  @DB(table = "t_order")
	  interface OrderDao extends CrudDao<Order, Integer> {
	  }

	}


订单表增删改查实例2
___________________

t_order2表

.. code-block:: sql

	CREATE TABLE `t_order2` (
	  `order_id` varchar(11) NOT NULL,
	  `uid` int(11) NOT NULL,
	  `status` int(11) NOT NULL,
	  PRIMARY KEY (`order_id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Order2对象

.. code-block:: java

	public class Order2 {

	  @ID
	  private String orderId;

	  private int uid;

	  private int status;

	  // 省略get与set方法

	}

上面的实例中：

业务主键为order_id，所以将@ID注解放在了Order2类的orderId属性上。

没有自增主键，所以不使用@AutoGenerated注解。

测试代码：

.. code-block:: java

	public class OrderDao2Main {

	  public static void main(String[] args) {
	    String driverClassName = "com.mysql.jdbc.Driver";
	    String url = "jdbc:mysql://localhost:3306/mango_example";
	    String username = "root"; // 这里请使用您自己的用户名
	    String password = "root"; // 这里请使用您自己的密码
	    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	    Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

	    Order2Dao dao = mango.create(Order2Dao.class);
	    Order2 order2 = new Order2();
	    String orderId = RandomUtils.randomStringId(10);
	    order2.setOrderId(orderId);
	    order2.setUid(100);
	    order2.setStatus(1);
	    dao.add(order2);
	    System.out.println(dao.getOne(orderId));
	    order2.setStatus(2);
	    dao.update(order2);
	    System.out.println(dao.getOne(orderId));
	  }

	  @DB(table = "t_order2")
	  interface Order2Dao extends CrudDao<Order2, String> {
	  }

	}

订单表增删改查实例3
___________________

t_order3表

.. code-block:: sql

	CREATE TABLE `t_order3` (
	  `id` int(11) NOT NULL AUTO_INCREMENT, 
	  `order_id` varchar(11) NOT NULL,      
	  `uid` int(11) NOT NULL,
	  `status` int(11) NOT NULL,
	  PRIMARY KEY (`id`),
	  KEY `key_order_id` (`order_id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;

Order3对象

.. code-block:: java

	public class Order3 {

	  @AutoGenerated
	  private int id;

	  @ID
	  private String orderId;

	  private int uid;

	  private int status;

	  // 省略get与set方法

	}

上面的实例中：

业务主键为order_id，所以将@ID注解放在了Order3类的orderId属性上。

自增主键为id，所以将@AutoGenerated注解放在了Order3类的id属性上。

测试代码：

.. code-block:: java

	public class OrderDao3Main {

	  public static void main(String[] args) {
	    String driverClassName = "com.mysql.jdbc.Driver";
	    String url = "jdbc:mysql://localhost:3306/mango_example";
	    String username = "root"; // 这里请使用您自己的用户名
	    String password = "root"; // 这里请使用您自己的密码
	    DataSource ds = new DriverManagerDataSource(driverClassName, url, username, password);
	    Mango mango = Mango.newInstance(ds); // 使用数据源初始化mango

	    Order3Dao dao = mango.create(Order3Dao.class);
	    Order3 order3 = new Order3();
	    String orderId = RandomUtils.randomStringId(10);
	    order3.setOrderId(orderId);
	    order3.setUid(100);
	    order3.setStatus(1);
	    dao.add(order3);
	    System.out.println(dao.getOne(orderId));
	    order3.setStatus(2);
	    dao.update(order3);
	    System.out.println(dao.getOne(orderId));
	  }

	  @DB(table = "t_order3")
	  interface Order3Dao extends CrudDao<Order3, String> {
	  }

	}


@Column与@Ignore
________________

OrderB表

.. code-block:: sql

	CREATE TABLE `t_order_b` (
	  `id` int(11) NOT NULL,
	  `uid` int(11) NOT NULL,
	  `status` int(11) NOT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;

OrderB对象

.. code-block:: java

	public class OrderB {

	  @ID
	  private int id;

	  @Column("uid")
	  private int userId;

	  private int status;

	  @Ignore
	  private String address;

	  // 省略get与set方法

	}

`@Column <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/Column.java>`_ 注解，将OrderB类的userId属性映射到t_order_b表的uid字段中

由于t_order_b表中没有address字段，我们使用 `@Ignore <https://github.com/jfaster/mango/blob/master/src/main/java/org/jfaster/mango/annotation/Ignore.java>`_ 注解，忽略OrderB类的address属性

基于方法名操作数据
__________________

CrudDao接口提供了简单的数据操作方法，如何进行复杂的自定义数据操作？

下面是使用SQL进行数据操作的代码：

.. code-block:: java

	@DB(table = "t_order")
	public interface OrderSqlDao {

	  @SQL("select id, uid, status from #table where id = :1")
	  Order findById(int id);

	  @SQL("select id, uid, status from #table where uid = :1")
	  List<Order> findByUid(int uid);

	  @SQL("select id, uid, status from #table where id = :1 and uid = :2")
	  Order findByIdAndUid(int id, int uid);

	  @SQL("select id, uid, status from #table where id = :1 or uid = :2")
	  Order findByIdOrUid(int id, int uid);

	  @SQL("select count(1) from #table where uid = :1")
	  int countByUid(int uid);

	  @SQL("delete from #table where uid = :1")
	  int deleteByUid(int uid);

	}

上面的代码与下面完全等价：

.. code-block:: java

	@DB(table = "t_order")
	public interface OrderNoSqlDao extends CrudDao<Order, Integer> {

	  Order findById(int id);

	  List<Order> findByUid(int uid);

	  Order findByIdAndUid(int id, int uid);

	  List<Order> findByIdOrUid(int id, int uid);

	  int countByUid(int uid);

	  int deleteByUid(int uid);

	}

mango框架提供使用方法名的方式进行自定义操作：

方法名以getBy,findBy,queryBy,selectBy开头表示查询

方法名以countBy开头表示计数

方法名以deleteBy,removeBy开头表示删除

以 **findById** 为例，findBy后面的关键字为Id，表示根据id查询，findById会被转化为SQL：*select id, uid, status from #table where id = :1*

以 **findByIdAndUid** 为例，findBy后面的关键字为IdAndUid，表示根据id和uid查询，findByIdAndUid会被转化为SQL：*select id, uid, status from #table where id = :1 and uid = :2*

以 **countByUid** 为例，countBy后面的关键字为Uid，表示根据uid计数，countByUid会被转化为SQL：*select count(1) from #table where uid = :1*

以 **deleteByUid** 为例，deleteBy后面的关键字为Uid，表示根据uid删除，deleteByUid会被转化为SQL：*delete from #table where uid = :1*

下面是常用的关键字-SQL对应表：

===============================    ========================================    ============================================
关键字                              样例                                         对应SQL       
===============================    ========================================    ============================================
And                                findByIdAndName                             … where id = :1 and name = :2
Or                                 findByIdOrName                              … where id = :1 or name = :2
Equals                             findById,findByIdEquals                     … where id = :1
Between                            findByStartDateBetween                      … where startDate between :1 and :2
LessThan                           findByAgeLessThan                           … where age < :1
LessThanEqual                      findByAgeLessThanEqual                      … where age <= :1
GreaterThan                        findByAgeGreaterThan                        … where age > :1
GreaterThanEqual                   findByAgeGreaterThanEqual                   … where age >= :1
IsNull                             findByAgeIsNull                             … where age is null
NotNull                            findByAgeNotNull                            … where age not null
OrderBy                            findByAgeOrderByIdDesc                      … where age = :1 order by id desc
Not                                findByLastnameNot                           … where lastname <> :1
In                                 findByAgeIn(Collection<Age> ages)           … where age in (:1)
NotIn                              findByAgeNotIn(Collection<Age> ages)        … where age not in (:1)
True                               findByActiveTrue                            … where active = true
False                              findByActiveFalse                           … where active = false
===============================    ========================================    ============================================

带分页的基于方法名操作数据
__________________________

请先查看 :ref:`排序与分页`

分页查询的代码如下：

.. code-block:: java

	@DB(table = "t_order")
	public interface OrderPageNoSqlDao extends CrudDao<Order, Integer> {

	  List<Order> findByUid(int uid, Page page);

	  List<Order> findByIdOrUid(int id, int uid, Page page);

	}

查看完整示例代码和表结构
________________________

**增删改查NoSQL** 的所有代码和表结构均可以在 `mango-example <https://github.com/jfaster/mango-example/tree/master/src/main/java/org/jfaster/mango/example/crud>`_ 中找到。

