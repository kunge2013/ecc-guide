# 场景 3: Go 编译错误

**语言：** Go
**工具：** go build / go vet
**优先级：** HIGH (阻止编译)

## 错误场景

编译错误、类型不匹配、未使用的变量等问题。

### 问题代码

```go
// file: user/user_service.go

package user

import (
	"errors"
	"fmt"
)

type User struct {
	ID    int
	Name  string
	Email string
}

type UserService struct {
	db *Database
}

func NewUserService(db *Database) *UserService {
	return &UserService{
		db: db,
	}
}

func (s *UserService) CreateUser(username, email, password string) (*User, error) {
	// ❌ 未使用的变量
	userID := len(s.db.Users) + 1

	user := User{
		ID:    userID,
		Name:  username,
		Email: email,
	}

	// ❌ 错误：s.db.Users 应该是指针
	s.db.Users = append(s.db.Users, user)

	return &user, nil
}

func (s *UserService) GetUserByID(userID int) (*User, error) {
	for _, user := range s.db.Users {
		if user.ID == userID {
			return &user, nil
		}
	}

	// ❌ 错误：返回值类型不匹配
	return nil, "User not found"
}

func (s *UserService) UpdateUser(userID int, updates map[string]interface{}) error {
	user, err := s.GetUserByID(userID)
	if err != nil {
		return err
	}

	// ❌ 错误：类型断言未检查
	if name, ok := updates["name"].(string); ok {
		user.Name = name
	}

	if email, ok := updates["email"]; ok { // ❌ 未使用 ok
		// ❌ 错误：email 是 interface{}，不是 string
		user.Email = email
	}

	// ❌ 错误：返回值类型不匹配
	return nil
}

func (s *UserService) DeleteUser(userID int) {
	for i, user := range s.db.Users {
		if user.ID == userID {
			// ❌ 错误：删除后没有返回
			s.db.Users = append(s.db.Users[:i], s.db.Users[i+1:]...)
		}
	}
}

func (s *UserService) ValidateEmail(email string) bool {
	// ❌ 错误：未导入 regex 包
	matched, _ := regexp.MatchString(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`, email)
	return matched
}

func (s *UserService) FormatUser(user *User) string {
	// ❌ 错误：fmt.Sprintf 参数数量不匹配
	return fmt.Sprintf("User: %s %s", user.Name)
}
```

### 编译错误

```bash
$ go build ./user

# user/user_service.go:23:20: cannot use "User not found" (type un)typed string) as type error in return argument

# user/user_service.go:42:20: cannot use email (type interface{}) as type string in assignment

# user/user_service.go:46: cannot use nil as type *error in return argument

# user/user_service.go:58:9: undefined: regexp

# user/user_service.go:65:22: fmt.Sprintf format %s reads arg #2, but call has 1 arg

# user/user_service.go:19:6: userID declared and not used

# go vet 警告:
# user/user_service.go:42:22: possible misuse of "email" (ok value is not used)
```

### 修复方案

```go
// file: user/user_service.go (已修复)

package user

import (
	"errors"
	"fmt"
	"regexp"
)

type User struct {
	ID    int
	Name  string
	Email string
}

type Database struct {
	Users []User
}

type UserService struct {
	db *Database
}

func NewUserService(db *Database) *UserService {
	return &UserService{
		db: db,
	}
}

func (s *UserService) CreateUser(username, email, password string) (*User, error) {
	// ✅ 使用变量
	userID := len(s.db.Users) + 1

	user := User{
		ID:    userID,
		Name:  username,
		Email: email,
	}

	// ✅ 修正类型错误
	s.db.Users = append(s.db.Users, user)

	return &user, nil
}

func (s *UserService) GetUserByID(userID int) (*User, error) {
	for _, user := range s.db.Users {
		if user.ID == userID {
			return &user, nil
		}
	}

	// ✅ 返回正确的错误类型
	return nil, errors.New("User not found")
}

func (s *UserService) UpdateUser(userID int, updates map[string]interface{}) error {
	user, err := s.GetUserByID(userID)
	if err != nil {
		return err
	}

	// ✅ 正确的类型断言
	if name, ok := updates["name"].(string); ok {
		user.Name = name
	}

	// ✅ 使用类型断言和检查
	if email, ok := updates["email"].(string); ok {
		user.Email = email
	}

	return nil
}

func (s *UserService) DeleteUser(userID int) error {
	for i, user := range s.db.Users {
		if user.ID == userID {
			// ✅ 删除用户并返回
			s.db.Users = append(s.db.Users[:i], s.db.Users[i+1:]...)
			return nil
		}
	}

	return errors.New("User not found")
}

func (s *UserService) ValidateEmail(email string) bool {
	// ✅ 导入 regex 包
	emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
	return emailRegex.MatchString(email)
}

func (s *UserService) FormatUser(user *User) string {
	// ✅ 修正格式化参数
	return fmt.Sprintf("User: %s (%s)", user.Name, user.Email)
}
```

### 验证修复

```bash
$ go build ./user
# ✅ 成功编译

$ go vet ./user
# ✅ 无警告
```

## 关键要点

### 常见编译错误

| 错误类型 | 原因 | 修复方法 |
|---------|------|----------|
| cannot use X as type Y | 类型不匹配 | 修正类型或使用类型转换 |
| undefined: X | 未导入或未定义 | 导入包或定义变量 |
| format %s reads arg | 格式化参数不匹配 | 修正格式化字符串或参数 |
| declared and not used | 未使用的变量 | 使用变量或使用 `_` 忽略 |
| missing return | 缺少返回值 | 添加返回语句 |

### Go 编译最佳实践

1. **错误处理**
   ```go
   // ✅ 正确的错误处理
   user, err := s.GetUserByID(id)
   if err != nil {
       return nil, err
   }

   // ✅ 使用 errors.New 或 fmt.Errorf
   return nil, errors.New("user not found")
   return nil, fmt.Errorf("user %d not found", id)
   ```

2. **类型断言**
   ```go
   // ✅ 安全的类型断言
   if value, ok := data["key"].(string); ok {
       // 使用 value
   } else {
       // 处理类型不匹配
   }
   ```

3. **未使用变量**
   ```go
   // ✅ 使用 _ 忽略未使用的变量
   for _, user := range users {
       // _ 是索引，不需要
       processUser(user)
   }
   ```

4. **指针和值**
   ```go
   // ✅ 明确使用指针或值
   func (s *UserService) Method() {
       // 指针接收者
   }

   func (u User) Method() {
       // 值接收者
   }
   ```

### go vet 警告处理

```go
// ❌ 可能的结构体标签错误
type User struct {
    Name string `json:"name"`  // ✅ 正确
    Age  int    `json:"age`    // ❌ 缺少引号
}

// ❌ 可能的拷贝锁
func (s *Service) Copy() Service {
    return *s  // ❌ 复制了包含锁的结构体
}

// ✅ 使用指针避免拷贝锁
func (s *Service) Copy() *Service {
    return &Service{...}  // 创建新的实例
}
```

### 编译检查脚本

```bash
# file: scripts/build.sh

#!/bin/bash

set -e

echo "Running go fmt..."
gofmt -s -w .

echo "Running go vet..."
go vet ./...

echo "Running go build..."
go build -o bin/app ./...

echo "Build successful!"
```

---

**场景价值：** Go 是静态类型语言，编译错误会阻止构建。及时修复这些错误是 CI/CD 流程的关键。
