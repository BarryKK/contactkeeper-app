# ContactKeeper

>此项目用于全栈学习, 用户可以创建账号并添加联系人, 数据会被保存到数据库, 用户再次登陆时记录会被保存.

项目预览: https://young-eyrie-46821.herokuapp.com/

此项目使用到的技术：
* Express.js： 后端Node.js框架
* React: 前端框架
* MongoDB：数据库
* Postman: 调试API

## Configuration
Express配置
```javascript
const express = require('express');
const connectDB = require('./config/db');
const path = require('path')

const app = express();

//Connect Database
connectDB();

//init Middleware
app.use(express.json({extended:false}));

//Define Routes
app.use('/api/users', require('./routes/users'));
app.use('/api/auth', require('./routes/auth'));
app.use('/api/contacts', require('./routes/contacts'));

// serve static assets in production
if(process.env.NODE_ENV === 'production') {
     //set static folder
     app.use(express.static('client/build'));

     app.get('*',(req,res) => res.sendFile(path.resolve(__dirname, 'client', 'build', 'index.html')));
}



const PORT = process.env.PORT || 5000;  

app.listen(PORT, () => console.log(`Server started on port ${PORT}`)); 
```
Express-validator:用于验证用户注册
```javascript
router.post('/', 
    [
        check('name','Please add name').not().isEmpty(),
        check('email','Please include a valid email').isEmail(),
        check('password','Please enter a password with 6 or more characters').isLength({min:6})

    ], 
    async (req,res) => {
        const errors = validationResult(req);
        if(!errors.isEmpty()){
            return res.status(400).json({errors: errors.array()});
        }
        
        const {name,email,password} = req.body;

        try {
            let user = await User.findOne({email});

            if(user){
                return res.status(400).json({msg:'User already exists'})
            }

            user = new User({
               name,
               email,
               password
            });

            const salt = await bcrypt.genSalt(10);

            user.password = await bcrypt.hash(password,salt);

            await user.save();
      }
```
Jsonwebtoken:用于加密用户登陆数据
```javascript
const payload= {
                user:{
                    id:user.id
                }
            }
            
            jwt.sign(
                payload, 
                config.get('jwtSecret'),
                {
                 expiresIn:3600
                },
                (err,token) => {
                if(err) throw err;
                res.json({token});
            });
```
### 保护路由
后端：
```javascript
const jwt = require('jsonwebtoken');
const config = require('config');

module.exports = function(req,res,next){
    //Get token from header
    const token = req.header('x-auth-token');

    //Check if not token
    if(!token) {
        return res.status(401).json({ msg:'No token, authorization denied'});
    }

    try {
        const decoded= jwt.verify(token, config.get('jwtSecret'));

        req.user = decoded.user;
        next();
    } catch (err) {
        res.status(401).json({msg:'Token is not valid'})
    }
}
```
前端：
```javascript
import React, {useContext} from 'react'
import {Route, Redirect} from 'react-router-dom'
import AuthContext from '../../context/auth/authContext'

const PrivateRoute = ({component: Component, ...rest}) => {
    const authContext = useContext(AuthContext);
    const {isAuthenticated,loading} = authContext;
    return (
        <Route {...rest } render={props => !isAuthenticated && !loading ?(
            <Redirect to='/login'/>
        ):(
            <Component {...props} />
        )}/>
    );
}
```
