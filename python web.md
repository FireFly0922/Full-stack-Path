# 运行
VScode + conda
开始运行：由于 powershell 有一定问题，使用 cmd
```cmd
conda activate fastapi-backend
python -m uvicorn learning:app --reload
```
`--reload`可以自动更新，也就是说更新代码后先`Ctrl + S`保存代码后，网址内容会自动更新
可以直接`Ctrl`并点击连接跳转到查看页面，并使用`http://127.0.0.1:8000/docs#/`在 ui 中查看

# 路由
FastAPI 的路由定义基于 python 的装饰器模式，是 URL 和处理函数间的映射关系
```python
@app.get("/hello")
async def get_hello():
    return {"msg":"你好 FastAPI"}
```
路径不一定需要完整定义，比如`/usr/hello`在没有定义`user`的情况下也可以存在

# 参数
- 路径参数
	- 是 URL 的一部分，指向唯一的资源，通过`get`调用
	- 通过 Path 函数增加类型注解
```python
	@app.get("/author/{name}")
	async def get_name(name:str = Path(default=...,min_length=2,max_length=10)):
	    return {"message":f"这是 {name} 的信息"}
```

- 查询参数
	- 在 URL? 后k1=v1&k2=v2，对资料进行过滤、排序、分页等操作
	- 给路径参数定义形参即可, python 原生注解(: int)或 Query 注解(= Query())
	- Query可以设置默认值，直接在()中最前面写即可
```python
@app.get("/usr/book_list")
async def get_book_list(
    skip:int = Query(0,description="跳过的记录数",lt=100),
    limit:int = 10
	):
	return {"skip":skip,"limit":limit}
```

- 请求体
	- HTTP 协议中：请求行(方法、URL、协议版本)；请求头(Content-Type等)；请求体(实际要发送的数据内容)
	- 在 HTTP 请求的消息体中，用于创建和更新资源，使用方法`POST`、`PUT`等
	- 用 Field 增加限制
```python
# 注册：用户名+密码
class User(BaseModel):
    username:str = Field(default="张三",min_length=2,max_length=10,description="用户名，长度为2到10")
    password:str
@app.post("/register")
async def register(user:User):
    return user
```

- 响应类型
	- 默认支持 JSON 格式，且会自动将路径操作函数返回的 python 对象，转换成 JSON 兼容的格式，并包装成 JSONResponse 格式返回
	- HTML 相应：
		- 在装饰器指定相应类：固定返回类型（HTML、纯文本等）
		- 返回相应对象
		```python
		# 设置相应类为HTMLResponse
		@app.get("/html",response_class=HTMLResponse)
		async def get_html():
		    return "<h1>这是一级标题</h1>"
		```
	- File 相应：
		- 直接返回响应对象，图片/PDF/Excel/音视频等
		```python
		@app.get("/file")
		async def get_file():
		    path = "./背景.jpg"
		    return FileResponse(path)
		```
	- 自定义相应数据格式：
		- response_model 通过 Pydantic 定义并约束输出格式
		- 通过一个“类”定义输出格式，使用时将该类赋值给类
		```python
		class News(BaseModel):
		    id:int
		    title:str
		    content:str
		@app.get("/news/{id}",response_model=News)
		async def get_news(id:int):
		    return {
		        "id":id,
		        "title":f"这是第{id}本书",
		        "content":"这是一本好书"
		    }
		```
- 异常处理
	- 用`fastapi.HTTPException`中断客户端引发的错误
