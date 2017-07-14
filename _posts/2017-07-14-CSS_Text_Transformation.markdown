CSST（CSS Text Transformation）
----------

### 什么是 CSST？

一种用 CSS 跨域传输文本的方案。相比 JSONP 更为安全，不需要执行跨站脚本。

## 原理

1、动态创建link标签，将参数通过href属性传给服务端。
2、服务端处理相应操作，并生成css数据，最后将返回的数据放进content属性中。
3、前端通过监听读取 css 样式中的 content 属性获取传送内容。

### 调用流程

![image](https://yunlaiwu0.cn-bj.ufileos.com/csst.png)

### 技术手段

* 怎么监听 `<link>` 加载完毕?

> IE浏览器可以通过onload事件来监听。FF,webkit则可以通过node.sheet.cssRules属性是否存在来判断是否加载完毕，但是需要使用setTimeout轮询。

> 如果是 CSS3 场景，可以取巧用动画开始 (`animationstart`) 这个事件来捕获，但是在server端返回的css代码，需要要加上动画样式。

```js
function cssReady (node, fn) {
        // for IE6-9 and Opera
        if (node.attachEvent) {
          node.attachEvent('onload', fn);
        }
        // polling for Firefox, Chrome, Safari
        else {
          setTimeout(function() {
            poll(node, fn);
          }, 0); // for cache
        }

        function poll(node, fn) {
          if (fn.isCalled) {
            return;
          }

          var isLoaded = false;

          if (/webkit/i.test(navigator.userAgent)) {//webkit
            if (node['sheet']) {
              isLoaded = true;
            }
          }
          // for Firefox
          else if (node['sheet']) {
            try {
              if (node['sheet'].cssRules) {
                isLoaded = true;
              }
            } catch (ex) {
              // NS_ERROR_DOM_SECURITY_ERR
              if (ex.code === 1000) {
                isLoaded = true;
              }
            }
          }

          if (isLoaded) {
            // give time to render.
            setTimeout(function() {
              fn();
            }, 1);
          }
          else {
            setTimeout(function() {
              poll(node, fn);
            }, 1);
          }
        }

      }
```

* 怎么传送特殊字符（"、'、\、\n、\r、\t）？

> Chrome、Safari 对 `content` 样式属性字符解析并不一致

> 为避免未知解析规则影响，统一使用 base64 编码

### 服务器应答的内容

```php
function response() {
    // 前端传来的元素 id，用来将返回的样式绑定到该元素上
    $__callbackId = $_GET['__callbackId'];
    
    $data = [
        "name" => "张三",
        "age" => "33",
        "sex" => "男",
    ];
    $ret = base64_encode(json_encode($data));
    
    header('Content-type: text/css');
    
    $css = "#%s{content='%s'}";
    
    echo sprintf($css, $__callbackId, $ret);
}
```

## 与 JSONP比较

* 缺点：没有 JSONP 适配广，低版本浏览器兼容方面或多或少会存在问题。
* 优点：不会因为跨站脚本对网站安全造成影响。

## 最后，粘贴一下实例：
```html

<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>CSS Text Transformation Example</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=0, minimum-scale=1.0, maximum-scale=1.0">
  <meta name="format-detection" content="telephone=no"/>

</head>
<body>
  <button onclick="fuck()">获取数据</button><br><br>

  id：<input type="text" id="id" value=""><br>
  name：<input type="text" id="name" value=""><br>
  age：<input type="text" id="age" value=""><br>
  sex：<input type="text" id="sex" value=""><br>

</body>
<script>

  (function() {

    function CSST(opts) {

      this.timer = null;

      this.opts = opts || {};

      this.opts.data = this.opts.data || {};

      this.opts.data.__csstCallbackId = opts.id || this.randomString();

      this.head = document.getElementsByTagName('head')[0];

      this.link = document.createElement('link');

      this.span = document.createElement('span');

      this.span.id = this.opts.data.__csstCallbackId;

      this.link.href = this.createUrl();

      this.link.rel = 'stylesheet';

      this.link.type = 'text/css';

      this.head.appendChild(this.link);

      document.documentElement.appendChild(this.span);

      this.main();

    }

    CSST.prototype = {

      main: function() {

        if (this.opts.timeout) {
          this.timeout();
        }

        this.cssReady(this.link, this.handler(this));
      },

      handler: function(context) {

        return function() {
          var computedStyle = window.getComputedStyle(context.span, false);

          var content = computedStyle.content;

          var match = content.match(/[\w+=\/]+/);
          // base64 解码
          if (match) {
            try {
              content = decodeURIComponent(escape(context.base64decode(match[0])));
            }
            catch (ex) {
              context.opts.error && context.opts.error(ex);
              return;
            }
          }

          try {
            content = JSON.parse(content);
          }
          catch (ex) {
            context.opts.error && context.opts.error(new Error('json parse error!'), content);
            return;
          }

          context.opts.success && context.opts.success(content);
          context.cleanup();
        }

      },

      cssReady: function (node, fn) {
        // for IE6-9 and Opera
        if (node.attachEvent) {
          node.attachEvent('onload', fn);
        }
        // polling for Firefox, Chrome, Safari
        else {
          setTimeout(function() {
            poll(node, fn);
          }, 0); // for cache
        }

        function poll(node, fn) {
          if (fn.isCalled) {
            return;
          }

          var isLoaded = false;

          if (/webkit/i.test(navigator.userAgent)) {//webkit
            if (node['sheet']) {
              isLoaded = true;
            }
          }
          // for Firefox
          else if (node['sheet']) {
            try {
              if (node['sheet'].cssRules) {
                isLoaded = true;
              }
            } catch (ex) {
              // NS_ERROR_DOM_SECURITY_ERR
              if (ex.code === 1000) {
                isLoaded = true;
              }
            }
          }

          if (isLoaded) {
            // give time to render.
            setTimeout(function() {
              fn();
            }, 1);
          }
          else {
            setTimeout(function() {
              poll(node, fn);
            }, 1);
          }
        }

      },

      createUrl: function() {

        var url = this.opts.url;

        var param = this.json2param(this.opts.data);

        url += (url.indexOf('?') >= 0 ? '&' : '?') + param;
        
        return url.replace('?&', '?');
      },

      json2param: function(data) {

        if (!data) {
          return '';
        }

        var index, item, params = [];

        for (index in data) {
          item = data[index];
          params.push(index + "=" + item);
        }

        return params.join('&');
      },

      timeout: function() {
        var self = this;

        self.timer = setTimeout(function () {
          self.cleanup();

          if (self.opts.error) {
            self.opts.error(new Error('Timeout'));
          }

        }, self.opts.timeout);
      },

      cleanup: function () {
        if (this.link && this.link.parentNode) {
          this.link.parentNode.removeChild(this.link);
          this.link = null;
        }

        if (this.span && this.span.parentNode) {
          this.span.parentNode.removeChild(this.span);
          this.span = null;
        }

        if (this.timer) {
          clearTimeout(this.timer);
          this.timer = null;
        }
      },

      randomString: function (len) {
        len = len || 16;
        var $chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678';    
        /** **默认去掉了容易混淆的字符oOLl,9gq,Vv,Uu,I1****/
        var maxPos = $chars.length;
        var pwd = '';
        for (i = 0; i < len; i++) {
          pwd += $chars.charAt(Math.floor(Math.random() * maxPos));
        }
        return pwd;
      },

      base64decode: function (input) {  
        var _keyStr = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";  
        var output = "";  
        var chr1, chr2, chr3;  
        var enc1, enc2, enc3, enc4;  
        var i = 0;  
        input = input.replace(/[^A-Za-z0-9\+\/\=]/g, "");  
        while (i < input.length) {  
            enc1 = _keyStr.indexOf(input.charAt(i++));  
            enc2 = _keyStr.indexOf(input.charAt(i++));  
            enc3 = _keyStr.indexOf(input.charAt(i++));  
            enc4 = _keyStr.indexOf(input.charAt(i++));  
            chr1 = (enc1 << 2) | (enc2 >> 4);  
            chr2 = ((enc2 & 15) << 4) | (enc3 >> 2);  
            chr3 = ((enc3 & 3) << 6) | enc4;  
            output = output + String.fromCharCode(chr1);  
            if (enc3 != 64) {  
                output = output + String.fromCharCode(chr2);  
            }  
            if (enc4 != 64) {  
                output = output + String.fromCharCode(chr3);  
            }  
        }  
        output = _utf8_decode(output);  

        function _utf8_decode (utftext) {  
          var string = "";  
          var i = 0;  
          var c = c1 = c2 = 0;  
          while ( i < utftext.length ) {  
              c = utftext.charCodeAt(i);  
              if (c < 128) {  
                  string += String.fromCharCode(c);  
                  i++;  
              } else if((c > 191) && (c < 224)) {  
                  c2 = utftext.charCodeAt(i+1);  
                  string += String.fromCharCode(((c & 31) << 6) | (c2 & 63));  
                  i += 2;  
              } else {  
                  c2 = utftext.charCodeAt(i+1);  
                  c3 = utftext.charCodeAt(i+2);  
                  string += String.fromCharCode(((c & 15) << 12) | ((c2 & 63) << 6) | (c3 & 63));  
                  i += 3;  
              }  
          }  
          return string;  
        } 
        
        return output;  
      }
    };

    window.csst = function(opts) {
      return new CSST(opts);
    }

  })();

</script>

<script>
function fuck () {
  window.csst({
      url: "http://t2.yunlaiwu.com:8099/sem/test",
      data: {
        id: '1001',
        name: '张三',
        age: 23,
        sex: "男"
      },
      success: function(data) {
        if (!data) {
          alert('蛤?');
          return;
        }

        for (var x in data) {
          document.getElementById(x).value = data[x];
        }
      },
      error: function(err) {
        console.error(err)
      }
  });
};
</script>
</html>

```


