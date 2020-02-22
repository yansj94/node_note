rest:
1.cs架构：关注点分离，服务端数据存储，提升简单性，客户端用户界面，提升可移植性
2.无状态：所有用户会话信息都保存在客户端，每次请求都必须包含所有信息，不能依赖上下文
3.缓存：所有服务端的响应都要被标记为可缓存或者不可缓存，减少前后端交互，提升性能
4.统一接口：接口设计统一通用，接口与实现解耦，使前后端可以独立开发迭代
5.分层系统：每层只知道相邻的一层，客户端不知道是和代理还是真实服务器通信，安全层，负载均衡，缓存层
6.按需代码：客户端可以下载运行服务器传来的代码js，减少功能，简化客户端

统一接口限制：
1.资源的标识：任何可以命名的事物，比如用户，评论，每个资源可以通过URI被唯一标识
2.通过表述来操作资源：客户端不能直接操作服务器资源，要通过表述  github api users
3.自描述信息：每个消息必须提供足够的信息让接收者理解，媒体类型json/xml，http方法，是否缓存cache-control
4.超媒体作为应用状态引擎：带文字的链接点击跳转到一个新的页面

restful api：
1.符合rest架构风格的api:
    1.1基本的URI
    1.2标准http方法
    1.3传输的数据媒体类型
GET/users #获取user
GitHub API 实例
2.设计规范：
    2.1URL使用名词，尽量用复数
    2.2URI使用嵌套表示关联关系 users/12/repos/5
    2.3使用正确的HTTP方法，GET/POST/PUT/DELETE
    2.4不符合CRUD的情况：POST/action/子资源 查询字符串：？后面的东西

响应设计规范：github users api
1.查询：？since
2.分页
3.字段过滤 ？fields=‘’
4.状态码 2xx 3xx 4xx 5xx
5.错误处理

安全：
1。https
2.鉴权
3.限流

Koa：
基于nodejs的下一代web框架
中间件 洋葱模型
chrome可以通过fetch调试请求，fetch('//github.api.com/users').then(res=>res.json()).then(res=>{console.log(res)})
postman
koa2里路由也是一个中间件

