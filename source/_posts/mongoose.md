---
title: Mongoose入门总结
date: 2017-11-15 18:57:38
categories:
- JavaScript
- Mongoose
tags: 
- mongoose
- mongodb
---
Mongoose 是一个基于Node.js的MongoDB接口ORM类库，也被称为ODM库。那些高大上的解释就不贴了。我的理解是mongoDB将数据库抽象成文档类型，而mongoose将文档映射到js对象上，让使用者可以使用js对象来操作文档从而实现对数据的增改删查，因此称之为Object Document Model。
<!--more-->
## 连接数据库-Connection
使用数据库之前肯定需要连接上数据库先，Mongoose的API提供了两种方式`mongoose.createConnection`和`mongoose.connect`。文档说前者可以连接多个数据库，后者使用默认连接，老实说这个解释没能让我明白区别，还是看源代码吧。
```js
function Mongoose() {
  this.connections = [];
  this.models = {};
  this.modelSchemas = {};
  /* default global options */
  this.options = {
    pluralization: true
  };
  var conn = this.createConnection(); /* default connection */
  conn.models = this.models;
}
var mongoose = module.exports = exports = new Mongoose;
```
可以看出来，我们引用mongoose时，就是引用Mongoose的一个实例。
构造函数格外的简洁，初始化四个属性，建立一个连接，再将该连接的models属性指向实例的同名属性。
下面再来看建立的这个连接
```js
var driver = global.MONGOOSE_DRIVER_PATH || './drivers/node-mongodb-native';
var Connection = require(driver + '/connection');

Mongoose.prototype.createConnection = function(uri, options) {
  var conn = new Connection(this);
  this.connections.push(conn);
  if (options && options.useMongoClient) {
    return conn.openUri(uri, options);
  }
  return conn;
};

Mongoose.prototype.__defineGetter__('connection', function() {
  return this.connections[0];
});

Mongoose.prototype.__defineSetter__('connection', function(v) {
  this.connections[0] = v;
});
```
Mongoose初始化实例即建立了和MongoDB的连接，也就是注释了说的默认连接，可以通过`mongoose.connection`访问，该连接可后续再行初始化。

到这里就可以进入正题说`mongoose.createConnection`和`mongoose.connect`的区别了。贴下connect的代码
```js
Mongoose.prototype.connect = function() {
  var conn = this.connection;
  if ((arguments.length === 2 || arguments.length === 3) &&
      typeof arguments[0] === 'string' &&
      typeof arguments[1] === 'object' &&
      arguments[1].useMongoClient === true) {
    return conn.openUri(arguments[0], arguments[1], arguments[2]);
  }
  if (rgxReplSet.test(arguments[0]) || checkReplicaSetInUri(arguments[0])) {
    return new MongooseThenable(this, conn.openSet.apply(conn, arguments));
  }

  return new MongooseThenable(this, conn.open.apply(conn, arguments));
};
```
connect就是直接使用mongoose初始化时建立的那个连接去连接数据库，具体怎么连接数据库，根据参数选择，可以有`openUri`、`openSet`和`open`, 4.x版本推荐第一种方式，后两种将被废弃。即需形如`mongoose.connect(db_url, {useMongoClient:true})`这种方式。
而`createConnection`则是mongoose初始化选择的连接方式，可以建立多个连接，每个连接都会被push到`mongoose.connections`数组中，同时返回此次建立的连接，如果指定了参数，且符合要求，则调用`openUri(uri, options)`连接数据库。
至此，基本捋清二者关系了。

小结下就是，mongoose初始化时会先建立一个与MongoDB驱动的默认连接，connect的方式就是使用该连接。createConnection 会再继续建立与驱动的连接，这种方式适合操作多个数据库的操作。使用openUri方法，是给驱动传参连接数据库。

下面举例以上的说明
```js
var mongoose = require('mongoose')
mongoose.Promise = global.Promise
var url = process.env.MONGO_URL || 'mongodb://localhost:27017/admin'

/* 使用默认连接传参连接数据库 */
mongoose.connection.openUri(url, (err, conn)=>{
    console.log(conn.db.databaseName)
})

/* 使用默认连接传参连接数据库 */
var db = mongoose.connect(url, {useMongoClient:true})
console.log(mongoose.connections.length)

/* 重新建立连接并传参连接数据库 */
var db1 = mongoose.createConnection(url, {useMongoClient:true})
console.log(mongoose.connections.length)
```

## 定义文档结构-Schema
mongoDB的collection对应关系型数据库的table，collection是文档的集合，mongoose里的`Schema`就是用来定义文档的数据结构。
支持的数据类型称为`SchemaTypes`，分别是
- String
- Number
- Date
- Boolean
- Array
- Buffer
- ObjectId
- Mixed

使用举例：
```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var blogSchema = new Schema({
  title:  {type: String, index: true},
  author: String,
  body:   String,
  comments: [{ body: String, date: Date }],
  date: { type: Date, default: Date.now },
  hidden: Boolean,
  meta: {
    votes: Number,
    fans:  Number
  }
});
```
除了定义文档结构，Schema还能定义文档实例方法，模型的静态方法，混合索引，虚拟属性，以及一些和文档生命周期有关的钩子。此处先不展开。

## 数据库模型-Model
定义Schema后，需要将它编译成Model。
```js
var schema = new mongoose.Schema({ name: 'string', size: 'string' });
var Tank = mongoose.model('Tank', schema);
```
Model是文档的构造函数，Model的实例即文档。
实例化方法有两种：
 - new 之后生成的实例使用save方法保存
 - 使用类方法create创建

```js
var Tank = mongoose.model(modelName, yourSchema);

var small = new Tank({ size: 'small' });
small.save(function (err) {
  if (err) return handleError(err);
  // saved!
})

// or

Tank.create({ size: 'small' }, function (err, small) {
  if (err) return handleError(err);
  // saved!
})
```
需要特别说明的是：
- model的操作不会生效，直到与数据库的连接完成。这里也是异步没有出错的原因。
- 如果使用默认连接需使用`mongoose.model()`，如果是自定义连接使用` connection's model()`，这是个挂载位置问题。


具体的model实例对应一个`collection`,collection是文档的集合，每一个具体的文档对应一张数据表，可以映射为一个javascript对象。其链式关系大体为
- db -> collections
- collection -> docs
- doc -> js Object (mongoose.Document instance)

## 数据库操作-CRUD
每个数据库都会有增改删查的操作，对应mongodb而言，操作的主要分两类
- collection 层的批量操作
- doc 层的具体操作
如果是第一类操作，所有方法都是基于model的，例如：
```js
model.create() //给该collection创建一个或多个新的doc
model.remove() //删除一个或多个符合条件的doc
model.find()   //查找，返回一个docs数组
model.update() //更新一个或多个doc
model.count() //统计该collection下docs的数量
```
如果是基于单个doc的操作，一些常用的方法有
```js
doc.remove() //删除该文档
doc.save() //更改后的保存
doc.update() //更新
doc.toObject() //将文档转为纯javascript对象，转化后就不再有save等方法
doc.id  //获取文档元素
```
不同的操作层级有类似的操作方法，但是作用的对象不同，逻辑上需辨别清楚。
## 总结
- 一切的数据库操作都是建立在与数据库的连接上，所有文章开头详细描述了mongoose对于与mongodb连接的管理
- schema和model的意义及使用
- 数据库操作可以理解为是先获取对应的对象，随后调用对应的操作方法


