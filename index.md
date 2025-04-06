-- users用户表(主键索引,唯一索引,联合索引,全文索引)
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    gender ENUM('male','female') DEFAULT 'male',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,  # 默认自动使用当前时间
    INDEX idx_username_email (username, email),
    FULLTEXT INDEX idx_ft_username (username)
) ENGINE=InnoDB;

-- products 商品表(主键索引,全文索引)
CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    price DECIMAL(10,2),
    category VARCHAR(50),
    tags TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FULLTEXT INDEX idx_ft_tags (tags)
) ENGINE=InnoDB;


-- orders 订单表(主键索引,联合索引,外键字段建立索引)
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    amount INT DEFAULT 1,
    total_price DECIMAL(10,2),
    status ENUM('pending', 'paid', 'shipped', 'cancelled'),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_product (user_id, product_id),
    INDEX idx_status_created (status, created_at),
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
) ENGINE=InnoDB;


-- 用户数据
INSERT INTO users (username, email, gender) VALUES
('alice', 'alice@example.com', 'female'),
('bob', 'bob@example.com', 'male'),
('tom', 'tom@example.com', 'male'),
('jack', 'jack@example.com', 'male'),
('kelly', 'kelly@example.com', 'female');

-- 商品数据
INSERT INTO products (name, price, category, tags) VALUES
('iPhone 14', 799.00, 'Electronics', '手机 苹果 智能设备'),
('iPhone 16 Pro', 899.00, 'Electronics', '手机 苹果 智能设备'),
('Apple Watch', 499.00, 'Electronics', '手表 苹果 智能设备'),
('Mackbook Pro', 1099.00, 'Electronics', '电脑 苹果 智能设备'),
('ViVo X100', 699.00, 'Electronics', '手机 vivo 智能设备'),
('AirPods Pro', 199.00, 'Electronics', '耳机 苹果 无线 音乐');

-- 订单数据
INSERT INTO orders (user_id, product_id, amount, total_price, status) VALUES
(1, 1, 1, 799.00, 'paid'),
(4, 2, 2, 1798.00, 'pending'),
(3, 5, 4, 2796.00, 'paid')
(2, 6, 1, 199.00, 'shipped');


- **场景1**
  - 根据用户 ID 获取用户详情（例如用户中心页面）
  - 涉及索引: 主键索引

```sql
select * from users where id = 3;
```

- **场景2**

  - 登录验证时通过邮箱查找用户

  - 涉及索引: 唯一索引

```sql
select * from users where email = "tom@example.com";
```

- **场景3**
  - 联合索引使用

```sql
-- 最左匹配
select * from users where email = "kelly@example.com";

-- 全值匹配
select * from users where username = "bob" and email = "bob@example.com";

-- 查找某个用户是否购买过某个商品
select * from orders where user_id = 2 and product_id = 5;

-- 查询最近 7 天内所有 "paid" 状态订单（按时间排序或分页）
SELECT * FROM orders
WHERE status = 'paid' AND created_at >= NOW() - INTERVAL 7 DAY
ORDER BY created_at DESC
LIMIT 20;
```

- **场景4**
  - 全文索引使用

```sql
-- 用户搜索商品（搜索“手机 苹果”）, 使用全文索引 idx_ft_tags，用于文本搜索，效率远高于 LIKE '%苹果%
SELECT * FROM products
WHERE MATCH(tags) AGAINST('苹果 手机' IN NATURAL LANGUAGE MODE);

-- 后台用户管理系统支持模糊搜索用户名
SELECT * FROM users
WHERE MATCH(username) AGAINST('jack' IN BOOLEAN MODE);

-- 索引失效场景, 普通索引对 %前缀% 无效
SELECT * FROM products WHERE tags LIKE '%手机%';
```

- **场景5**
  - 多表查询上的索引使用
  - 涉及索引: 外键字段上的索引（辅助索引）

```sql
-- 查询用户的订单详情
# orders.user_id、orders.product_id 均走索引
# products.id 是主键，也走索引，JOIN 性能高
SELECT o.*, p.name, p.price
FROM orders o
JOIN products p ON o.product_id = p.id
WHERE o.user_id = 3;
```

- **场景6**
  - 查询条件中的范围、排序场景（优化 B+Tree 用法)
  - 涉及索引: 范围查询（`BETWEEN` + 联合索引）

```sql
SELECT * FROM orders
WHERE status = 'paid'
  AND created_at BETWEEN '2025-04-01' AND '2025-04-06';
```


