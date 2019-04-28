# fucking-util-signature-all

兼容支持 uni-app/普通网页/微信小程序, 目前只测试了web和微信小程序, 其它平台有问题可以提issue

模块依赖 **fucking-util** 模块, 请预先安装

模块包含 uni-app **预编译代码**

uni-app 开发者直接使用此模块

## 模块选择
1. 要使用 `uni-app` 模式开发, 使用此模块
2. 不使用 `uni-app` 模式开发
	- `h5` 或 `web` 或 `mui`项目 (window自带crypto模块) 或 `node`环境使用 `fucking-util-signature`
	- 小程序使用 `fucking-util-signature-uni`

## 使用方法

```javascript

// 多选一
var Signature = require ( "fucking-util-signature" ); 

// 生成公私钥 pkcs #8 (需要其它版本可以提issue)
let { publicKey, privateKey } = Signature.RSA.generateKeys ( );

console.log ( publicKey, privateKey );

// RSA对象
let rsa = new Signature.RSA ( );

// 设置公私钥
rsa.setPublicKey ( publicKey );
rsa.setPrivateKey ( privateKey );

let unsigned = 'hello world!';

// 字符串签名
let sign = rsa.sign ( unsigned, 'base64'/*buffer|binary|hex*/ );

console.log ( 'sign', sign );

// 签名验证
let verified = rsa.verify ( unsigned, sign, 'base64'/*buffer|binary|hex*/ );

console.log ( 'verified', verified );

let unencrypt = 'Unencrypt string';

// 字符串加密
let encrypted = rsa.encrypt ( unencrypt, 'base64'/*buffer|binary|hex*/ );

console.log ( 'encrypted', encrypted );

// 解密
let decrypted = rsa.decrypt ( encrypted );

console.log ( 'decrypted', decrypted );
```

## 高级用法
```javascript

let keys = Signature.RSA.generateKeys ( );

let signature = new Signature ( );

let rsa = signature.rsa ( );
// let md5 = signature.md5 ( );
// let sha256 = signature.sha256 ( );

let unsigned = 'hello world!';

let sign = rsa
.update ( unsigned ) // 更新源数据
.setPrivateKey ( keys.privateKey ) // 设置私钥
.digest ( 'base64' ); // 生成签名

console.log ( 'sign', sign );

let verified = rsa
.update ( unsigned ) // 更新源数据, 此处可以忽略
.setPublicKey ( keys.publicKey ) // 设置公钥
.verify ( sign, 'base64' ); // 验证签名

console.log ( 'verified', verified );
```

## 对象签名/验签
对象签名模式是先把对象转成`querystring`, 对象内的`json`内容会被转成`JSONString`, 然后再加上签名类型字段, 有盐的话会加盐字段, 最后进行字符串签名.
```javascript

let signature = new Signature ( );

// 默认的 formOptions, 可以不用设置
signature.formOptions = {
  signKey: 'sign', // 放置签名的字段
  signTypeKey: 'signType', // 放置签名类型的字段
  signSaltKey: 'key', // 盐值字段
  ignoreKeys: [ // 签名忽略的字段
    'sign', 'key'
  ],
  salt: "" // 盐
};

let signer = signature.sha256 ( );
// let signer = signature.rsa ( );

let form = { user: "1589235", userInfo: { 
  nickname: "Liping Ruan",
  avatar: "@%FFOISJAFJZC"
} };

let { sign, querystring } = signer
.update ( form )
.form ( true ) // 设置 签名/验签 为对象签名模式
.digest ( 'hex' ); // 签名输出 16进制 字符串

console.log ( 'sign', sign );

let verified = signer.verify ( sign, 'hex' );

console.log ( 'verified', verified );
```
#### 加盐和自定义`formOptions`
```javascript

let signature = new Signature ( );

let formOptions = signature.formOptions;

formOptions.signKey = 'SIGN';
formOptions.signTypeKey = 'SIGN_TYPE';
formOptions.signSaltKey = 'SIGN_SALT';
formOptions.ignoreKeys = [ 'SIGN', 'SIGN_SALT' ];
formOptions.salt = "Default salt"; // 默认盐, 可以不设置

let keys = Signature.RSA.generateKeys ( 1024 );

let json = { a:1,b:2 };

let signer = signature.rsa ( )
.update ( json )
.setPublicKey ( keys.publicKey )
.setPrivateKey ( keys.privateKey )
.form ( true )
.formOptions ( {
  salt: "This is salt", // &SIGN_SALT=This%20is%20salt
} );


let { sign, querystring } = signer.digest ( 'hex' );

console.log ( 'sign', sign );
console.log ( 'signed form', json );
console.log ( 'signed querystring', querystring );

let verified = signer.verify ( sign, 'hex' );

console.log ( 'verified', verified );
```