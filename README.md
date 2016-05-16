## 极简天气
一款简单的chrome天气插件。

如图：

![截图](./images/1.png "截图")

戳这里右键另存为：<https://github.com/yohnz/weather/weather.crx>


### 创建文件
新建weather文件夹，里面包含manifest.json，popup.html和images文件夹。images文件夹放16,48,128三种不同尺寸的图标

manifest.json内代码如下：
```
{
  "manifest_version":2,
  "name":"极简天气",
  "description":"极简天气预报",
  "version":"1.0",
  "icons": {
        "16": "images/sun16.png",
        "48": "images/sun48.png",
        "128": "images/sun128.png"
   },
  "browser_action":{
      "default_icon":"images/sun48.png",
      "default_title":"天气预报",
      "default_popup":"popup.html"
  },
   
}
```

popup.html的代码如下：
```
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>天气</title>
</head>
<body>
 <div class="weather">    
    Test    
 </div>
</body>
</html>
```
### 文件说明

**manifest.json**

必需文件，是整个扩展的入口，每个Chrome扩展都包含一个Manifest文件。Manifest必须包含name、version和manifest_version属性。

属性说明：

- `manifest_version`指定文件格式的版本，在Chrome18之后，应该都是2
- `name`扩展名称
- `version` 扩展版本号
- `version`扩展的版本
- `icons`扩展列表图标
- `browser_action`指定扩展在Chrome工具栏中的显示信息。
- `default_icon`、`default_title`、`default_popup`依次指定图标、标题、对应的页面

**Popup页面**

Popup页面是当用户点击扩展图标时，展示在图标下面的页面。

打开chrome扩展程序界面，勾选"开发者模式",拖入weather文件夹，然后就可以看到weather扩展已经出现在chrome扩展程序列表了

![](./images/1.jpg)

同时，工具栏也出现了weather的图标，点击之后会弹出popup界面:

![](./images/2.jpg)


### 完善页面和样式
完善静态popup页面,模拟天气数据:
```
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>天气</title>
</head>
<body>
    <div class="weather">
        <div class="today" id="today">
            <h1 class="city">厦门</h1>
            <div class="row_detail"><img src="http://files.heweather.com/cond_icon/104.png">
                <h1>19<span>℃</span></h1></div>
            <div class="wind">
                <h2>阴</h2>
                <h4>风速 20   湿度 89%</h4></div>
        </div>
        <div class="content">
            <div class="wrap" id="wrap">
                <div class="row">
                    <h4>2016-05-16</h4><img src="http://files.heweather.com/cond_icon/104.png">
                    <h1>19~24</h1>
                    <h4>阴</h4>
                </div>               
            </div>
        </div>
    </div>
</body>
</html>
```

新建CSS文件，并在popup页面引入
```
body{
    width:740px;
    height:400px;
    font-family: 'Microsoft Yahei';
    color:#333;
    background:#fefefe;
    text-shadow:1px 1px 6px #333;
}

.city{
    text-align:center
}
.today{
    padding-bottom:30px;
}
.row_detail{
    display: flex;
    direction: row;
    justify-content:center;
    align-items: center;
}
.row_detail img{
    width:80px;    
}
.row_detail h1{
    font-size:60px;
}
.wind{
    text-align: center;
}
.content{
    display: flex;
    direction: column
}

.wrap{
    display: flex;
    direction: row;
    flex: 1;
    justify-content:space-around;
    align-items: center;
}
.row{
    background:#fff;
    border:1px solid #ccc;
    padding:10px;
    box-shadow: 0 2px 10px rgba(0,0,0,.3);
}
.row img{
    width:80px;
}
.row h1{
    font-size:18px;
}
h1,h4{
    text-align: center;
    margin:0;
}

```
点击工具栏weather图标，此时界面如图：

![](./images/3.jpg)

