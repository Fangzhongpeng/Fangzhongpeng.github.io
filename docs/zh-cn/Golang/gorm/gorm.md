## GORM 默认规则

GORM 是一个功能强大的 Go 语言 ORM（对象关系映射）库，用于简化与关系型数据库的交互。
它支持多种数据库（如 MySQL、PostgreSQL、SQLite、SQL Server 等），并提供了丰富的功能，
如模型定义、查询构建、事务管理、钩子（Hooks）等。

GORM 默认规则

GORM 是一个功能强大的 Go 语言 ORM 库，它遵循一些默认规则来简化开发工作。
以下是 GORM 的默认规则及其详细说明。

---

1. **表名规则**

GORM 默认将结构体名称转换为复数形式作为数据库表名。例如：

- 结构体 `User` 对应的表名为 `users`。
- 结构体 `Product` 对应的表名为 `products`。

自定义表名
在 GORM 中，如果你希望自定义表名（而不是使用 GORM 默认的复数形式表名），可以通过在模型结构体上实现 TableName 方法来指定表名。以下是具体的使用方法和示例：
假设你有一个 User 模型，你希望将其映射到数据库中的 user 表（而不是默认的 users 表）：


```
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

type User struct {
    gorm.Model // 内嵌 gorm.Model，包含 ID、CreatedAt、UpdatedAt、DeletedAt 字段
    Name  string
    Email string
}

// 实现 TableName 方法，指定表名为 user
func (User) TableName() string {
    return "user"
}
```

2. **字段名规则**

GORM 默认将结构体字段名转换为蛇形命名（snake_case）作为数据库列名。例如：

- 结构体字段 `UserName` 对应的列名为 `user_name`。
- 结构体字段 `CreatedAt` 对应的列名为 `created_at`。

自定义列名

可以通过 `gorm:"column:xxx"` 标签自定义列名：
```
type User struct {
    Name string `gorm:"column:full_name"` 列名为 full_name
}
```
---

3. **主键规则**

GORM 默认将名为 `ID` 的字段作为主键。如果字段名不是 `ID`，可以通过 `gorm:"primaryKey"` 标签指定主键：
```
type User struct {
    UserID int `gorm:"primaryKey"` 指定 UserID 为主键
}
```
---

4. **时间戳规则**

GORM 默认支持以下时间戳字段：

- `CreatedAt`：记录创建时间，GORM 会在插入数据时自动填充当前时间。
- `UpdatedAt`：记录更新时间，GORM 会在更新数据时自动填充当前时间。
- `DeletedAt`：用于软删除（Soft Delete），GORM 会在删除数据时自动填充当前时间。

示例
```
type User struct {
    gorm.Model 内嵌 gorm.Model，包含 ID、CreatedAt、UpdatedAt、DeletedAt 字段
    Name string
}
```
---

5. **软删除规则**

GORM 默认支持软删除功能。如果模型包含 `DeletedAt` 字段，GORM 会在删除记录时自动填充当前时间，而不是真正删除记录。

示例
```
type User struct {
    gorm.Model
    Name string
}

func softDelete(db *gorm.DB) {
    var user User
    db.Delete(&user) 软删除

    查询时排除已删除的记录
    var users []User
    db.Find(&users)

    查询已删除的记录
    db.Unscoped().Find(&users)
}
```
---

6. **字段类型规则**

GORM 会根据 Go 语言字段类型自动推断数据库字段类型。例如：

- `string` → `varchar(255)`
- `int` → `int`
- `time.Time` → `datetime`

 自定义字段类型

可以通过 `gorm:"type:xxx"` 标签自定义字段类型：
```
type User struct {
    Name string `gorm:"type:varchar(100)"` 指定字段类型为 varchar(100)
}
```
---

7. **默认值规则**

GORM 支持为字段设置默认值。可以通过 `gorm:"default:xxx"` 标签指定默认值：
```
type User struct {
    Age int `gorm:"default:18"` 默认值为 18
}
```
---

8. **索引规则**

GORM 支持为字段创建索引。可以通过 `gorm:"index"` 标签创建普通索引，或通过 `gorm:"uniqueIndex"` 标签创建唯一索引：
```
type User struct {
    Email string `gorm:"uniqueIndex"` 创建唯一索引
    Name  string `gorm:"index"`       创建普通索引
}
```
---

9. **关联关系规则**

GORM 支持定义模型之间的关联关系，如一对一、一对多、多对多等。
```
type Post struct {
    gorm.Model
    Title  string
    UserID uint 外键
}

type User struct {
    gorm.Model
    Name  string
    Posts []Post 一对多关系
}
```
---

10. **钩子（Hooks）规则**

