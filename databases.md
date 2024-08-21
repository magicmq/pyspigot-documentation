# 数据库管理器

PySpigot 包含了一个数据库管理器，允许你连接并与 SQL 类型的数据库（包括 MySQL、MariaDB、PostgreSQL 等）以及 MongoDB 进行交互。在底层，PySpigot 使用 [HikariCP](https://github.com/brettwooldridge/HikariCP) 来与 SQL 类型的数据库服务器进行接口，并使用 [MongoDB Java 驱动程序](https://www.mongodb.com/docs/drivers/java-drivers/) 来与 MongoDB 数据库服务器进行接口。

请参见 [一般信息](writingscripts#PySpigot-的管理器) 页面了解如何将数据库管理器导入到你的脚本中的说明。

这不是一个关于使用 SQL 或 MongoDB 的全面指南。如果你不确定如何使用 SQL 或 MongoDB，请查找适当的教程和信息。

# SQL

数据库管理器允许你连接并与 SQL 类型的数据库进行交互。

## SQL 数据库的数据库管理器使用

数据库管理器提供了几个函数供你使用来与 SQL 数据库进行交互。这些函数包括：

- `newHikariConfig()`: 返回一个新的 HikariConfig 对象以方便使用。
- `connectSql(host, port, database, username, password)`: 使用给定的主机、端口、数据库名、用户名和密码连接到远程 SQL 数据库。
- `connectSql(host, port, database, username, password, hikariConfig)`: 使用给定的主机、端口、数据库名、用户名、密码以及提供的 HikariConfig 连接到远程 SQL 数据库。
- `connectSql(uri)`: 使用给定的连接字符串连接到远程 SQL 数据库（有关连接字符串的详细信息，请参见下文）。
- `connectSql(uri, hikariConfig)`: 使用给定的连接字符串以及提供的 HikariConfig 连接到远程 SQL 数据库（有关连接字符串的详细信息，请参见下文）。
- `connectSql(hikariConfig)`: 使用给定的 HikariConfig 连接到远程 SQL 数据库。
- `disconnect(database)`: 断开并关闭对数据库的打开连接。接受 `connectSql` 返回的 `SqlDatabase` 对象。

上述所有的 `connectSql` 函数都实现了相同的目标：初始化与远程数据库的新连接，并返回一个 `SQLDatabase` 对象，该对象可用于选择和更新数据库中的表。你使用哪个 `connectSql` 函数取决于你的具体应用场景。在所有上述函数中，`connectSql(host, port, database, username, password)` 是最基础的，如果你刚开始使用，你应该可能使用这个函数。

数据库管理器还允许你指定一个 [连接字符串/URI/URL](https://learn.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)，如果你想要更精细地控制连接参数和设置。你也可以在连接字符串中指定选项。接受 `uri` 的 `connectSql` 函数是指接受连接字符串的函数。

?> 如果你完成了对数据库连接的使用，遵循良好的实践关闭它是好的。如果你在脚本停止或终止时有任何打开的数据库连接，则这些打开的连接将被自动关闭。如果在执行语句期间或即将执行语句时关闭了数据库，则无法保证语句的执行。

## HikariConfig

HikariConfig 是一个配置对象，允许你对 SQL 连接有更大的控制权。例如，它可以让你设置最小空闲时间、池大小、空闲超时时间等。有关 HikariConfig 的详细信息，请参阅 [HikariCP JavaDoc](https://www.javadoc.io/doc/com.zaxxer/HikariCP/latest/com.zaxxer.hikari/com/zaxxer/hikari/HikariConfig.html)。

你也可以单独使用 HikariConfig 对象（通过 `connectSql(hikariConfig)` 函数）来建立连接。HikariConfig 对象允许你在对象内直接指定主机、端口、数据库、用户名和密码。这可能是与远程数据库建立新连接最简单的方法。

数据库管理器中提供了一个 `newHikariConfig()` 函数以方便使用。它返回一个带有默认选项的新 HikariConfig，你可以对其进行修改。一旦你按照自己的喜好修改了选项，你就可以将你的 HikariConfig 传递给其中一个 `connectSql` 函数。

## SqlDatabase 对象

当我们调用上述任何一个 `connectSql` 函数时，就会建立连接。如果连接成功建立，则会返回一个 `SqlDatabase` 对象。SqlDatabase 对象包含了用于直接与数据库交互的函数：

- `getHikariDataSource()`: 返回底层的 HikariDataSource 连接对象。
- `select(sql)`: 使用提供的 `sql` 在数据库上执行 SELECT 语句。返回一个映射（基本上等同于 Python 字典），其中键代表列名，值代表列数据（对象列表）。
- `select(sql, values)`: 使用提供的 `sql` 和 `values`（对象列表）在数据库上执行 SELECT 语句。返回一个映射（基本上等同于 Python 字典），其中键代表列名，值代表列数据（对象列表）。
- `update(sql)`: 使用提供的 `sql` 在数据库上执行 UPDATE 语句。返回一个整数，指示受影响的行数。
- `update(sql, values)`: 使用提供的 `sql` 和 `values`（对象列表）在数据库上执行 UPDATE 语句。返回一个整数，指示受影响的行数。

上述函数中的 `sql` 仅仅是一个 SQL 语句。例如：

```sql
SELECT * FROM test_table;
```

我们还可以定义将在 SQL 语句中自动插入的值。例如，我们可以定义一个这样的 SQL 语句：

```sql
SELECT * FROM test_table WHERE id = ?;
```

这个语句被称为**参数化查询**，因为它包含了问号 (`?`)。SQL 语句中的问号充当*占位符*，即稍后将插入的值。这就是 `values` 参数发挥作用的地方：问号会被 `values` 中传递的值替换。例如，如果我们调用

```python
sql.select('SELECT * FROM test_table WHERE id = ?', [10])
```

那么实际发送到服务器的 SQL 语句将是

```python
sql.select('SELECT * FROM test_table WHERE id = ?', [10])
```

因为问号被我们在 `values` 中传递的值 `10` 替换。如果你想要在 `values` 中传递多个值，这也适用：

```python
sql.update('INSERT INTO test_table (id, val) VALUES (?, ?)', [11, 1])
```

上述语句实际上变成了 `INSERT INTO test_table (id, val) VALUES (11, 1);`。请注意 `values` 中对象的顺序很重要，因为它决定了 SQL 语句中占位符被替换的顺序。第一个问号被位置零的对象替换，第二个问号被位置一的对象替换，依此类推。

## SQL 数据库代码示例

以下代码连接到远程 SQL 数据库并执行一些简单操作。交互的表名为 `test_table`，具有列 `id`（自动递增，不可为空，唯一）和 `value`（不可为空）：

```python
import pyspigot as ps

db_manager = ps.database_manager()

sql = db_manager.connectSql('localhost', '3306', 'test', 'root', '')

data = sql.select('SELECT * FROM test_table ORDER BY val DESC;')

for column, col_data in data.items():
    print(column)
    print(col_data)

rows_affected = sql.update('INSERT INTO test_table (id, val) VALUES (?, ?)', [11, 1])
print('Rows affected: ' + str(rows_affected))

db_manager.disconnect(sql)
```

第 1 行，我们导入 PySpigot 并将其别名为 `ps` 以利用数据库管理器。

第 3 行，我们从 `ps` 获取数据库管理器，并将其设置为 `db_manager`。

第 5 行，我们使用 `connectSql` 建立新的连接，并将返回的对象赋值给 `sql`。此变量是我们随后用于与数据库交互的对象。

第 7 行，我们从 `test_table` 中选择所有数据，并按 `val` 列降序排列，将所选数据分配给变量 `data`。

第 9 至 11 行，我们遍历所有数据并打印列名以及对应于该列的数据。

第 13 行，我们对 `test_table` 执行更新操作，插入新行，其中 `id` 的值为 11，`val` 的值为 1，并将结果（受影响的行数）分配给变量 `rows_affected`。

第 14 行，我们打印受影响的行数。

第 16 行，我们通过调用数据库管理器的 `disconnect` 函数并传递我们之前连接时获取的数据库对象来关闭数据库连接。

!> 需要注意的是，与任何远程数据库的交互都是 **I/O 操作**。如果我们在此主服务器线程上运行此代码（即不在异步任务中运行），则服务器将会挂起且无响应，直到与远程服务器的交互完成。对于延迟高或高延迟的连接，这可能会导致大量的服务器延迟。为了避免延迟，最佳做法是将所有 **I/O 操作** 包装在异步任务中。请参见下面的部分了解示例。

## 异步 SQL 数据库代码示例

以下代码采用上述代码示例，但将与数据库的交互包装在一个异步任务中。值得注意的是，与数据库的连接（第 6 行的 `connectSql` 函数）仍然是同步的，但我们希望它保持同步，因为连接失败会影响脚本其余部分的执行，如果连接失败（即抛出错误），脚本应该终止。

下面的代码利用了任务管理器的回调任务，该任务异步运行一个任务，然后当异步任务完成后通过另一个用户定义的函数回调到主服务器线程。

```python
import pyspigot as ps

task_manager = ps.task_manager()
db_manager = ps.database_manager()

sql = db_manager.connectSql('localhost', '3306', 'test', 'root', '')

def select_data():
    data = sql.select('SELECT * FROM test_table ORDER BY val DESC;')
    return data

def sync_callback(data):
    for column, col_data in data.items():
        print(column)
        print(col_data)

task_manager.runSyncCallbackTask(select_data, sync_callback)

def update():
	rows_affected = sql.update('INSERT INTO test_table (id, val) VALUES (?, ?)', [11, 1])
	print('Rows affected: ' + str(rows_affected))

task_manager.runTaskAsync(update)
```

# MongoDB

数据库管理器允许您连接并交互 MongoDB 数据库（MongoDB）。

## MongoDB 数据库管理器使用方法

数据库管理器提供了多个函数供您用于与 MongoDB 进行交互。这些函数包括：

- `newMongoClientSettings()`: 返回一个新的 MongoClientSettings 构造器以方便使用。
- `connectMongo(host, port, username, password)`: 使用指定的主机、端口、用户名和密码连接到远程 MongoDB 服务器。
- `connectMongo(host, port, username, password, clientSettings)`: 使用指定的主机、端口、用户名和密码以及提供的 MongoClientSettings 连接到远程 MongoDB 服务器。
- `connectMongo(uri)`: 使用指定的连接字符串连接到远程 MongoDB 服务器。
- `connectMongo(uri, clientSettings)`: 使用指定的连接字符串和提供的 MongoClientSettings 连接到远程 MongoDB 服务器。
- `connectMongo(clientSettings)`: 使用提供的 MongoClientSettings 连接到远程 MongoDB 服务器。
- `disconnect(database)`: 断开并关闭与数据库的打开连接。接受 `connectMongo` 返回的 MongoDatabase 对象。

如果您想要更精细地控制连接参数和设置，数据库管理器允许您指定[连接字符串/URI/URL](https://learn.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)。您也可以在连接字符串中指定选项。接受 `uri` 的 `connectMongo` 函数是可以接受连接字符串的函数。

这些函数与初始化 SQL 数据库连接的函数非常相似。唯一的真正区别在于，我们不是使用 HikariConfig，而是可以使用 MongoClientSettings 来指定连接设置。

?> 如果您完成了对数据库连接的使用，遵循良好实践关闭它是明智的。如果您的脚本停止或终止时有未关闭的数据库连接，则这些打开的连接将自动关闭。如果在操作执行期间或即将执行时关闭数据库，则无法保证操作会执行。

## MongoClientSettings

MongoClientSettings 是一个配置对象，允许对 MongoDB 连接进行更多控制。例如，它可以设置连接池设置、连接字符串、加密设置、读取偏好、写入偏好等。有关 MongoClientSettings 的详细信息，请参阅[MongoDB 文档](https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/connection/mongoclientsettings/)。

您还可以仅使用 MongoClientSettings 对象来建立连接（通过 `connectMongo(clientSettings)` 函数）。MongoClientSettings 构造器对象允许您直接在对象中指定连接字符串或主机、端口、数据库、用户名和密码。这可能是与远程数据库建立新连接最简单的方法。

数据库管理器提供了一个 `newMongoClientSettings()` 函数以供方便使用。它返回一个带有默认选项的新 MongoClientSettings 构造器，您可以对其进行修改。一旦您按照自己的喜好修改了选项，就可以调用 `build()` 方法并将结果传递给其中一个 `connectMongo` 函数。

!> 在您完成 MongoClientSettings 的设置后，必须调用 `build()` 以构建设置为可读对象，然后再将其传递给其中一个 `connectMongo` 函数。


## MongoDatabase 对象

当我们调用上述任何一个 `connectMongo` 函数时，就会建立连接。如果连接成功建立，那么会返回一个 `MongoDatabase` 对象。`MongoDatabase` 对象包含了直接与数据库交互使用的函数。

`MongoDatabase` 对象中的函数太多，无法在这里一一列出。要查看完整列表，请参阅[PySpigot JavaDocs](https://javadocs.magicmq.dev/pyspigot/dev/magicmq/pyspigot/manager/database/mongo/MongoDatabase.html)。

此外，[MongoDB 文档](https://www.mongodb.com/docs/drivers/java/sync/current/usage-examples/)包含了详细的文档和示例代码。我强烈建议您查阅这些文档来了解如何使用 `MongoDatabase` 对象，因为我定义的许多 `MongoDatabase` 函数仅仅是底层 MongoDB 库的传递接口。

?> 如果您完成了对数据库连接的使用，遵循良好实践关闭它是明智的。如果您的脚本停止或终止时有未关闭的数据库连接，则这些打开的连接将自动关闭。如果在操作执行期间或即将执行时关闭数据库，则无法保证操作会执行。

## MongoDB 代码示例

以下代码建立了与远程 MongoDB 服务器的连接，创建了一个集合，插入了一个文档，然后检索了创建的文档。

```python
import pyspigot as ps

db_manager = ps.database_manager()

mongo = db_manager.connectMongo('localhost', '27017', None, None)

if (not mongo.doesCollectionExist('test', 'test_collection')):
    mongo.createCollection('test', 'test_collection')

mongo.insertDocument('test', 'test_collection', mongo.createDocument('test_key', 'test_value'))

documents = mongo.getDocuments('test', 'test_collection')

for document in documents:
    print(document)
```

!> 上述代码是同步运行的。如上所述，最好在异步上下文中与外部数据库服务器交互，以避免服务器延迟。

## 总结：{docsify-ignore}

- 数据库管理器允许您连接并交互 SQL 类型和 MongoDB 数据库。
- 使用 `connectSql` 函数（根据您的具体情况提供相应的参数）来连接到 SQL 数据库。
- 使用 `connectMongo` 函数（根据您的具体情况提供相应的参数）来连接到 MongoDB 数据库。
- 当连接到数据库时，连接函数会返回一个数据库对象（要么是 `SqlDatabase` 要么是 `MongoDatabase`），然后使用该对象与数据库交互。
- 与数据库的交互是一个 *I/O 操作*，这些交互应在异步上下文中进行，例如在计划的异步任务中或在回调任务内。
- 当脚本停止时，数据库连接会自动关闭。在其他任何时候，如果您完成了对数据库的使用，应通过调用数据库管理器的 `disconnect` 函数来关闭它。`disconnect` 函数接受在连接到数据库时获取的数据库对象。