### 获取真实天气数据
Google允许Chrome扩展应用不必受限于跨域限制。但出于安全考虑，需要在Manifest的permissions属性中声明需要跨域的权限。
这里以[和风天气API](http://www.heweather.com/)为例.
首先，在Manifest里添加要请求的API接口：
```
"permissions":[
     "http://api.openweathermap.org/data/2.5/forecast?q=*",   
  ]
```
然后新建popup.js并在popup页面中引入
简单的ajax函数：
```
function httpRequest(url,callback) {
    var xhr = new XMLHttpRequest();
    xhr.open('GET',url,true);
    xhr.onreadystatechange = function() {
        if(xhr.readyState === 4){
            callback(xhr.responseText);
        }
    }
    xhr.send();
}
```
和风天气API可以通过城市名称，城市代码,IP三种方式来获取指定城市天气数据，不过经过测试发现，IP获取的方式不准确，城市有偏差，所以，直接用城市名称来获取。这里借用`http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=json`获取城市名称。
```
httpRequest('http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=json',function(data) {
    if(!data) return;
    data = JSON.parse(data);
    var city = data.city;
    var url = 'https://api.heweather.com/x3/weather?city='+city+'&key=youkey';
        httpRequest(url,function(data) {
            data = JSON.parse(data);
            var result = data["HeWeather data service 3.0"][0];        
            showWeather(city,result);            
        });
});
```
为了方便的解析图片，构建一个json
```
var cond_info = {
100:"http://files.heweather.com/cond_icon/100.png",
101:"http://files.heweather.com/cond_icon/101.png",
102:"http://files.heweather.com/cond_icon/102.png",
103:"http://files.heweather.com/cond_icon/103.png",
104:"http://files.heweather.com/cond_icon/104.png",
200:"http://files.heweather.com/cond_icon/200.png",
201:"http://files.heweather.com/cond_icon/201.png",
202:"http://files.heweather.com/cond_icon/202.png",
203:"http://files.heweather.com/cond_icon/203.png",
204:"http://files.heweather.com/cond_icon/204.png",
205:"http://files.heweather.com/cond_icon/205.png",
206:"http://files.heweather.com/cond_icon/206.png",
207:"http://files.heweather.com/cond_icon/207.png",
208:"http://files.heweather.com/cond_icon/208.png",
209:"http://files.heweather.com/cond_icon/209.png",
210:"http://files.heweather.com/cond_icon/210.png",
211:"http://files.heweather.com/cond_icon/211.png",
212:"http://files.heweather.com/cond_icon/212.png",
213:"http://files.heweather.com/cond_icon/213.png",
300:"http://files.heweather.com/cond_icon/300.png",
301:"http://files.heweather.com/cond_icon/301.png",
302:"http://files.heweather.com/cond_icon/302.png",
303:"http://files.heweather.com/cond_icon/303.png",
304:"http://files.heweather.com/cond_icon/304.png",
305:"http://files.heweather.com/cond_icon/305.png",
306:"http://files.heweather.com/cond_icon/306.png",
307:"http://files.heweather.com/cond_icon/307.png",
308:"http://files.heweather.com/cond_icon/308.png",
309:"http://files.heweather.com/cond_icon/309.png",
310:"http://files.heweather.com/cond_icon/310.png",
311:"http://files.heweather.com/cond_icon/311.png",
312:"http://files.heweather.com/cond_icon/312.png",
313:"http://files.heweather.com/cond_icon/313.png",
400:"http://files.heweather.com/cond_icon/400.png",
401:"http://files.heweather.com/cond_icon/401.png",
402:"http://files.heweather.com/cond_icon/402.png",
403:"http://files.heweather.com/cond_icon/403.png",
404:"http://files.heweather.com/cond_icon/404.png",
405:"http://files.heweather.com/cond_icon/405.png",
406:"http://files.heweather.com/cond_icon/406.png",
407:"http://files.heweather.com/cond_icon/407.png",
500:"http://files.heweather.com/cond_icon/500.png",
501:"http://files.heweather.com/cond_icon/501.png",
502:"http://files.heweather.com/cond_icon/502.png",
503:"http://files.heweather.com/cond_icon/503.png",
504:"http://files.heweather.com/cond_icon/504.png",
506:"http://files.heweather.com/cond_icon/506.png",
507:"http://files.heweather.com/cond_icon/507.png",
508:"http://files.heweather.com/cond_icon/508.png",
900:"http://files.heweather.com/cond_icon/900.png",
901:"http://files.heweather.com/cond_icon/901.png",
999:"http://files.heweather.com/cond_icon/999.png"
}
```
showWeather()函数构建DOM;

```
function showWeather(city,result) { 
    var daily = result.daily_forecast;
    var now = result.now;
    var dailyDom='';
    for(var i=0;i<daily.length;i++){
        var day = daily[i];
         dailyDom += '<div class="row">'
            +'<h4>'+day.date+'</h4>'
            +'<img src="'+cond_info[day.cond.code_d]+'" />'
            +'<h1>'+day.tmp.min+'~'+day.tmp.max+'</h1>'
            +'<h4>'+day.cond.txt_d+'</h4>'     
        +'</div>'       
    }
    var today = '<h1 class="city">'+city+'</h1>'
                +'<div class="row_detail">'
                    +'<img src="'+cond_info[now.cond.code]+'" />'
                    +'<h1>'+now.tmp+'<span>℃</span></h1>'            
                +'</div>'
                +'<div class="wind">'
                    +'<h2>'+now.cond.txt+'</h2>'
                    +'<h4>风速 '+now.wind.spd+'   湿度 '+now.hum+'%</h4>'            
                +'</div>'
    
    document.getElementById('today').innerHTML = today;
    document.getElementById('wrap').innerHTML = dailyDom; 
}
```

这时，再点击weather图标，天气扩展基本上就完成了，不过因为和风API有请求次数限制，也为了减少请求，这里做一下数据缓存。
```
var _time = new Date().getTime()-(60*60*1000*2); //接口次数有限，两小时请求一次
var storageTime = localStorage.updateTime||0;

httpRequest('http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=json',function(data) {
    if(!data) return;
    data = JSON.parse(data);
    var city = data.city;
    var url = 'https://api.heweather.com/x3/weather?city='+city+'&key=youkey';
    if(_time>storageTime){
        httpRequest(url,function(data) {
            data = JSON.parse(data);
            var result = data["HeWeather data service 3.0"][0];        
            showWeather(city,result);
            localStorage.updateTime = new Date().getTime();  
            localStorage.data = JSON.stringify(result);    
        });
    }else{
        var result = JSON.parse(localStorage.data);
        showWeather(city,result);
    }

});
```

至此，一个简单的chrome天气扩展就完成了，是不是比想象中更简单。