GORM 提供了钩子机制，允许在特定事件（如创建、更新、删除）前后执行自定义逻辑。
```
func (u *User) BeforeCreate(tx *gorm.DB) error {
    u.CreatedAt = time.Now()
    return nil
}

func (u *User) AfterCreate(tx *gorm.DB) error {
    log.Println("User created:", u.ID)
    return nil
}
```
---

11. **事务规则**

GORM 支持事务操作，确保多个操作的原子性。
```
func transaction(db *gorm.DB) {
    tx := db.Begin()
    if err := tx.Create(&User{Name: "Alice"}).Error; err != nil {
        tx.Rollback()
        log.Fatal(err)
    }
    if err := tx.Create(&User{Name: "Bob"}).Error; err != nil {
        tx.Rollback()
        log.Fatal(err)
    }
    tx.Commit()
}
```
---

12. **日志规则**

GORM 默认会输出 SQL 日志。可以通过配置禁用日志或设置日志级别：
```
func disableLogger(db *gorm.DB) {
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Silent), 禁用日志
    })
    if err != nil {
        log.Fatal(err)
    }
}
```
---

13. **自动迁移规则**

GORM 的自动迁移（Auto Migration）功能可以根据 Go 语言中的模型定义自动创建或更新数据库表结构。它极大地简化了数据库表的管理工作，避免了手动编写和维护 SQL 脚本的繁琐过程。以下是自动迁移规则的详细说明。
1. 自动迁移的作用

>创建表：如果表不存在，自动根据模型定义创建表。
>
>更新表：如果表已存在，自动根据模型定义更新表结构（如添加新字段、修改字段类型等）。
>
>同步模型和数据库：确保数据库表结构与 Go 语言模型定义一致。

自动迁移的示例
模型定义
```
type User struct {
    gorm.Model
    Name  string
    Email string `gorm:"uniqueIndex"`
    Posts []Post // 一对多关系
}

type Post struct {
    gorm.Model
    UserID uint
    Title  string
}
```
自动迁移
```
db.AutoMigrate(&User{}, &Post{})
```
自动迁移的注意事项

* 字段删除：自动迁移不会删除表中已存在但模型中未定义的字段。
如果需要删除字段，需要手动修改表结构。
* 字段修改：自动迁移会尝试修改字段类型，但如果修改可能导致数据丢失（如 varchar 改为 int），可能会失败。

* 索引管理：自动迁移会创建新的索引，但不会删除已存在但模型中未定义的索引。

* 数据丢失风险：自动迁移可能会修改表结构，导致数据丢失。在生产环境中使用时应谨慎。

---

14.  **查询规则**

GORM 提供了链式调用方法，用于构建复杂的查询。
```
func queryBuilder(db *gorm.DB) {
    var users []User
    db.Where("age > ?", 20).
        Order("created_at desc").
        Limit(10).
        Offset(0).
        Find(&users)
}
```
---

15. **原生 SQL 规则**

GORM 支持执行原生 SQL 查询。
```
func rawSQL(db *gorm.DB) {
    var result []User
    db.Raw("SELECT * FROM users WHERE age > ?", 20).Scan(&result)
}
```
---

总结

GORM 的默认规则旨在简化开发工作，同时提供了丰富的自定义选项。通过合理使用这些规则

## gorm使用说明


---

1. **安装 GORM**

使用以下命令安装 GORM 和 MySQL 驱动：

```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```

---

2. **初始化 GORM**

在使用 GORM 之前，需要初始化数据库连接。以下是一个初始化 MySQL 连接的示例：
```
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func initDB() (*gorm.DB, error) {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        return nil, err
    }
    return db, nil
}
```
---

3. **定义模型**

GORM 使用结构体（Struct）来定义数据库表模型。每个结构体字段对应数据库表的一列。
```
type User struct {
    gorm.Model        内嵌 gorm.Model，包含 ID、CreatedAt、UpdatedAt、DeletedAt 字段
    Name      string `gorm:"column:name;type:varchar(100);not null"`
    Email     string `gorm:"column:email;type:varchar(100);unique;not null"`
    Age       int    `gorm:"column:age;type:int;default:0"`
}
```
字段标签

- `gorm:"column:xxx"`：指定数据库列名。
- `gorm:"type:xxx"`：指定字段类型（如 `varchar(100)`、`int` 等）。
- `gorm:"not null"`：指定字段不能为空。
- `gorm:"unique"`：指定字段值唯一。
- `gorm:"default:xxx"`：指定字段的默认值。

---

4. **自动迁移**

GORM 支持自动迁移功能，可以根据模型自动创建或更新数据库表。
```
func autoMigrate(db *gorm.DB) {
    if err := db.AutoMigrate(&User{}); err != nil {
        log.Fatal("failed to migrate table:", err)
    }
}
```
---

5. **CRUD 操作**

 创建记录
