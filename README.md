# Ding-Notes

### Mock static methods

We can use **PowerMock** to mock static methods. In this section we will see how we can mock a static method using PowerMock. We will use java.util.UUID class for this. A UUID represents an immutable universally unique identifier (128 bit value). More details about this class can be found here: UUID Java Docs. In this class there is a method called randomUUID(). It is used to retrieve a type 4 (pseudo randomly generated) UUID. The UUID is generated using a cryptographically strong pseudo random number generator.
We will create a simple method in the UserController class to create random user id.
```
public String createUserId(User user) {
  return String.format("%s_%s", user.getSurname(), UUID.randomUUID().toString());
}
// To tests this we will create a new test method in the UserControllerTest class.
@Test
public void testMockStatic() throws Exception {
  PowerMock.mockStaticPartial(UUID.class, "randomUUID");
  EasyMock.expect(UUID.randomUUID()).andReturn(UUID.fromString("067e6162-3b6f-4ae2-a171-2470b63dff00"));
  PowerMock.replayAll();
  UserController userController = new UserController();
  Assert.assertTrue(userController.createUserId(getNewUser()).contains("067e6162-3b6f-4ae2-a171-2470b63dff00"));
  PowerMock.verifyAll();
}
```
First we will call the mockStaticPartial() method of the org.powermock.api.easymock.PowerMock class passing the class and the static method name as string which we want to mock:
```
PowerMock.mockStaticPartial(UUID.class, "randomUUID");
```
Then we will define the expectation by calling the expect method of EasyMock and returning the test value of the random UUID.
```
EasyMock.expect(UUID.randomUUID()).andReturn(UUID.fromString("067e6162-3b6f-4ae2-a171-2470b63dff00"));
```
Now we will call the replayAll() method of PowerMock.
```
PowerMock.replayAll();
```
It replays all classes and mock objects known by **PowerMock**. This includes all classes that are prepared for test using the `PrepareForTest` or `PrepareOnlyThisForTest` annotations and all classes that have had their static initializers removed by using the `SuppressStaticInitializationFor` annotation. It also includes all mock instances created by PowerMock such as those created or used by createMock(Class, Method...), mockStatic(Class, Method...), expectNew(Class, Object...), createPartialMock(Class, String...) etc.

To make it easy to pass in additional mocks not created by the PowerMock API you can optionally specify them as additionalMocks. These are typically those mock objects you have created using pure EasyMock or EasyMock class extensions. No additional mocks needs to be specified if you’re only using PowerMock API methods. The additionalMocks are also automatically verified when invoking the verifyAll() method.
Now we will call the createUserId() method of the UserController class by passing in the test user details.
```
Assert.assertTrue(userController.createUserId(getNewUser()).contains("067e6162-3b6f-4ae2-a171-2470b63dff00"));
```
In the end we will call the verifyAll()
```
PowerMock.verifyAll();
```
It verifies all classes and mock objects known by PowerMock. This includes all classes that are prepared for test using the `PrepareForTest` or `PrepareOnlyThisForTest` annotations and all classes that have had their static initializers removed by using the `SuppressStaticInitializationFor` annotation. It also includes all mock instances created by PowerMock such as those created or used by `createMock(Class, Method...), mockStatic(Class, Method...), expectNew(Class, Object...), createPartialMock(Class, String...)` etc. Note that all additionalMocks passed to the `replayAll(Object...)` method are also verified here automatically.



### Mock private methods
In this section we will see how we can mock a private method using PowerMock. It’s a very useful feature which uses java reflection. We will define a public method in out UserController class as below:
```
public String getGreetingText(User user) {
  return String.format(getGreetingFormat(), user.getFirstName(), user.getSurname());
}
// As we can see that this method calls a private method getGreetingFormat(), which is defined as below:
private String getGreetingFormat() {
  return "Hello %s %s";
}
```
We will try to change the behavior of this private method using PowerMock.
You can create **spies** of real objects. When you use the spy then the real methods are called (unless a method was stubbed). Spying on real objects can be associated with “partial mocking” concept. We will first create a spy on the `UserController` class.
```
UserController spy = spy(new UserController());
```
Then we will define how the private method should behave when it’s been called by using org.powermock.api.mockito.PowerMockito.when.
```
when(spy, method(UserController.class, "getGreetingFormat")).withNoArguments().thenReturn("Good Morning %s %s");
```
Now we will call the public method (which uses the private method) on the spied object
```
assertEquals("Good Morning Code Geeks", spy.getGreetingText(user));
```
Below is the code snippet of the whole method
```
@Test
public void testMockPrivateMethod() throws Exception {
  UserController spy = spy(new UserController());
  when(spy, method(UserController.class, "getGreetingFormat")).withNoArguments().thenReturn("Good Morning %s %s");
  User user = new User();
  user.setFirstName("Code");
  user.setSurname("Geeks");
  assertEquals("Good Morning Code Geeks", spy.getGreetingText(user));
}
```
Reference from: https://examples.javacodegeeks.com/core-java/powermockito/powermockito-tutorial-beginners/


