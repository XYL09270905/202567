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
