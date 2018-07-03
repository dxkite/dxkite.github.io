---
title: '绕过StarUML3 正版验证，去除水印'
date: 2018-06-15 16:01:28
categories:
    - '技术分享'
---

StarUML 是一个方便的UML绘图工具，一直在使用这个，现在StarUML更新到了3.0，对弹窗做了很多的优化，现在支持了原始弹窗，和右键菜单，比以前的右键菜单好多了，但是未注册的导出的图片很尬尬，有水印，学生党什么的，肯定不能购买咯， :joy: ，话不多说，开始分析。

<!-- more -->

**仅供学习，就是为了去除水印，不要拿来干啥子**

## StarUML 2 绕过

StarUML 使用的技术是 `Electorn` ，在 2.x 时代，程序都没打包，安装在程序目录下，具体是 `C:\Program Files (x86)\StarUML` ,这个是默认安装目录，破解方式很简单，找到授权验证的部分，修改下代码就好了，具体文件： `C:\Program Files (x86)\StarUML\www\license\node\LicenseManagerDomain.js` 目录相对于安装目录，文件修改地方就在20多行的验证函数

```javascript
function validate(PK, name, product, licenseKey)
```

将原始函数修改为如下即可，内容随意，只要返回一个授权包即可：

```javascript
function validate(PK, name, product, licenseKey) {
	return { 
            name: "DXkite", 
            product: "StarUML", 
            licenseType: "Personal",
            quantity: "DXkite",
            licenseKey: "DXkite Key"
    };
}
```

## StarUML3 绕过

现在升级到了 3.0 系统，整个系统的安装目录从C盘转到了用户数据文件夹，Windows 10的具体路径 `C:\Users\[用户名]\AppData\Local\Programs\StarUML`

### 解包 app.asar

我的路径为 `C:\Users\DXkite\AppData\Local\Programs\StarUML`，其中管理许可证书的文件在 `C:\Users\DXkite\AppData\Local\Programs\StarUML\resources\app.asar` 文件中，在这里使用 `asar` 命令解压 。

```bash
npm install asar -g
asar e app.asar app
```

解压后可以发现目录下多了个`app`的文件夹。**备份 `app.asar` 文件** ，修改 `app` 文件夹中的文件 `C:\Users\DXkite\AppData\Local\Programs\StarUML\resources\app\src\engine\license-manager.js`找到验证函数 

```javascript
validate () {
    return new Promise((resolve, reject) => {
      try {
        // Local check
        var file = this.findLicense()
        if (!file) {
          reject('License key not found')
        } else {
          var data = fs.readFileSync(file, 'utf8')
          licenseInfo = JSON.parse(data)
          var base = SK + licenseInfo.name +
            SK + licenseInfo.product + '-' + licenseInfo.licenseType +
            SK + licenseInfo.quantity +
            SK + licenseInfo.timestamp + SK
          var _key = crypto.createHash('sha1').update(base).digest('hex').toUpperCase()
          if (_key !== licenseInfo.licenseKey) {
            reject('Invalid license key')
          } else {
            // Server check
            $.post(app.config.validation_url, {licenseKey: licenseInfo.licenseKey})
              .done(data => {
                resolve(data)
              })
              .fail(err => {
                if (err && err.status === 499) { /*License key not exists*/
                  reject(err)
                } else {
                  // If server is not available, assume that license key is valid
                  resolve(licenseInfo)
                }
              })
          }
        }
      } catch (err) {
        reject(err)
      }
    })
  }
```

修改验证逻辑, 调用 `resolve` 方法，给定一个可用的许可，内容随意
 
```javascript
  validate () {
    return new Promise((resolve, reject) => {
		resolve({
			name : "DXkite",
			product : "DXkite product",
			licenseType : "DXkite Personal",
			quantity : "DXkite Quantity",
			timestamp :"1529049036",
		});
    })
  }
```

### 打包 app 文件夹

现在证书验证逻辑已经被修改了，我们所需要做的就是继续打包回去。

```bash
 asar p app app.asar
```

打包回去以后你会发现你的StarUML3就被破解了。

> 文章纯属记录，部分细节问题就没说，主要是破解程序还是不要太皮。有钱的可以买一个，我们程序员不容易
