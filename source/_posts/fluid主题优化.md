---
title: fluid主题优化
date: 2022-03-17 23:33:12
tags: 
 - hexo
 - 花里胡哨
categories: 环境配置
math: false
index_img: /img/article/hexo_fluid.png
---

虽然fluid 主题已经非常好看了，但我依旧可以通过一些简单的操作，让你的博客更加易用（更花里胡哨和装逼）:smile:

## 1. Fluid 页脚增加网站运行时长

主题预览网站中有了很详细的介绍，这里就不在重复了

[配置链接](https://hexo.fluid-dev.com/posts/fluid-footer-custom/)，效果如下：

![202203172342237](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261337892.png)

## 2. markdown 使用emoji

默认不支持输入emoji表情，我们可以通过hexo-filter-emoji插件实现

```bash
npm install hexo-filter-emoji
```

之后在hexo配置文件，_config.html中添加

```yml
## hexo-filter-emoji
emoji:
  enable: true
  className: github-emoji
  styles:
  customEmojis:
```

效果如下:thumbsup::

![202203172346789](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261338900.png)

## 3. 使用Tidio进行实时交流

 Tidio是一个客服服务平台，可以在blog中实现Live Chat,存在网页端和移动端，可以及时回复消息，同时免费功能足够我们使用，效果如下：

![202203191703130](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261337746.png)

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261338806.png" alt="202203191703265" style="zoom:50%;" />

- 首先进入 [Tidio 官网](https://www.tidio.com/) 进行注册，可能需要魔法上网

- 选择live chat 在settings 中调整自己的语句和外观

- 在setting->developver中复制public key

  <img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202207031414402.png" alt="202203191711294" style="zoom:50%;" />

- 在主题的配置文件中添加

  ````yml
  # Tidio online chat
  tidio:
    enable: true
    key: your public key
  ````

- 更改主题文件 

  `/hexo-theme-fluid/layout/_partials/footer.ejs`中添加

  ```ejs
  <!-- 在线通讯Tidio -->
  <% if (theme.tidio.enable){ %>
  	<script src="//code.tidio.co/your_public_key.js"></script>
  <% } %>
  ```

  ![image-20230626225056114](https://cdn.jsdelivr.net/gh/F7kyyy/picture/img/202306262251211.png)

- 重新部署

  ```bash
  hexo clean
  hexo g
  hexo s
  ```

  

## 4. 添加特效，鼠标点击，背景

在`source/js/`文件夹下添加自己需要的js文件`background.js`与`fireworks.js`,在主题配置中引入。

`backgound.js` 背景添加多边形：

```js
// 背景线条 bynote.cn
(function () {
    function getAttributeOrDefault(element, attribute, defaultValue) {
        return element.getAttribute(attribute) || defaultValue;
    }

    function getElementsByTagName(tagName) {
        return document.getElementsByTagName(tagName);
    }

    function getOptions() {
        var scripts = getElementsByTagName("script");
        var scriptCount = scripts.length;
        var script = scripts[scriptCount - 1];
        return {
            count: getAttributeOrDefault(script, "count", 99),
            color: getAttributeOrDefault(script, "color", "0,255,128"),
            opacity: getAttributeOrDefault(script, "opacity", 0.5),
            zIndex: getAttributeOrDefault(script, "zIndex", -1),
        };
    }

    function resizeCanvas() {
        canvas.width = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;
        canvas.height = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;
    }

    function draw() {
        context.clearRect(0, 0, canvas.width, canvas.height);
        var particles = [mainParticle].concat(otherParticles);
        var particle, otherParticle, distance, xDiff, yDiff, alpha, lineWidth, strokeStyle;
        otherParticles.forEach(function (otherParticle) {
            otherParticle.x += otherParticle.xa;
            otherParticle.y += otherParticle.ya;
            otherParticle.xa *= otherParticle.x > canvas.width || otherParticle.x < 0 ? -1 : 1;
            otherParticle.ya *= otherParticle.y > canvas.height || otherParticle.y < 0 ? -1 : 1;
            context.beginPath();
            context.arc(otherParticle.x, otherParticle.y, 2, 0, 2 * Math.PI);
            context.fillStyle = "rgb(135, 206, 250)";
            context.fill();
            for (var i = 0; i < particles.length; i++) {
                particle = particles[i];
                if (otherParticle !== particle && particle.x !== null && particle.y !== null) {
                    xDiff = otherParticle.x - particle.x;
                    yDiff = otherParticle.y - particle.y;
                    distance = xDiff * xDiff + yDiff * yDiff;
                    if (distance < particle.max) {
                        if (particle === mainParticle && distance >= particle.max / 2) {
                            otherParticle.x -= 0.03 * xDiff;
                            otherParticle.y -= 0.03 * yDiff;
                        }
                        alpha = (particle.max - distance) / particle.max;
                        lineWidth = alpha;
                        strokeStyle = "rgba(" + options.color + "," + (alpha + 0.2) + ")";
                        context.beginPath();
                        context.lineWidth = lineWidth;
                        context.strokeStyle = strokeStyle;
                        context.moveTo(otherParticle.x, otherParticle.y);
                        context.lineTo(particle.x, particle.y);
                        context.stroke();
                    }
                }
            }
            particles.splice(particles.indexOf(otherParticle), 1);
        });
        requestAnimationFrame(draw);
    }

    var canvas = document.createElement("canvas");
    var options = getOptions();
    var canvasId = "c_n" + options.count;
    var context = canvas.getContext("2d");
    var mainParticle = { x: null, y: null, max: 20000 };
    var otherParticles = [];
    var requestAnimationFrame =
        window.requestAnimationFrame ||
        window.webkitRequestAnimationFrame ||
        window.mozRequestAnimationFrame ||
        window.oRequestAnimationFrame ||
        window.msRequestAnimationFrame ||
        function (callback) {
            window.setTimeout(callback, 1000 / 45);
        };
    var random = Math.random;

    canvas.id = canvasId;
    canvas.style.cssText =
        "position: fixed; top: 0; left: 0; z-index: " +
        options.zIndex +
        "; opacity: " +
        options.opacity +
        ";";
    getElementsByTagName("body")[0].appendChild(canvas);

    resizeCanvas();
    window.addEventListener("resize", resizeCanvas);
    window.addEventListener("mousemove", function (event) {
        mainParticle.x = event.clientX;
        mainParticle.y = event.clientY;
    });
    window.addEventListener("mouseout", function () {
        mainParticle.x = null;
        mainParticle.y = null;
    });

    for (var i = 0; i < options.count; i++) {
        var x = random() * canvas.width;
        var y = random() * canvas.height;
        var xa = 2 * random() - 1;
        var ya = 2 * random() - 1;
        otherParticles.push({ x: x, y: y, xa: xa, ya: ya, max: 6000 });
    }

    setTimeout(function () {
        draw();
    }, 100);
})();
```

`fireworks.js` 鼠标点击气泡特效

```js
class Circle {
  constructor({ origin, speed, color, angle, context }) {
    this.origin = origin
    this.position = { ...this.origin }
    this.color = color
    this.speed = speed
    this.angle = angle
    this.context = context
    this.renderCount = 0
  }

  draw() {
    this.context.fillStyle = this.color
    this.context.beginPath()
    this.context.arc(this.position.x, this.position.y, 4, 0, Math.PI * 2)
    this.context.fill()
  }

  move() {
    this.position.x = (Math.sin(this.angle) * this.speed) + this.position.x
    this.position.y = (Math.cos(this.angle) * this.speed) + this.position.y + (this.renderCount * 0.3)
    this.renderCount++
  }
}

class Boom {
  constructor({ origin, context, circleCount = 16, area }) {
    this.origin = origin
    this.context = context
    this.circleCount = circleCount
    this.area = area
    this.stop = false
    this.circles = []
  }

  randomArray(range) {
    const length = range.length
    const randomIndex = Math.floor(length * Math.random())
    return range[randomIndex]
  }

randomColor() {
  const range = ['FF5733', 'FFC300', 'DAF7A6', 'C70039', '900C3F', '581845', '00FFFF', 'FF00FF', 'FFFF00']
  return '#' + this.randomArray(range)
}

  randomRange(start, end) {
    return (end - start) * Math.random() + start
  }

  init() {
    for (let i = 0; i < this.circleCount; i++) {
      const circle = new Circle({
        context: this.context,
        origin: this.origin,
        color: this.randomColor(),
        angle: this.randomRange(Math.PI - 1, Math.PI + 1),
        speed: this.randomRange(1, 3)
      })
      this.circles.push(circle)
    }
  }

  move() {
    this.circles.forEach((circle, index) => {
      if (circle.position.x > this.area.width || circle.position.y > this.area.height) {
        return this.circles.splice(index, 1)
      }
      circle.move()
    })
    if (this.circles.length == 0) {
      this.stop = true
    }
  }

  draw() {
    this.circles.forEach(circle => circle.draw())
  }
}

class CursorSpecialEffects {
  constructor() {
    this.computerCanvas = document.createElement('canvas')
    this.renderCanvas = document.createElement('canvas')

    this.computerContext = this.computerCanvas.getContext('2d')
    this.renderContext = this.renderCanvas.getContext('2d')

    this.globalWidth = window.innerWidth
    this.globalHeight = window.innerHeight

    this.booms = []
    this.running = false
  }

  handleMouseDown(e) {
    const boom = new Boom({
      origin: { x: e.clientX, y: e.clientY },
      context: this.computerContext,
      area: {
        width: this.globalWidth,
        height: this.globalHeight
      }
    })
    boom.init()
    this.booms.push(boom)
    this.running || this.run()
  }

  handlePageHide() {
    this.booms = []
    this.running = false
  }

  init() {
    const style = this.renderCanvas.style
    style.position = 'fixed'
    style.top = style.left = 0
    style.zIndex = '999999999999999999999999999999999999999999'
    style.pointerEvents = 'none'

    style.width = this.renderCanvas.width = this.computerCanvas.width = this.globalWidth
    style.height = this.renderCanvas.height = this.computerCanvas.height = this.globalHeight

    document.body.append(this.renderCanvas)

    window.addEventListener('mousedown', this.handleMouseDown.bind(this))
    window.addEventListener('pagehide', this.handlePageHide.bind(this))
  }

  run() {
    this.running = true
    if (this.booms.length == 0) {
      return this.running = false
    }

    requestAnimationFrame(this.run.bind(this))

    this.computerContext.clearRect(0, 0, this.globalWidth, this.globalHeight)
    this.renderContext.clearRect(0, 0, this.globalWidth, this.globalHeight)

    this.booms.forEach((boom, index) => {
      if (boom.stop) {
        return this.booms.splice(index, 1)
      }
      boom.move()
      boom.draw()
    })
    this.renderContext.drawImage(this.computerCanvas, 0, 0, this.globalWidth, this.globalHeight)
  }
}


(function cursorFireworksCusor() {
  const cursorSpecialEffects = new CursorSpecialEffects()
  cursorSpecialEffects.init()
})();


```

在`_config.fluid.yml`引入

```yaml
# 指定自定义 .js 文件路径，支持列表；路径是相对 source 目录，如 /js/custom.js 对应存放目录 source/js/custom.js
# Specify the path of your custom js file, support list. The path is relative to the source directory, such as `/js/custom.js` corresponding to the directory `source/js/custom.js`
custom_js:
  - //cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.js  
  - //cdn.jsdelivr.net/gh/metowolf/Metingjs@1.2/dist/Meting.min.js 
  - //cdn.bootcss.com/animejs/2.2.0/anime.min.js
  - /js/background.js
  - /js/fireworks.js
```

