---
layout: post
title: "5 Bước Xác Thực Nodejs Với JSON Web Token (JWT)"
description: "5 Bước Xác Thực Nodejs Với JSON Web Token (JWT)"
comments: true
keywords: "nodejs"
---
Với bài viết này, mình sẽ hướng dẫn các bạn thông qua 5 bước để chúng ta tích hợp xác thực JWT vào project của chúng ta. Ở bài viết này mình không đi sâu vào tìm hiểu lý thuyết, các bạn có thể tìm hiểu lý thuyết ở đây https://viblo.asia/p/tim-hieu-ve-json-web-token-jwt-7rVRqp73v4bP Mình sẽ tập trung vào practice. Let's go! Trước khi thực hiện xác thực JWT phải cài đặt một số package.

### Prerequisite

1. Nodejs
2. Node.js app

### Tools

1. Jsonwebtoken package
2. Bcrypt package
3. Postman

### Steps

#### 1. Cài đặt "jsonwebtoken" package

jsonwebtoken là package của Node phát triển dựa trên `draft-ietf-jose-json-web-signature-08`

#### 2. Tạo user model

Trong folder api/models folder, tạo một file `userModel.js`. Nhưng các bạn đã biết, MongoDB cho phép chúng ta tạo một schema và là nơi chúng ta có thể tạo documents. Chúng ta sẽ sử dụng nó để tạo user trực tiếp trong User document. Trong bài viết này chúng ta sử dụng package `mongoose` của nodejs để tạo schema với các thuộc tính như sau:
1. fullName
2. email address
3. passwork
4. created (ngày tạo)

```
'use strict';

var mongoose = require('mongoose'),
  bcrypt = require('bcrypt'),
  Schema = mongoose.Schema;

/**
 * User Schema
 */
var UserSchema = new Schema({
  fullName: {
    type: String,
    trim: true,
    required: true
  },
  email: {
    type: String,
    unique: true,
    lowercase: true,
    trim: true,
    required: true
  },
  hash_password: {
    type: String,
    required: true
  },
  created: {
    type: Date,
    default: Date.now
  }
});

UserSchema.methods.comparePassword = function(password) {
  return bcrypt.compareSync(password, this.hash_password);
};


mongoose.model('User', UserSchema);
```

#### 3. Tạo phương thức cho user (sign in, register and yêu cầu login)

Trong thư mục api/controllers tạo file `userController.js` Trong file `userController` chúng ta tạo export 3 thương thức khác nhau
'use strict';
```
var mongoose = require('mongoose'),
  jwt = require('jsonwebtoken'),
  bcrypt = require('bcrypt'),
  User = mongoose.model('User');

exports.register = function(req, res) {
};

exports.sign_in = function(req, res) {
};

exports.loginRequired = function(req, res, next) {
};
```
Trong phương thức đăng kí, chúng ta tạo instance user model với User Schema và lưu trong MongoDB
```
exports.register = function(req, res) {
  var newUser = new User(req.body);
  newUser.hash_password = bcrypt.hashSync(req.body.password, 10);
  newUser.save(function(err, user) {
    if (err) {
      return res.status(400).send({
        message: err
      });
    } else {
      user.hash_password = undefined;
      return res.json(user);
    }
  });
};
```
> Lưu ý: Password lưu xuống database phải bcrypt sử dụng Bcrypt package của nodejs

Trong phương thức sign_in chúng ta thực hiện việc hoạt động login. Đầu tiên, chúng ta kiểm tra user đó có tồn tại trong database hay không thông qua email(primary). Nếu có chúng ta kiểm tra password mới phương thức `comparePassword` trong user model user. Nếu thành công response json với tham số email, fullName, id (những tham số này đã được mã hóa khi truyền đến clients). Nếu không match thì response trả lỗi về client
```
exports.sign_in = function(req, res) {
  User.findOne({
    email: req.body.email
  }, function(err, user) {
    if (err) throw err;
    if (!user) {
      res.status(401).json({ message: 'Authentication failed. User not found.' });
    } else if (user) {
      if (!user.comparePassword(req.body.password)) {
        res.status(401).json({ message: 'Authentication failed. Wrong password.' });
      } else {
        return res.json({token: jwt.sign({ email: user.email, fullName: user.fullName, _id: user._id}, 'RESTFULAPIs')});
      }
    }
  });
};

```
Trong phương thúc loginRequired middleware kiểm tra user đã login hay là chưa. Phương thức này sẽ chạy đầu tiên thành công next() sẽ chuyển hướng đến các hoạt động tiếp theo
```
exports.loginRequired = function(req, res, next) {
  if (req.user) {
    next();
  } else {
    return res.status(401).json({ message: 'Unauthorized user!' });
  }
};
```

#### 4. Tạo user routes

Trong folder api/routes tạo file `todoListRoutes.js`. Đây là router cho user register, signup vào sign in để trong cập trên ứng dụng web của chúng ta
'use strict';
```
module.exports = function(app) {
   var userHandlers = require('../controllers/userController.js');

   app.route('/auth/register')
   	.post(userHandlers.register);

   app.route('/auth/sign_in')
   	.post(userHandlers.sign_in);
};
```
#### 5. Updating server.js file

Chúng ta tạo file `server.js`. File này chúng ta tạo instanse User và thiết lập middleware cho express server để check trạng thái của user.
'use strict';
```
var express = require('express'),
  app = express(),
  port = process.env.PORT || 3000,
  mongoose = require('mongoose'),
  Task = require('./api/models/todoListModel'),
  User = require('./api/models/userModel'),
  bodyParser = require('body-parser'),
  jsonwebtoken = require("jsonwebtoken");

mongoose.Promise = global.Promise;
mongoose.connect('mongodb://localhost/Tododb');


app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
```
Sau đó chúng ta tạo middleware cho express, phải chắc chắn rằng middleware này chạy đầu tiên
```
app.use(function(req, res, next) {
  if (req.headers && req.headers.authorization && req.headers.authorization.split(' ')[0] === 'JWT') {
    jsonwebtoken.verify(req.headers.authorization.split(' ')[1], 'RESTFULAPIs', function(err, decode) {
      if (err) req.user = undefined;
      req.user = decode;
      next();
    });
  } else {
    req.user = undefined;
    next();
  }
});
```
Như vậy chúng ta đã tạo server, model, controller đã xong. Bây giờ chúng ta init server và kết hợp với postman để test API chúng ta viết ra
```
npm run start
```
Đây là những thứ tự hình ảnh mà chúng ta test với postman

![Minion](https://viblo.asia/uploads/c159977f-57d1-43c0-a1ab-d4fa2701035f.png)
![Minion](https://viblo.asia/uploads/d0a5a7a0-d5e3-4c4b-ac7e-ba832ff8621b.png)
![Minion](https://viblo.asia/uploads/d6e32492-70cb-4fc5-a4af-f4bcb7d25de3.png)

### Tổng kết

Như vậy chúng ta qua 5 bước đơn giản chúng ta đã thiết lập được server xác thực với JWT. Đây là source code trên github [Github](https://github.com/generalgmt/RESTfulAPITutorial/tree/authentication). Chúc các bạn thành công

Nguồn: Viblo.asia