Based on the docs, you're not using this correctly:https://github.com/powermock/powermock/wiki/MockitoUsage
From the docs:
	1.	Annotate the class with @PrepareForTest(Static.class) (You're not doing this)
	2.	Call mockStatic PowerMockito.mockStatic(Static.class);
	3.	Use Mockito.when(). You are using PowerMockito.when()

https://stackoverflow.com/questions/41287948/org-mockito-exceptions-misusing-missingmethodinvocationexception-when-require

```
@RunWith(PowerMockRunner.class)
@PrepareForTest({ IVWConfigurationLoader.class })
public class NotificationFacadeTest {
    private NotificationFacade notificationFacade;

    @Before
    public void setUp() {
        notificationFacade = new NotificationFacade();
    }

    @Test
    public void shouldFilterReceiverList_ForEmail_AllMatch() {
        // Arrange
        Map<String, String> map = new HashMap<>();
        map.put(IVWConfigurationLoader.EMAIL_WHITE_LIST, ".*@aimsio.com|some@gmail.com");
        PowerMockito.mockStatic(IVWConfigurationLoader.class);
        Mockito.when(IVWConfigurationLoader.getConfigurationProperties()).thenReturn(map);

        List<String> receivers = Arrays.asList("temp@aimsio.com", "some@gmail.com");

        // Act
        List<String> result = notificationFacade.filterReceiverList(receivers, IVWConfigurationLoader.EMAIL_WHITE_LIST);

        // Assert
        Assert.assertTrue(result.isEmpty());
    }
}
```

### What is AtomicReference?
http://a-developers-notes.blogspot.ca/2012/03/what-is-atomicreference-for-really.html
http://tutorials.jenkov.com/java-util-concurrent/atomicreference.html
Best atomic reference explanation

### Double-checked locking
https://en.wikipedia.org/wiki/Double-checked_locking

In software engineering, double-checked locking (also known as "double-checked locking optimization") is a **software design pattern** used to reduce the overhead of acquiring a lock by first testing the locking criterion (the "lock hint") without actually acquiring the lock. Only if the locking criterion check indicates that locking is required does the actual locking logic proceed.
The pattern, when implemented in some language/hardware combinations, can be unsafe. At times, it can be considered an Show more…

“One of the dangers of using double-checked locking in J2SE 1.4 (and earlier versions) is that it will often appear to work: it is not easy to distinguish between a correct implementation of the technique and one that has subtle problems. Depending on the compiler, the interleaving of threads by the scheduler and the nature of other concurrent system activity, failures resulting from an incorrect implementation of double-checked locking may only occur intermittently. Reproducing the failures can be difficult.”

“As of J2SE 5.0, this problem has been fixed. The volatile keyword now ensures that multiple threads handle the singleton instance correctly. ”
Volatile makes the multi threads or multi cores to have one single instance of the variable and each thread is just having a reference to that variable, so once the variable is changed, the cache inside each thread is surely changed. So, instead of storing the variable’s value, the threads are only storing the variable’s reference.

As of J2SE 5.0, this problem has been fixed. The volatile keyword now ensures that multiple threads handle the singleton instance correctly. This new idiom is described in [4] and [5].

```
// Works with acquire/release semantics for volatile in Java 1.5 and later
// Broken under Java 1.4 and earlier semantics for volatile
class Foo {
    private volatile Helper helper;
    public Helper getHelper() {
        Helper result = helper;
        if (result == null) {
            synchronized(this) {
                result = helper;
                if (result == null) {
                    helper = result = new Helper();
                }
            }
        }
        return result;
    }

    // other functions and members...
}
```

### When to use AtomicReference in Java?
https://stackoverflow.com/questions/3964211/when-to-use-atomicreference-in-java
**Atomic reference** should be used in a setting where you need to do simple atomic (i.e., thread safe, non-trivial) operations on a reference, for which monitor-based synchronization is not appropriate. Suppose you want to check to see if a specific field only if the state of the object remains as you last checked:
```
AtomicReference<Object> cache = new AtomicReference<Object>();

Object cachedValue = new Object();
cache.set(cachedValue);

//... time passes ...
Object cachedValueToUpdate = cache.get();
//... do some work to transform cachedValueToUpdate into a new version
Object newValue = someFunctionOfOld(cachedValueToUpdate);
boolean success = cache.compareAndSet(cachedValue,cachedValueToUpdate);
```
Because of the atomic reference semantics, you can do this even if the cache object is shared amongst threads, without using synchronized. In general, you're better off using synchronizers or the java.util.concurrent framework rather than bare Atomic* unless you know what you're doing.

Two excellent dead-tree references which will introduce you to this topic: **Herlihy's excellent Art of Multiprocessor Programming and Java Concurrency in Practice**.

Note that (I don't know if this has always been true) reference assignment (i.e., =) is itself atomic (updating primitive 64-bit types(long/double) may not be atomic; but updating a reference is always atomic, even if it's 64 bit) without explicitly using an Atomic*. See the JLS 3ed, Section 17.7, …

### To use an AtomicReference:
```
public static AtomicReference<String> shared = new AtomicReference<>();
String init="Inital Value";
shared.set(init);
//now we will modify that value
boolean success=false;
while(!success){
  String prevValue=shared.get();
  // do all the work you need to
  String newValue=shared.get()+"lets add something";
  // Compare and set
  success=shared.compareAndSet(prevValue,newValue);
}
```
Now why is this better? Honestly that code is a little less clean than before. But there is something really important that happens under the hood in AtomicRefrence, and that is compare and swap. It is a single CPU instruction, not an OS call, that makes the switch happen. That is a single instruction on the CPU. And because there are no locks, there is no context switch in the case where the lock gets exercised which saves even more time!

The catch is, for AtomicReferences, this does not use a .equals() call, but instead an == comparison for the expected value. So make sure the expected is the actual object returned from get in the loop.

### SimpleDateFormat
SimpleDateFormat stores **intermediate** results in instance fields. So if one instance is used by two threads they can mess each other's results.

Looking at the source code reveals that there is a Calendar instance field, which is used by operations on DateFormat / SimpleDateFormat

For example parse(..) calls calendar.clear() initially and then calendar.add(..). If another thread invokes parse(..) before the completion of the first invocation, it will clear the calendar, but the other invocation will expect it to be populated with intermediate results of the calculation.

One way to **reuse date formats** without trading **thread-safety** is to put them in a **ThreadLocal** - some libraries do that. That's if you need to use the same format multiple times within one thread. But in case you are using a servlet container (that has a thread pool), remember to clean the thread-local after you finish.
To be honest, I don't understand why they need the instance field, but that's the way it is. You can also use `jodatime DateTimeFormat` which is **threadsafe**.
https://stackoverflow.com/questions/6840803/why-is-javas-simpledateformat-not-thread-safe

### regex
You need `test.split("\\|")`;
split uses regular expression and in regex | is a metacharacter representing the OR operator. You need to escape that character using \ (written in String as "\\" since \ is also a metacharacter in String literals and require another \ to escape it).

You can also use `test.split(Pattern.quote("|"));`
and let `Pattern.quote` create the escaped version of the regex representing |.

### differences among grep, awk and sed [duplicate]
Short definition:
grep: search for specific terms in a file
```
#usage
$ grep This file.txt
Every line containing "This"
Every line containing "This"
Every line containing "This"
Every line containing "This"

$ cat file.txt
Every line containing "This"
Every line containing "This"
Every line containing "That"
Every line containing "This"
Every line containing "This"
```
Now **awk** and **sed** are completly different than grep. awk and sed are **text processors**. Not only do they have the ability to find what you are looking for in text, they have the ability to _remove, add and modify_ the text as well (and much more). 

**awk** is mostly used for _data extraction and reporting_. **sed** is a stream editor. Each one of them has its own functionality and specialties.

Example **Sed**
```
$ sed -i 's/cat/dog/' file.txt
# this will replace any occurrence of the characters 'cat' by 'dog'
```
**Awk**
```
$ awk '{print $2}' file.txt
# this will print the second column of file.txt
```
Basic awk usage: **Compute sum/average/max/min/etc. what ever you may need.**
```
$ cat file.txt
A 10
B 20
C 60
$ awk 'BEGIN {sum=0; count=0; OFS="\t"} {sum+=$2; count++} END {print "Average:", sum/count}' file.txt
Average:    30
```
I recommend that you read this book: <<Sed & Awk: 2nd Ed>>.
It will help you become a proficient sed/awk user on any unix-like environment.

### diff between checkNotNull && @Nonnull
checkNotNull is ran at runtime but @Nonnull lets intellij idea plugin to detect if a null value is passed (depending to the code) and complain about that (by high lighting that piece of code) :)

