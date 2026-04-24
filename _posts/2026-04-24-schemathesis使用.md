
# 安装依赖
```shell
pip install fastapi uvicorn schemathesis
```

# 运行测试api
测试用的api：
```
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="Demo API for Schemathesis")

# 请求体模型
class UserCreate(BaseModel):
    username: str
    age: int
    email: str | None = None

# 接口1：GET 带路径参数 + 查询参数
@app.get("/user/{user_id}", summary="获取用户信息")
def get_user(user_id: int, page: int = 1):
    return {
        "user_id": user_id,
        "page": page,
        "name": "test_user",
        "status": "ok"
    }

# 接口2：POST 带JSON请求体
@app.post("/user/create", summary="创建用户")
def create_user(item: UserCreate):
    return {
        "code": 200,
        "data": item.dict(),
        "msg": "创建成功"
    }
```
运行api：`uvicorn main:app --reload --host 0.0.0.0 --port 80`

# 使用schemathesis
## CLI执行
直接使用Fuzz测试来测试api，参数自动生成：`schemathesis run http://127.0.0.1/openapi.json`。
指定具体的api进行测试：`schemathesis run http://127.0.0.1/openapi.json --include-path="/user/{user_id}" --include-method="GET"`

## pytest用例
编写pytest用例并保存到test.py中，用`pytest test.py`来执行用例。不过这个用例有个问题，这里实际schemathesis会生成多个Fuzz用例，这里修改参数只是强行修改了参数，实际效果就是相同参数的用例会跑多次。
```
import schemathesis
import requests

schema = schemathesis.openapi.from_url("http://127.0.0.1/openapi.json")

@schema.parametrize()
def test_case(case):
    # 只测试你要测试的接口
    if case.path == "/user/{user_id}" and case.method == "GET":
        # 设置参数
        case.path_parameters = {"user_id": 100}
        case.query = {"page": 1}
        response = case.call()
        case.validate_response(response)
```
