# 202567
homework
 编写更健壮的软件：防御性编程实践指南（Python示例）

在软件工程中，防御性编程是减少错误、提高系统健壮性的核心技术。本文将通过真实场景展示如何实施关键防御策略。

 🔍 防御性编程四大核心原则
1. 输入验证 - 假设所有外部输入都是恶意的
2. 异常处理 - 优雅地处理一切可能故障
3. 状态验证 - 防止非法状态流转
4. 日志审计 - 确保可追溯性和诊断能力

 🛡 场景一：输入验证 - 用户API请求处理

python
import re
from pydantic import BaseModel, validator, HttpUrl

class UserRequest(BaseModel):
    username: str
    email: str
    avatar_url: HttpUrl   自动验证URL格式
    
    @validator('username')
    def validate_name(cls, v):
        if not re.match(r'^\w\-{4,20}$', v):
            raise ValueError('用户名只能包含字母、数字和连字符（4-20字符）')
        if v.lower() in 'admin', 'root', 'system':
            raise ValueError('保留用户名禁止注册')
        return v
    
    @validator('email')
    def validate_email(cls, v):
        if "@" not in v or v.count("@") > 1:
            raise ValueError('无效的邮箱格式')
        domain = v.split('@')-1
        if domain.lower() in 'example.com', 'test.com':
            raise ValueError('禁止使用测试域名')
        return v.lower()

 测试用例
try:
    request = UserRequest(
        username="john_doe-2023",
        email="john@realcompany.com",
        avatar_url="https://cdn.com/avatar.png"
    )
    print("✅ 验证通过:", request)
except ValueError as e:
    print(f"🚨 输入错误: {e}")

关键防御点：
- 使用Pydantic模型自动验证数据类型
- 正则表达式约束用户名格式
- 黑名单过滤敏感用户名
- 邮箱格式和域名检查
- URL自动格式验证

💥 场景二：异常处理 - 数据库操作的弹性设计

python
import logging
import sqlite3
from contextlib import contextmanager
from tenacity import retry, wait_exponential, stop_after_attempt

logger = logging.getLogger(__name__)

@contextmanager
def db_connection(db_path):
    conn = None
    try:
        conn = sqlite3.connect(db_path, timeout=5)
         关键：设置自动回滚
        conn.isolation_level = None
        yield conn.cursor()
        conn.commit()
    except sqlite3.Error as e:
        logger.error(f"数据库错误: {e}", exc_info=True)
        if conn:
            conn.rollback()
        raise
    finally:
        if conn:
            conn.close()

@retry(stop=stop_after_attempt(3), 
       wait=wait_exponential(multiplier=1, min=2, max=10))
def safe_query(query, params=None):
    try:
        with db_connection('app.db') as cursor:
             预防SQL注入
            cursor.execute(query, params or ())
            return cursor.fetchall()
    except sqlite3.OperationalError:
        logger.warning("数据库超时，重试中...")
        raise
    except Exception as e:
        logger.critical(f"致命查询错误: {e}")
        return

防御机制详解：
1. 使用`contextmanager`自动管理连接生命周期
2. 设置事务隔离级别，确保失败时自动回滚
3. 通过tenacity实现指数退避重试
4. 参数化查询防止SQL注入
5. 异常分级处理（警告/错误/致命）

---