```
@Rule
    public ExpectedException expectedException = ExpectedException.none();

@Test
    public void shouldThrowExceptionWhenMoreThanOneCustomerIntegrationIsFound() {
        // Arrange
        CustomerIntegration customerIntegration1 = new CustomerIntegration();
        customerIntegration1.setOwner(fieldTicketingCustomer);
        customerIntegration1.setGuid(fieldTicketTestValues.generateUUIDString());
        customerIntegration1.setUname("junitIntegration1");

        CustomerIntegration customerIntegration2 = new CustomerIntegration();
        customerIntegration2.setOwner(fieldTicketingCustomer);
        customerIntegration2.setGuid(fieldTicketTestValues.generateUUIDString());
        customerIntegration2.setUname("junitIntegration2");

        myEntityManagerFactoryFacade.withEntityManagerTransaction(em -> customerIntegrationFacade
                .saveCustomerIntegration(
                em,
                customerIntegration1));
        myEntityManagerFactoryFacade.withEntityManagerTransaction(em -> customerIntegrationFacade
                .saveCustomerIntegration(
                em,
                customerIntegration2));

        expectedException.expect(IVWMatchers.isA(IVWException.class,
                IVWMatchers.hasField(IVWException::getMessage,
                        "message",
                        equalTo(String.format(
                                "There is more than one CustomerIntegration for the customer with GUID " + "%s",
                                fieldTicketingCustomer.getGuid())))));

        // Act
        myEntityManagerFactoryFacade.withEntityManagerTransaction(em -> customerIntegrationFacade
                .getCustomerIntegration(
                em,
                fieldTicketingCustomer));

        // Assert
        fail("Expected an exception");
    }
```

```
select guid, count(*) cnt From `integration_import_result` group by 1 having cnt > 1;
select * From `integration_import_result`;
```


### Vaadin Table Example
http://www.tnwblog.com/vaadin-table-example.html
```
//create table instance
    Table table = new Table("My table");
    //add table columns
    table.addContainerProperty("Numeric column", Integer.class, null);
    table.addContainerProperty("String column", String.class, null);
    table.addContainerProperty("Date column", Date.class, null);
    //add table data (rows)
    table.addItem(new Object[]{new Integer(100500), "this is first", new Date()}, new Integer(1));
    table.addItem(new Object[]{new Integer(100501), "this is second", new Date()}, new Integer(2));
    table.addItem(new Object[]{new Integer(100502), "this is third", new Date()}, new Integer(3));
    table.addItem(new Object[]{new Integer(100503), "this is forth", new Date()}, new Integer(4));
    table.addItem(new Object[]{new Integer(100504), "this is fifth", new Date()}, new Integer(5));
    table.addItem(new Object[]{new Integer(100505), "this is sixth", new Date()}, new Integer(6));
```

### Guava MultiMap
`Map<String,List<MyClass>> myClassListMap  = new HashMap<String,List<MyClass>>()`
http://tomjefferys.blogspot.ca/2011/09/multimaps-google-guava.html
```
public class MutliMapTest {
  public static void main(String... args) {
	  Multimap<String, String> myMultimap = ArrayListMultimap.create();

	  // Adding some key/value
	  myMultimap.put("Fruits", "Bannana");
	  myMultimap.put("Fruits", "Apple");
	  myMultimap.put("Fruits", "Pear");
	  myMultimap.put("Vegetables", "Carrot");

	  // Getting the size
	  int size = myMultimap.size();
	  System.out.println(size);  // 4

	  // Getting values
	  Collection<string> fruits = myMultimap.get("Fruits");
	  System.out.println(fruits); // [Bannana, Apple, Pear]

	  Collection<string> vegetables = myMultimap.get("Vegetables");
	  System.out.println(vegetables); // [Carrot]

	  // Iterating over entire Mutlimap
	  for(String value : myMultimap.values()) {
	   System.out.println(value);
	  }

	  // Removing a single value
	  myMultimap.remove("Fruits","Pear");
	  System.out.println(myMultimap.get("Fruits")); // [Bannana, Pear]

	  // Remove all values for a key
	  myMultimap.removeAll("Fruits");
	  System.out.println(myMultimap.get("Fruits")); // [] (Empty Collection!)
  }
}
```


### Arrays.asList VS. new ArrayList<>()
When you call Arrays.asList it does not return a java.util.ArrayList. It returns a java.util.Arrays$ArrayList which is an immutable list. You cannot add to it and you cannot remove from it.

If you try to add or remove elements from them you will get UnsupportedOperationException
In fact, I'll expand my comment a little bit,
One problem that can occur if you use **asList** as it wasn't different from ArrayList object:
```
List list = Arrays.asList(array) ;
list.remove(0); //UnsupportedOperationException :(
```
Here you cannot remove the 0 element because asList returns a fixed-size list backed by the specified array. So you should do something like:
```
List newList = new ArrayList(Arrays.asList(array));
```
in order to make the newList modifiable.