路由解析参数
```js
if(ctx.url.match(/\/users\/\w+/)){
    const userId = ctx.url.match(/\/users\/(\w+)/)[1];
    ctx.bod='用户'+userId;
}

koa-router:
const Router = require('koa-router')
const router = new Router()
router.get('/',ctx=>{
    ctx.body=''
})
router.get('/users/:id',auth,ctx=>{ // 多中间件做鉴权
    const id = ctx.params.id // 参数解析
})
app.use(router.routes())

const usersRouter = new Router({prefix:'/users'}) //前缀

// 多中间件
const auth = async(ctx,next) => {
    if(ctx.url !== 'users'){
        ctx.throw(401)
    }
    await next()
}

// HTTP options
koa-router allowedMethods

app.user(usersRouter.allowMethods()) // 所有接口都支持options请求了

// restful最佳实践，curd应该返回什么响应
delete :ctx.status(204)

// 控制器简介：拿到路由分配的任务并执行，再koa中也是一个中间件
// 获取http请求参数：queryString ？q=keyword
// Router params ： users/：id
// body {key：value} content-type application/json
// header accept jwt/token cookie
// 编写控制器最佳实践：每个资源的控制器放在不同的文件里，尽量使用类+类方法的形式编写控制器，严谨处理错误

// 获取http请求参数
ctx.query
ctx.params
koa-body-parser // 解析request的body
ctx.request.body
ctx.headers
// 发送http响应：status body header 实现curd
ctx.set("Allow","get,post")
// 更合理的目录结构 controls routers
// 写脚本读取routers目录下面的文件名自动添加到index中
const fs = require('fs')
module.exports = app => {
    fs.readdirSync(__dirname).forEach(file=>{
        if(file==='index.js') return;
        const route = require(`./${file}`);
        app.use(route.routes()).use(route.allowMethods())
    })
}

// P6 错误处理 运行时错误 500， 逻辑错误4XX，找不到404.参数格式不对422，
// 为什么要用错误处理：防止程序挂掉，告诉用户错误信息，便于开发者调试
// koa2自带的错误处理 制造404 412 500 三种错误
404客户端错误，资源不存在
412 先决条件失败
if(){ctx.throw(412)} // precondition failed
500运行时错误
a.b // a undefined internal service error

app.use(async(ctx,next)=>{ // 捕获不了404 ，可以捕获手动throw的error以及运行时错误
    try{
        await next()
    }catch(err){
        ctx.status = err.status || err.statusCode || 500
        ctx.body={
            message:err.message
        }
    }
})

// koa-json-error
const error = require('koa-json-error')
app.use(error({
    postFormat: (e,{stack,...rest}) => process.env.NODE_ENV === 'production' ? rest : {stack,...rest}
}))

// koa-parameter 校验参数
const parameter   = require('koa-parameter')
app.use(parameter(app))
ctx => {
    ctx.verifyParams({
        name: {type: 'string',requiere:true}
    })
}

// NoSQL HBase列存储 MongoDB 文档存储json key-value存储 redies。。。
// 优点：简单，没有原子性，一致性，隔离性等复杂规范，便于横向拓展，适合超大规模数据存储，很灵活的存储复杂结构数据 schema free
// 云mongoDB 阿里云，mongoDB atlas： 注册用户，创建集群，添加数据库用户，设置ip白名单，获取链接地址 
//  用mongoose链接mongodb： 
const mongoose = require('mongoose')
mongoose.connect('xxxx',{...options}, ()=>{console.log('success')})
mongoose.connection.on('error',()=>{})

// 设计用户模块的schema：分析属性，编写schema，生成model
// model-》user.js
const {Schema,model} = mongoose
const userSchema = new Schema({
    name: {type: String, required: true, default: 'xx'}
})

module.exports = model('user',userSchema)
// 注意错误处理 ctx.throw()
const User = require('./models/users')
await User.find() // get all
await User.findById(ctx.params.id)
await User.findByIdAndUpdata(ctx.params.id,ctx.request.body)
await new User(ctx.request.body).save()
findByIdAndRemove
 // P8 session
//  session保存在服务端，相对安全，结合cookie使用比较灵活，但是在跨域场景下表现不好，分布式部署需要多机共享session，基于cookie的机制很容易被csrf，查询session可能有数据库操作，sessionstorage关闭被清除，jwt的token存储localstorage
// jwt json web token jwt无缝水平拓展，session横向扩展很复杂，
// 再nodejs中使用jwt 安装jsonwebtoken
const jwt = require('jsonwebtoken')
const token = jwt.sign({name:'ss'},'secret'})
jwt.decode(token)
jwt.verify(token,'secret')
// 实现用户注册 设计用户schema，编写保证唯一性的逻辑
const userSchema = new Schema({
    name: {type: String, required: true, default: 'xx'},
    password: {type:String,requried:true, select:false}
})
// 写中间件鉴权
const auth = async(ctx,next)=>{
    const {authorization=''} = ctx.request.header;
    const token = authorization.replace('Bearer','');
    try{
        const user = jsonwebtoken.verify(token,secret)
        ctx.state.user=user;
    }catch(e){
        ctx.throw(401,e.messages)
    }
    await next()
}
if(ctx.params.id!==ctx.state.user._id){throw(403)}

//koa-jwt
const jwt = require('koa-jwt')
const auth = jwt({secret})

// 上传图片
koa-body 支持文件 json form
app.use(koaBody({
    multipart: true,
    formidable:{
        uploadDir: path.join(__dirname,'public'),
        keepExtension: true
    }
}))
koa-static 生成静态资源
app.use(koaStatic(path.join(__dirname,'public')))
const basename = path.basename(file.path);
ctx.body={url:`${ctx.origin}/uploads/${basename}`}
// 前端上传页面
<form action='/upload' enctyp='multpart/form-data' method='POST'>
<input type='file' name='file' accept='.png' />
<button typ='submit'>submit</button>
</form>

//P10 个人资料需求分析