```
func createUser(db *gorm.DB) {
    user := User{Name: "Alice", Email: "alice@example.com", Age: 25}
    result := db.Create(&user) 插入记录
    if result.Error != nil {
        log.Fatal(result.Error)
    }
}
```
查询记录

查询单条记录
```
func getUser(db *gorm.DB) {
    var user User
    db.First(&user, 1) 查询 ID 为 1 的记录
    db.First(&user, "name = ?", "Alice") 查询 name 为 Alice 的记录
}
```
查询多条记录
```
func getUsers(db *gorm.DB) {
    var users []User
    db.Find(&users) 查询所有记录
    db.Where("age > ?", 20).Find(&users) 查询 age 大于 20 的记录
}
```
 更新记录
```
func updateUser(db *gorm.DB) {
    var user User
    db.First(&user, 1)
    db.Model(&user).Update("Name", "Bob") 更新单个字段
    db.Model(&user).Updates(User{Name: "Bob", Age: 30}) 更新多个字段
}
```
 删除记录
```
func deleteUser(db *gorm.DB) {
    var user User
    db.First(&user, 1)
    db.Delete(&user) 删除记录
    db.Where("age < ?", 20).Delete(&User{}) 删除 age 小于 20 的记录
}
```
---

6. **查询构建器**

GORM 提供了链式调用方法，用于构建复杂的查询。
```
func queryBuilder(db *gorm.DB) {
    var users []User
    db.Where("age > ?", 20).
        Order("created_at desc").
        Limit(10).
        Offset(0).
        Find(&users)
}
```
常用方法

- `Where`：添加查询条件。
- `Order`：指定排序规则。
- `Limit`：限制返回的记录数。
- `Offset`：指定偏移量（用于分页）。
- `Select`：指定查询的字段。
- `Joins`：连接其他表。

---

7. **事务管理**

GORM 支持事务操作，确保多个操作的原子性。
```
func transaction(db *gorm.DB) {
    tx := db.Begin()
    if err := tx.Create(&User{Name: "Alice"}).Error; err != nil {
        tx.Rollback()
        log.Fatal(err)
    }
    if err := tx.Create(&User{Name: "Bob"}).Error; err != nil {
        tx.Rollback()
        log.Fatal(err)
    }
    tx.Commit()
}
```
---

8. **钩子（Hooks）**

GORM 提供了钩子机制，允许在特定事件（如创建、更新、删除）前后执行自定义逻辑。
```
func (u *User) BeforeCreate(tx *gorm.DB) error {
    u.CreatedAt = time.Now()
    return nil
}

func (u *User) AfterCreate(tx *gorm.DB) error {
    log.Println("User created:", u.ID)
    return nil
}
```
---

9. **关联关系**

GORM 支持定义模型之间的关联关系，如一对一、一对多、多对多等。
```
type Post struct {
    gorm.Model
    Title  string
    UserID uint 外键
}

func associations(db *gorm.DB) {
    var user User
    db.Preload("Posts").First(&user, 1)
}
```
---

10. **性能优化**

 禁用日志
```
func disableLogger(db *gorm.DB) {
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Silent),
    })
    if err != nil {
        log.Fatal(err)
    }
}
```
 批量插入
```
func batchInsert(db *gorm.DB) {
    var users = []User{{Name: "Alice"}, {Name: "Bob"}}
    db.Create(&users)
}
```
 使用原生 SQL
```
func rawSQL(db *gorm.DB) {
    var result []User
    db.Raw("SELECT * FROM users WHERE age > ?", 20).Scan(&result)
}
```
---

11. **最佳实践**

1. **使用 `gorm.Model`**：
   - 内嵌 `gorm.Model` 可以自动添加 `ID`、`CreatedAt`、`UpdatedAt`、`DeletedAt` 字段。

2. **明确字段类型**：
   - 使用 `gorm:"type:xxx"` 明确指定字段类型，避免歧义。

3. **处理错误**：
   - 始终检查 GORM 操作的错误，避免程序崩溃或数据不一致。

4. **使用事务**：
   - 对于多个操作，使用事务确保原子性。

5. **避免 N+1 查询**：
   - 使用 `Preload` 或 `Joins` 预加载关联数据，避免多次查询。

---

12. **总结**

GORM 是一个功能强大且易于使用的 ORM 库，适用于大多数 Go 语言项目。通过合理使用 GORM 的功能，
可以显著提高开发效率，并减少手动编写 SQL 的工作量。
```
func main() {
    db, err := initDB()
    if err != nil {
        log.Fatal("failed to connect database:", err)
    }

    autoMigrate(db)
    createUser(db)
    getUser(db)
    getUsers(db)
    updateUser(db)
    deleteUser(db)
    queryBuilder(db)
    transaction(db)
    associations(db)
    disableLogger(db)
    batchInsert(db)
    rawSQL(db)
}
```