### SQL DATE DIFF
SELECT DATEDIFF(‘2008-05-20','2008-05-19');  1 day
SELECT DATEDIFF(‘2008-05-20','2008-05-20');  0 day
SELECT DATEDIFF(‘2008-05-18','2008-05-20');  -2 day
SELECT DATE_SUB("2017-06-15 09:34:21", INTERVAL 15 MINUTE);    2017-06-15 09:19:21
SELECT DATE_SUB("2017-06-15 09:34:21", INTERVAL 3 HOUR); 2017-06-15 06:34:21
SELECT DATE_SUB("2017-06-15", INTERVAL -2 MONTH); 2017-08-15

### select uuid();  — random guid generator

### SQL DATE
```
SELECT CURDATE();   2018-01-05
select now();    2018-01-05 16:38:52
```

### SELECT COALESCE(NULL,1); 
Returns the first non-NULL value in the list, or NULL if there are no non-NULL values.
The return type of COALESCE() is the aggregated type of the argument types.

### Aggregate Functions (MySQL)
Some functions, such as SUM, are used to perform calculations on a group of rows, these are called aggregate functions.  In most cases these functions operate on a group of values which are defined using the GROUP BY clause.  When there isn’t a GROUP BY clause, it is generally understood the aggregate function applies to all filtered results.
Some of the most common aggregate functions include:

These functions can be used on their own on in conjunction with the GROUP BY clause.  On their own, they operate across the entire table; however, when used with GROUP BY, their calculations are “reset” each time the grouping changes.  In this manner they act as subtotals.
https://www.essentialsql.com/get-ready-to-learn-sql-6-how-to-group-and-summarize-your-results/
https://docs.thunderstone.com/site/texisman/summarizing_values.html
```
select GREATEST('2018-01-04 22:00:00', null);  ->  null
select LEAST('2018-01-04 22:00:00', null);   -> null
select LEAST('2018-01-04 22:00:00', ‘2018-01-03');  -> 2018-01-03
SELECT count(er.id), er.`EMPLOYEE_ID`, job.jobNo FROM EmployeeRecord er LEFT JOIN JobOrder job ON er.`JOB_ORDER_ID` = job.`id` WHERE job.`OWNER_GUID` = 'b7c08a16-20c3-4062-9f64-26d1be2b6e21' GROUP BY er.`EMPLOYEE_ID`, er.`JOB_ORDER_ID`;  // bcflynn

SELECT job.jobNo AS `Job No.`, CONCAT(em.`firstName`, ' ', em.`lastName`) AS `Employee`, 
er.`start_date`, er.`end_date`, SUM(DATEDIFF(COALESCE(er.`end_date`,CURDATE()), er.`start_date`)) AS `Duration (Days)`  
FROM EmployeeRecord er 
LEFT JOIN JobOrder job ON er.`JOB_ORDER_ID` = job.`id` 
LEFT JOIN Employee em ON er.`EMPLOYEE_ID` = em.`id` 
WHERE job.`jobNo` = 2853 AND job.`OWNER_GUID` = 'b7c08a16-20c3-4062-9f64-26d1be2b6e21' GROUP BY job.`jobNo`, `Employee`;
```
```
INSERT INTO `Report_Entity` (`guid`, `name`, `uname`, `query`, `active`, `showIndexColumn`, `hierarchyColumn`, `createNewRowForHierarhcy`, `owner_id`, `OWNER_GUID`, `isStandalone`, `VIEW_PERMISSION_ID`, `VIEW_PERMISSION_GUID`)
VALUES
	('46e65716-f25c-11e7-a03c-bf121c5863e6', 'Employee Longevity Report', 'employee_longevity_report', '', 1, 0, NULL, 1, NULL, 'b7c08a16-20c3-4062-9f64-26d1be2b6e21', 1, NULL, NULL);

INSERT INTO `Report_Entity_Parameter` (`guid`, `index`, `paramName`, `type`, `title`, `defaultOp`, `defaultVal`, `extra`, `extra2`, `extra3`, `extra4`, `positionInUI`, `requiresNewLine`, `VIEW_PERMISSION_GUID`, `owner_guid`)
VALUES
	('23275124-f25f-11e7-a03c-bf121c5863e6', 0, 'job', 'ComboBox', 'Job No.', '', '-1', 'SELECT \'All\' m_key, -1 m_value\n\nUNION\n\n(SELECT jobNo m_key, id m_value FROM JobOrder\nWHERE OWNER_GUID = \'b7c08a16-20c3-4062-9f64-26d1be2b6e21\'\nAND status NOT IN (\'Deleted\', \'Finished\'))\n\nORDER BY m_key DESC;', '', '', '', 'Left', 0, NULL, '46e65716-f25c-11e7-a03c-bf121c5863e6');
```
```
SELECT job.jobNo AS `Job No.`, CONCAT(em.`firstName`, ' ', em.`lastName`) AS `Employee`, 
SUM(DATEDIFF(LEAST(COALESCE(er.`end_date`,CURDATE()), CURDATE()), LEAST(er.`start_date`, CURDATE()))) AS `Duration (Days)`  
FROM EmployeeRecord er 
LEFT JOIN JobOrder job ON er.`JOB_ORDER_ID` = job.`id` 
LEFT JOIN Employee em ON er.`EMPLOYEE_ID` = em.`id` 
WHERE job.`jobNo` = 2853 AND job.`OWNER_GUID` = 'b7c08a16-20c3-4062-9f64-26d1be2b6e21' 
GROUP BY job.`jobNo`, `Employee`;
```
```
INSERT INTO `Report_Entity` (`guid`, `name`, `uname`, `query`, `active`, `showIndexColumn`, `hierarchyColumn`, `createNewRowForHierarhcy`, `owner_id`, `OWNER_GUID`, `isStandalone`, `VIEW_PERMISSION_ID`, `VIEW_PERMISSION_GUID`)
VALUES
    ('46e65716-f25c-11e7-a03c-bf121c5863e6', 'Employee Longevity Report', 'employee_longevity_report', '', 1, 0, NULL, 1, NULL, 'b7c08a16-20c3-4062-9f64-26d1be2b6e21', 1, NULL, NULL);
```
```
INSERT INTO `Report_Entity_Parameter` (`guid`, `index`, `paramName`, `type`, `title`, `defaultOp`, `defaultVal`, `extra`, `extra2`, `extra3`, `extra4`, `positionInUI`, `requiresNewLine`, `VIEW_PERMISSION_GUID`, `owner_guid`)
VALUES
    ('23275124-f25f-11e7-a03c-bf121c5863e6', 0, 'job', 'ComboBox', 'Job No.', '', '-1', 'SELECT \'All\' m_key, -1 m_value\n\nUNION\n\n(SELECT jobNo m_key, id m_value FROM JobOrder\nWHERE OWNER_GUID = \'b7c08a16-20c3-4062-9f64-26d1be2b6e21\'\nAND status NOT IN (\'Deleted\', \'Finished\'))\n\nORDER BY m_key DESC;', '', '', '', 'Left', 0, NULL, '46e65716-f25c-11e7-a03c-bf121c5863e6');
```
```
INSERT INTO `Report_Entity_Parameter` (`guid`, `index`, `paramName`, `type`, `title`, `defaultOp`, `defaultVal`, `extra`, `extra2`, `extra3`, `extra4`, `positionInUI`, `requiresNewLine`, `VIEW_PERMISSION_GUID`, `owner_guid`)
VALUES
    ('25288c04-f4cb-11e7-9358-b96beb5d0c26', 1, 'customer', 'CurrentCustomer', 'CurrentCustomer', '', '', '', '', '', '', 'None', 0, NULL, '46e65716-f25c-11e7-a03c-bf121c5863e6');
```

```
INSERT INTO `customer_shared_report` (`guid`, `CUSTOMER_GUID`, `REPORT_ENTITY_GUID`)
VALUES
	('5f7f9f14-f584-11e7-b739-3b15ba66d1e4', 'b7c08a16-20c3-4062-9f64-26d1be2b6e21', ‘46e65716-f25c-11e7-a03c-bf121c5863e6');
```


### select UTC_TIMESTAMP();  // 2018-01-10 21:21:43
```
select er.`start_date` AS `start date`, er.end_date AS `end date`, CONCAT(em.`firstName`, ' ', em.`lastName`) AS `employee` from EmployeeRecord er JOIN Employee em ON er.`EMPLOYEE_ID` = em.`id` where er.`JOB_ORDER_ID` = ‘2121';
```

### The difference between Procedure and Function in MySQL?

The most general difference between procedures and functions is that they are invoked differently and for different purposes:

1. A procedure does not return a value. Instead, it is invoked with a CALL statement to perform an operation such as modifying a table or processing retrieved records.

2. A function is invoked within an expression and returns a single value directly to the caller to be used in the expression.

3. You cannot invoke a function with a CALL statement, nor can you invoke a procedure in an expression.

Syntax for routine creation differs somewhat for procedures and functions:

1. Procedure parameters can be defined as input-only, output-only, or both. This means that a procedure can pass values back to the caller by using output parameters. These values can be accessed in statements that follow the CALL statement. Functions have only input parameters. As a result, although both procedures and functions can have parameters, procedure parameter declaration differs from that for functions.

2. Functions return value, so there must be a RETURNS clause in a function definition to indicate the data type of the return value. Also, there must be at least one RETURN statement within the function body to return a value to the caller. RETURNS and RETURN do not appear in procedure definitions.

3. To invoke a stored procedure, use the CALL statement. To invoke a stored function, refer to it in an expression. The function returns a value during expression evaluation.

4. A procedure is invoked using a CALL statement, and can only pass back values using output variables. A function can be called from inside a statement just like any other function (that is, by invoking the function's name), and can return a scalar value.

5. **Specifying a parameter as IN, OUT, or INOUT is valid only for a PROCEDURE. For a FUNCTION, parameters are always regarded as IN parameters.**

6. **If no keyword is given before a parameter name, it is an IN parameter by default. Parameters for stored functions are not preceded by IN, OUT, or INOUT. All function parameters are treated as IN parameters.**

To define a stored procedure or function, use CREATE PROCEDURE or CREATE FUNCTION respectively:
```
CREATE PROCEDURE proc_name ([parameters])
 [characteristics]
 routine_body
```
```
CREATE FUNCTION func_name ([parameters])
 RETURNS data_type       // diffrent
 [characteristics]
 routine_body
 ```
A MySQL extension for stored procedure (not functions) is that a procedure can generate a result set, or even multiple result sets, which the caller processes the same way as the result of a SELECT statement. However, the contents of such result sets cannot be used directly in expression.

Stored routines (referring to both stored procedures and stored functions) are associated with a particular database, just like tables or views. When you drop a database, any stored routines in the database are also dropped.
Stored procedures and functions do not share the same namespace. It is possible to have a procedure and a function with the same name in a database.

In Stored procedures dynamic SQL can be used but not in functions or triggers.

SQL prepared statements (PREPARE, EXECUTE, DEALLOCATE PREPARE) can be used in stored procedures, but not stored functions or triggers. Thus, stored functions and triggers cannot use Dynamic SQL (where you construct statements as strings and then execute them). (Dynamic SQL in MySQL stored routines)

### Some more interesting differences between FUNCTION and STORED PROCEDURE:

1. (This point is copied from a blogpost.) Stored procedure is precompiled execution plan where as functions are not. Function Parsed and compiled at runtime. Stored procedures, Stored as a pseudo-code in database i.e. compiled form.

2. (I'm not sure for this point.)**Stored procedure has the security and reduces the network traffic and also we can call stored procedure in any no. of applications at a time. reference**

3. Functions are normally used for computations where as procedures are normally used for executing business logic.

4. Functions Cannot affect the state of database (Statements that do explicit or implicit commit or rollback are disallowed in function) Whereas Stored procedures Can affect the state of database using commit etc. refrence: J.1. Restrictions on Stored Routines and Triggers

5. Functions can't use FLUSH statements whereas Stored procedures can do.

6. Stored functions cannot be recursive Whereas Stored procedures can be. Note: Recursive stored procedures are disabled by default, but can be enabled on the server by setting the max_sp_recursion_depth server system variable to a nonzero value. See Section 5.2.3, “System Variables”, for more information.

7. Within a stored function or trigger, it is not permitted to modify a table that is already being used (for reading or writing) by the statement that invoked the function or trigger. Good Example: How to Update same table on deletion in MYSQL?
	
**Note**: that although some restrictions normally apply to stored functions and triggers but not to stored procedures, those restrictions do apply to stored procedures if they are invoked from within a stored function or trigger. For example, although you can use FLUSH in a stored procedure, such a stored procedure cannot be called from a stored function or trigger.

```
DELIMITER //

CREATE FUNCTION IncomeLevel ( monthly_value INT )
RETURNS varchar(20)

BEGIN

   DECLARE income_level varchar(20);

   IF monthly_value <= 4000 THEN
      SET income_level = 'Low Income';

   ELSEIF monthly_value > 4000 AND monthly_value <= 7000 THEN
      SET income_level = 'Avg Income';

   ELSE
      SET income_level = 'High Income';

   END IF;

   RETURN income_level;

END; //

DELIMITER ;
```
```
SELECT job.jobNo AS `Job No.`, CONCAT(em.`firstName`, ' ', em.`lastName`) AS `Employee`, 
SUM(DurationDaysCalculator(er.`start_date`, er.`end_date`, cus.`timezone`)) AS `Duration (Days)`  
FROM EmployeeRecord er 
JOIN JobOrder job ON er.`JOB_ORDER_ID` = job.`id` 
JOIN Employee em ON er.`EMPLOYEE_ID` = em.`id` 
JOIN Customer cus ON job.`OWNER_GUID` = cus.`guid` 
WHERE job.`id` = :jobId AND job.`OWNER_GUID` = :customer 
GROUP BY job.`jobNo`, `Employee` 
ORDER BY `Employee`;
```
```
DROP FUNCTION IF EXISTS DurationDaysCalculator;

DELIMITER //

CREATE FUNCTION DurationDaysCalculator(start_date TIMESTAMP, end_date TIMESTAMP, time_zone VARCHAR(255))
  RETURNS BIGINT(20)

  BEGIN

    DECLARE current_time1 TIMESTAMP;
    DECLARE real_start_date TIMESTAMP;
    DECLARE real_end_date TIMESTAMP;
    DECLARE duration_days BIGINT(20);

    SET current_time1 = UTC_TIMESTAMP();

    SET real_end_date = CONVERT_TZ(LEAST(COALESCE(end_date, current_time1), current_time1), 'UTC', time_zone);
    SET real_start_date = CONVERT_TZ(start_date, 'UTC', time_zone);

    SET duration_days = DATEDIFF(real_end_date, real_start_date);

    IF duration_days >= 0
    THEN
      SET duration_days = duration_days + 1;
    ELSE
      SET duration_days = 0;
    END IF;

    RETURN duration_days;

  END;
//

DELIMITER ;
```

### Some git stash
```
git stash list [<options>]
git stash show [<stash>]
git stash drop [-q|--quiet] [<stash>]
```

### Java DateFormat Styles:
The preceding code example specified the DEFAULT formatting style. The DEFAULT style is just one of the predefined formatting styles that the DateFormat class provides, as follows:

* DEFAULT
* SHORT
* MEDIUM
* LONG
* FULL

The following table shows how dates are formatted for each style with the U.S. and French locales:
Sample Date Formats
```
dateFormatter = DateFormat.getDateInstance(DateFormat.DEFAULT, currentLocale);
```
```
certificateListComboBox.setCaption("<font size=\"5\">Please choose a certificate that you would like to submit:</font>");
certificateListComboBox.setCaptionAsHtml(true);
```

```
select MAX(cd.`updated_at`), cd.`guid`, cd.`CERTIFICATION_TYPE_GUID`, cd.`status` from certification_data cd WHERE cd.`CREATED_BY_ID` = 392   GROUP BY cd.`CERTIFICATION_TYPE_GUID`;
select cd.* from certification_data cd WHERE cd.`CREATED_BY_ID` = 392  AND cd.`updated_at` = (SELECT max(cd2.`updated_at`) from certification_data cd2 where cd2.`CREATED_BY_ID` = 392 AND cd2.`CERTIFICATION_TYPE_GUID` = cd.`CERTIFICATION_TYPE_GUID` group by cd2.`CERTIFICATION_TYPE_GUID`) ;

select cd.* from certification_data cd WHERE cd.`CREATED_BY_ID` = 392  AND cd.`updated_at` = (SELECT max(cd2.`updated_at`) from certification_data cd2 where cd2.`CREATED_BY_ID` = 392 AND cd2.`CERTIFICATION_TYPE_GUID` = cd.`CERTIFICATION_TYPE_GUID` group by cd2.`CERTIFICATION_TYPE_GUID`) AND cd.`status` = ‘Certified';
```

```
/**
     * Gets the certification data for certification types of the given employee. If
     * certificationDataStatusList is not null, will only select the ones with certain CertificationDataStatus.
     * If for certain cert type, there are multiple cert data, will only select the most recently updated one.
     */
    public List<CertificationData> getCertificationDataForEmployeeByStatus(Employee employee,
            List<CertificationDataStatus> certificationDataStatusList) {
        try {
            if (employee == null) {
                throw new IVWException("No employee found.");
            }
            Map<String, Object> params = new HashMap<>();
            params.put("CREATED_BY_ID", employee.getLinkedUser().getId());
            String query = String.format(
                    "c.createdBy.id = :CREATED_BY_ID AND c.updatedAt = (SELECT MAX(cd2.updatedAt) FROM "
                            + "%s cd2 WHERE cd2.createdBy.id = :CREATED_BY_ID AND cd2.certificationType.guid = c"
                            + ".certificationType.guid GROUP BY cd2.certificationType.guid)",
                    CertificationData.class.getName());
            query = concatCertificationDataStatusQuery(certificationDataStatusList, query);
            return modelFacade.getListOfEntities(CertificationData.class, query, params);
        } catch (Exception e) {
            Utility.logException(e);
            IVWExceptionHelper.rethrowAsAbstractIVWException(e);
        }
        return new ArrayList<>();
    }
```
```
SELECT c FROM com.invistaware.compliance.model.CertificationData c WHERE c.createdBy.id = :CREATED_BY_ID AND c.updatedAt = (SELECT MAX(cd2.updatedAt) FROM com.invistaware.compliance.model.CertificationData cd2 WHERE cd2.createdBy.id = :CREATED_BY_ID AND cd2.certificationType.guid = c.certificationType.guid GROUP BY cd2.certificationType.guid) AND c.status IN (‘CERTIFIED’);
```

### How To Order Null Values First Or Last In Mysql?
https://www.designcise.com/web/tutorial/how-to-order-null-values-first-or-last-in-mysql

```
-- Certified certs
SELECT ct.name `Certificate Name`, CONCAT(e.firstName, ' ', e.lastName) `Employee`,
IF(ct.Type IS NULL or LENGTH(ct.Type) < 2, ct.Type, CONCAT(UCASE(LEFT(ct.Type, 1)), LCASE(SUBSTRING(ct.Type, 2)))) `Type`,
DATE_FORMAT(CONVERT_TZ(cd.`reviewed_at`, 'UTC', cu.timezone), '%Y-%m-%d') `Certified On`,
-- Expires On field is based on CertificationData.getEffectiveExpirationDate method
DATE_FORMAT(
  CONVERT_TZ(
    GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`
      , cd.`certified_date`, cd.`expiration_date`),
    'UTC', cu.timezone)
  ,'%Y-%m-%d') `Expires On`,
ct.guid `certTypeGuid`, cd.guid`certDataGuid`, e.`id` `employeeId`

FROM certification_type ct
  LEFT JOIN Customer cu ON (ct.CUSTOMER_GUID = cu.`guid`)
  LEFT JOIN certification_data cd ON (ct.guid = cd.CERTIFICATION_TYPE_GUID)
  LEFT JOIN Employee e ON (cd.CREATED_BY_ID = e.LINKED_USER_ID)
  LEFT JOIN certification_type_trades cts ON (ct.guid = cts.CERTIFICATION_TYPE_GUID)
  LEFT JOIN certification_type_clients ctc ON (ct.guid = ctc.CERTIFICATION_TYPE_GUID)
  LEFT JOIN certification_type_areas cta ON (ct.guid = cta.CERTIFICATION_TYPE_GUID)
  LEFT JOIN certification_type_jobs ctj ON (ct.guid = ctj.CERTIFICATION_TYPE_GUID)

WHERE cd.status = 'CERTIFIED'
  AND (cd.is_expired = 0)
  AND (ct.is_active = 1)
  AND (ct.is_deleted = 0)
  AND ct.CUSTOMER_GUID = :customer

  -- Report Parameters
	AND (ct.GUID like :CERT_TYPE_GUID)
	AND (-1 = :EMPLOYEE_ID OR e.ID = :EMPLOYEE_ID)
 	AND (-1 = :TRADE_ID OR (cts.TRADE_ID = :TRADE_ID AND e.JOB_ITEM_ID = cts.TRADE_ID))
	AND (-1 = :LOCATION_ID OR (cta.LOCATION_ID = :LOCATION_ID
							AND EXISTS (SELECT * FROM employee_areas ea
										WHERE ea.`EMPLOYEE_ID` = e.id AND ea.`LOCATION_ID` = cta.`LOCATION_ID`)))
	AND (-1 = :CLIENT_ID OR (ctc.CLIENT_ID = :CLIENT_ID
							AND EXISTS (SELECT * FROM EmployeeRecord ecr
												JOIN JobOrder j ON (ecr.JOB_ORDER_ID = j.id)
										WHERE ecr.`EMPLOYEE_ID` = e.id AND j.`client_id` = ctc.CLIENT_ID)))
	AND ((0 = :EXPIRY_IN_DAYS AND (GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`, cd.`certified_date`, cd.`expiration_date`) > now() OR GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`, cd.`certified_date`, cd.`expiration_date`) IS NULL)) 
    OR ( 0 <> :EXPIRY_IN_DAYS AND
		  GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`, cd.`certified_date`, cd.`expiration_date`) BETWEEN now() AND DATE_ADD(now(), INTERVAL :EXPIRY_IN_DAYS DAY)))

-- remove duplicate results
GROUP BY cd.guid

ORDER BY -`Expires On` DESC, ct.name ASC;
```

```
INSERT INTO `Report_Entity` (`guid`, `name`, `uname`, `query`, `active`, `showIndexColumn`, `hierarchyColumn`, `createNewRowForHierarhcy`, `owner_id`, `OWNER_GUID`, `isStandalone`, `VIEW_PERMISSION_ID`, `VIEW_PERMISSION_GUID`)
VALUES
	('249793b4-d6a1-11e6-ab5d-c474fb8ccc58', 'Certification Certified Report', NULL, '-- Certified certs\nSELECT ct.name `Certificate Name`, CONCAT(e.firstName, \' \', e.lastName) `Employee`,\nIF(ct.Type IS NULL or LENGTH(ct.Type) < 2, ct.Type, CONCAT(UCASE(LEFT(ct.Type, 1)), LCASE(SUBSTRING(ct.Type, 2)))) `Type`,\nDATE_FORMAT(CONVERT_TZ(cd.`reviewed_at`, \'UTC\', cu.timezone), \'%Y-%m-%d\') `Certified On`,\n-- Expires On field is based on CertificationData.getEffectiveExpirationDate method\nDATE_FORMAT(\n  CONVERT_TZ(\n    GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`\n      , cd.`certified_date`, cd.`expiration_date`),\n    \'UTC\', cu.timezone)\n  ,\'%Y-%m-%d\') `Expires On`,\nct.guid `certTypeGuid`, cd.guid`certDataGuid`, e.`id` `employeeId`\n\nFROM certification_type ct\n  LEFT JOIN Customer cu ON (ct.CUSTOMER_GUID = cu.`guid`)\n  LEFT JOIN certification_data cd ON (ct.guid = cd.CERTIFICATION_TYPE_GUID)\n  LEFT JOIN Employee e ON (cd.CREATED_BY_ID = e.LINKED_USER_ID)\n  LEFT JOIN certification_type_trades cts ON (ct.guid = cts.CERTIFICATION_TYPE_GUID)\n  LEFT JOIN certification_type_clients ctc ON (ct.guid = ctc.CERTIFICATION_TYPE_GUID)\n  LEFT JOIN certification_type_areas cta ON (ct.guid = cta.CERTIFICATION_TYPE_GUID)\n  LEFT JOIN certification_type_jobs ctj ON (ct.guid = ctj.CERTIFICATION_TYPE_GUID)\n\nWHERE cd.status = \'CERTIFIED\'\n  AND (cd.is_expired = 0)\n  AND (ct.is_active = 1)\n  AND (ct.is_deleted = 0)\n  AND ct.CUSTOMER_GUID = :customer\n\n  -- Report Parameters\n	AND (ct.GUID like :CERT_TYPE_GUID)\n	AND (-1 = :EMPLOYEE_ID OR e.ID = :EMPLOYEE_ID)\n 	AND (-1 = :TRADE_ID OR (cts.TRADE_ID = :TRADE_ID AND e.JOB_ITEM_ID = cts.TRADE_ID))\n	AND (-1 = :LOCATION_ID OR (cta.LOCATION_ID = :LOCATION_ID\n							AND EXISTS (SELECT * FROM employee_areas ea\n										WHERE ea.`EMPLOYEE_ID` = e.id AND ea.`LOCATION_ID` = cta.`LOCATION_ID`)))\n	AND (-1 = :CLIENT_ID OR (ctc.CLIENT_ID = :CLIENT_ID\n							AND EXISTS (SELECT * FROM EmployeeRecord ecr\n												JOIN JobOrder j ON (ecr.JOB_ORDER_ID = j.id)\n										WHERE ecr.`EMPLOYEE_ID` = e.id AND j.`client_id` = ctc.CLIENT_ID)))\n	AND ((0 = :EXPIRY_IN_DAYS AND (GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`, cd.`certified_date`, cd.`expiration_date`) > now() OR GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`, cd.`certified_date`, cd.`expiration_date`) IS NULL)) \n    OR ( 0 <> :EXPIRY_IN_DAYS AND\n		  GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`, cd.`certified_date`, cd.`expiration_date`) BETWEEN now() AND DATE_ADD(now(), INTERVAL :EXPIRY_IN_DAYS DAY)))\n\n-- remove duplicate results\nGROUP BY cd.guid\n\nORDER BY -`Expires On` DESC, ct.name ASC;', 1, NULL, NULL, NULL, NULL, NULL, 0, NULL, NULL);
```


```
-- Certified certs
SELECT ct.name `Certificate Name`, CONCAT(e.firstName, ' ', e.lastName) `Employee`,
IF(ct.Type IS NULL or LENGTH(ct.Type) < 2, ct.Type, CONCAT(UCASE(LEFT(ct.Type, 1)), LCASE(SUBSTRING(ct.Type, 2)))) `Type`,
DATE_FORMAT(CONVERT_TZ(cd.`reviewed_at`, 'UTC', cu.timezone), '%Y-%m-%d') `Certified On`,
-- Expires On field is based on CertificationData.getEffectiveExpirationDate method
DATE_FORMAT(
  CONVERT_TZ(
    GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`
      , cd.`certified_date`, cd.`expiration_date`),
    'UTC', cu.timezone)
  ,'%Y-%m-%d') `Expires On`,
ct.guid `certTypeGuid`, cd.guid`certDataGuid`, e.`id` `employeeId`

FROM certification_type ct
  LEFT JOIN Customer cu ON (ct.CUSTOMER_GUID = cu.`guid`)
  LEFT JOIN certification_data cd ON (ct.guid = cd.CERTIFICATION_TYPE_GUID)
  LEFT JOIN Employee e ON (cd.CREATED_BY_ID = e.LINKED_USER_ID)
  LEFT JOIN certification_type_trades cts ON (ct.guid = cts.CERTIFICATION_TYPE_GUID)
  LEFT JOIN certification_type_clients ctc ON (ct.guid = ctc.CERTIFICATION_TYPE_GUID)
  LEFT JOIN certification_type_areas cta ON (ct.guid = cta.CERTIFICATION_TYPE_GUID)
  LEFT JOIN certification_type_jobs ctj ON (ct.guid = ctj.CERTIFICATION_TYPE_GUID)

WHERE cd.status = 'CERTIFIED'
  AND (cd.is_expired = 0)
  AND (ct.is_active = 1)
  AND (ct.is_deleted = 0)
  AND ct.CUSTOMER_GUID = :customer

  -- Report Parameters
	AND (ct.GUID like :CERT_TYPE_GUID)
	AND (-1 = :EMPLOYEE_ID OR e.ID = :EMPLOYEE_ID)
 	AND (-1 = :TRADE_ID OR (cts.TRADE_ID = :TRADE_ID AND e.JOB_ITEM_ID = cts.TRADE_ID))
	AND (-1 = :LOCATION_ID OR (cta.LOCATION_ID = :LOCATION_ID
							AND EXISTS (SELECT * FROM employee_areas ea
										WHERE ea.`EMPLOYEE_ID` = e.id AND ea.`LOCATION_ID` = cta.`LOCATION_ID`)))
	AND (-1 = :CLIENT_ID OR (ctc.CLIENT_ID = :CLIENT_ID
							AND EXISTS (SELECT * FROM EmployeeRecord ecr
												JOIN JobOrder j ON (ecr.JOB_ORDER_ID = j.id)
										WHERE ecr.`EMPLOYEE_ID` = e.id AND j.`client_id` = ctc.CLIENT_ID)))
	AND ((0 = :EXPIRY_IN_DAYS AND (GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`, cd.`certified_date`, cd.`expiration_date`) > now() OR GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`, cd.`certified_date`, cd.`expiration_date`) IS NULL)) 
    OR ( 0 <> :EXPIRY_IN_DAYS AND
		  GET_COMPLIANCE_CERTIFICATION_EXPIRATION_DATE(ct.`expiration_type`, ct.`expiration_date`, ct.`expiration_num_of_days_after_certified`, cd.`certified_date`, cd.`expiration_date`) BETWEEN now() AND DATE_ADD(now(), INTERVAL :EXPIRY_IN_DAYS DAY)))

-- remove duplicate results
GROUP BY cd.guid

-- order the rows based on effective expiration date, if the date is null, put it at the bottom, and then order the rows with null expiration date by the cert type name alphabetically
ORDER BY ISNULL(`Expires On`) ASC, `Expires On` ASC, ct.name ASC;
```

### How to Add One Day to a Date?
Given a Date dt you have several possibilities:

**Solution 1: You can use the Calendar class for that:**
```
Date dt = new Date();
Calendar c = Calendar.getInstance(); 
c.setTime(dt); 
c.add(Calendar.DATE, 1);
dt = c.getTime();
```
**Solution 2: You should seriously consider using the Joda-Time library, because of the various shortcomings of the Date class. With Joda-Time you can do the following:**
```
Date dt = new Date();
DateTime dtOrg = new DateTime(dt);
DateTime dtPlusOne = dtOrg.plusDays(1);
```
**Solution 3: With Java 8 you can also use the new JSR 310 API (which is inspired by Joda-Time):**
```
LocalDateTime.from(dt.toInstant()).plusDays(1);
```

### Calendar.getInstance(TimeZone.getTimeZone(“UTC”)) is not returning UTC time
[Resource From](https://stackoverflow.com/questions/21349475/calendar-getinstancetimezone-gettimezoneutc-is-not-returning-utc-time)
The `System.out.println(cal_Two.getTime())` invocation returns a Date from getTime(). It is the Date which is getting converted to a string for println, and that conversion will use the default IST timezone in your case.

You'll need to explicitly use DateFormat.setTimeZone() to print the Date in the desired timezone.
```
TimeZone timeZone = TimeZone.getTimeZone("UTC");
Calendar calendar = Calendar.getInstance(timeZone);
SimpleDateFormat simpleDateFormat = 
       new SimpleDateFormat("EE MMM dd HH:mm:ss zzz yyyy", Locale.US);
simpleDateFormat.setTimeZone(timeZone);

System.out.println("Time zone: " + timeZone.getID());
System.out.println("default time zone: " + TimeZone.getDefault().getID());
System.out.println();

System.out.println("UTC:     " + simpleDateFormat.format(calendar.getTime()));
System.out.println("Default: " + calendar.getTime());
```

### Sorting List
```
List<JobOrder> jobOrders = new ArrayList<>();
jobOrders.addAll(jobBoardModelFacade.getAllOpenJobsAsNonManaged(customer, view.getSelectedDate()));
Collections.sort(jobOrders, Comparator.comparing(JobOrder::getJobNo, String.CASE_INSENSITIVE_ORDER));
```

`jobOrders` is now sorted based on the JobNo and case-insensitive and alphabetically!

### Alternate row colors in javascript table
Here is the pure css version,
```
table tr:nth-child(odd) td{
}
table tr:nth-child(even) td{
}
```
And here is the jQuery solution for the same,
```
$(function(){
   $("table tr:even").addClass("evenClassName");
   $("table tr:odd").addClass("oddClassName");
});
```
Here is the pure JavaScript solution,
```
function altrows(firstcolor,secondcolor)
{
    var tableElements = document.getElementsByTagName("table") ;
    for(var j = 0; j < tableElements.length; j++)
    {
        var table = tableElements[j] ;

        var rows = table.getElementsByTagName("tr") ;
        for(var i = 0; i <= rows.length; i++)
        {
            if(i%2==0){
                rows[i].style.backgroundColor = firstcolor ;
            }
            else{
                rows[i].style.backgroundColor = secondcolor ;
            }
        }
    }
}
```
