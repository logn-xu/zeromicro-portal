---
title: limit.TokenLimiter
sidebar_label: 令牌桶
slug: /docs/components/limiter/token
---

`TokenLimiter` 是一个用来控制事件在一秒钟内发生频率的限流器。它依赖于 Redis 来存储令牌桶的状态，并且提供了备用的限流机制，以防 Redis 不可用。

### 新建 TokenLimiter

```go
func NewTokenLimiter(rate, burst int, store *redis.Redis, key string) *TokenLimiter
```

创建并返回一个新的 `TokenLimiter` 实例。

- **参数**:
    - `rate`: 每秒允许的事件数。
    - `burst`: 允许的突发事件数。
    - `store`: Redis 客户端实例。
    - `key`: 用于生成 Redis 存储键的关键字。

- **返回值**: 一个 `TokenLimiter` 实例。

### Allow

```go
func (lim *TokenLimiter) Allow() bool
```

检查是否允许一个事件发生，这是 `AllowN(time.Now(), 1)` 的简写。

- **返回值**: 如果允许事件发生，返回 `true`；否则返回 `false`。

### AllowCtx

```go
func (lim *TokenLimiter) AllowCtx(ctx context.Context) bool
```

带有上下文信息的 `Allow` 方法，是 `AllowNCtx(ctx, time.Now(), 1)` 的简写。

- **参数**:
    - `ctx`: 上下文信息，用于控制请求的生命周期。

- **返回值**: 如果允许事件发生，返回 `true`；否则返回 `false`。

### AllowN

```go
func (lim *TokenLimiter) AllowN(now time.Time, n int) bool
```

检查在指定时间点是否允许 `n` 个事件发生。

- **参数**:
    - `now`: 当前时间。
    - `n`: 请求的事件数。

- **返回值**: 如果允许事件发生，返回 `true`；否则返回 `false`。

### AllowNCtx

```go
func (lim *TokenLimiter) AllowNCtx(ctx context.Context, now time.Time, n int) bool
```

带有上下文信息的 `AllowN` 方法，检查在指定时间点是否允许 `n` 个事件发生。

- **参数**:
    - `ctx`: 上下文信息，用于控制请求的生命周期。
    - `now`: 当前时间。
    - `n`: 请求的事件数。

- **返回值**: 如果允许事件发生，返回 `true`；否则返回 `false`。

## 示例

以下是一些如何使用 `TokenLimiter` 的示例：

### 简单示例

```go
package main

import (
	"fmt"
	"time"

	"github.com/zeromicro/go-zero/core/limit"
	"github.com/zeromicro/go-zero/core/stores/redis"
)

func main() {
	store := redis.NewRedis("localhost:6379", redis.NodeType)
	limiter := limit.NewTokenLimiter(10, 20, store, "example-key")

	if limiter.Allow() {
		fmt.Println("Request allowed")
	} else {
		fmt.Println("Request not allowed")
	}
}
```

在这个示例中，我们创建了一个每秒最多允许 10 个事件、最多允许 20 个突发事件的 `TokenLimiter`。然后我们使用 `Allow` 方法检查是否允许当前请求。

### 使用上下文信息的示例

```go
package main

import (
	"context"
	"fmt"
	"time"

	"github.com/zeromicro/go-zero/core/limit"
	"github.com/zeromicro/go-zero/core/stores/redis"
)

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	store := redis.NewRedis("localhost:6379", redis.NodeType)
	limiter := limit.NewTokenLimiter(5, 10, store, "example-key")

	if limiter.AllowCtx(ctx) {
		fmt.Println("Request allowed with context")
	} else {
		fmt.Println("Request not allowed with context")
	}
}
```

在这个示例中，我们使用上下文信息来控制请求的生命周期。如果在指定的时间内未能获得许可，`AllowCtx` 方法将返回 `false`。

### 检查多个事件的示例

```go
package main

import (
	"fmt"
	"time"

	"github.com/zeromicro/go-zero/core/limit"
	"github.com/zeromicro/go-zero/core/stores/redis"
)

func main() {
	store := redis.NewRedis("localhost:6379", redis.NodeType)
	limiter := limit.NewTokenLimiter(10, 20, store, "example-key")

	now := time.Now()
	requests := 3
	if limiter.AllowN(now, requests) {
		fmt.Printf("%d requests allowed at %v\n", requests, now)
	} else {
		fmt.Printf("%d requests not allowed at %v\n", requests, now)
	}
}
```

在这个示例中，我们检查在当前时间点是否允许 3 个事件发生。如果允许，将输出相应的信息。

通过以上这些示例，你可以更好地理解如何在不同场景下使用 `TokenLimiter` 来进行事件限流。
