# 附录

## 代码

### 控制图Windows

#### 移动窗口

```javascript
function fixedWindow(size) {
    this.name = 'window'; 
    this.ready = false; 
    this.points = []; 
    this.total = 0;
    this.sos = 0;
    this.push = function(newValue) {
        if (this.points.length == size) {
            var removed = this.points.shift(); 
            this.total -= removed;
            this.sos -= removed*removed;
        }
        this.total += newValue;
        this.sos += newValue*newValue; this.points.push(newValue);
        this.ready = (this.points.length == size);
    }
    this.mean = function() {
        if (this.points.length == 0) {
            return 0; 
        }
        return this.total / this.points.length;
    }
    this.stddev = function() {
        var mean = this.mean();
        return Math.sqrt(this.sos/this.points.length - mean*mean);
    }
}
var window = new fixedWindow(5); window.push(1);
window.push(5);
window.push(9); 
console.log(window); 
console.log(window.mean()); 
console.log(window.stddev()*3);

```

#### EWMA窗口

```javascript
function movingAverage(alpha) {
    this.name = 'ewma'; 
    this.ready = true;
    function ma() {
        this.value = NaN;
        this.push = function(newValue) {
            if (isNaN(this.value)) { 
                this.value = newValue; 
                ready = true;
                return;
            }
            this.value = alpha*newValue + (1 - alpha)*this.value; };
    }
    this.MA = new ma(alpha);
    this.sosMA = new ma(alpha);
    this.push = function(newValue) { 
        this.MA.push(newValue); 
        this.sosMA.push(newValue*newValue);
    };
    this.mean = function() { 
        return this.MA.value;
    };
    this.stddev = function() {
        return Math.sqrt(this.sosMA.value - this.mean()*this.mean());
    }
}
var ma = new movingAverage(0.5); 
ma.push(1);
ma.push(5);
ma.push(9);
console.log(ma);
console.log(ma.mean());
console.log(ma.stddev()*3);
```

#### 窗口功能

```javascript
function kernelSmoothing(weights) {
    this.name = 'kernel';
    this.ready = false;
    this.points = [];
    this.lag = (weights.length-1)/2;
    this.push = function(newValue) {
        if (this.points.length == weights.length) {
            var removed = this.points.shift();
        }
        this.points.push(newValue);
        this.ready = (this.points.length == weights.length);
    }
    this.mean = function() {
        var total = 0;
        for (var i = 0; i < weights.length; i++) {
            total += weights[i]*this.points[i]; 
        }
        return total;
    }
    this.stddev = function() {
        var mean = this.mean();
        varsos=0;
        for (var i = 0; i < weights.length; i++) {
            sos += weights[i]*this.points[i]*this.points[i]; 
        }
        return Math.sqrt(sos - mean*mean);
    }
}
var ksmooth = new kernelSmoothing([0.3333, 0.3333, 0.3333]); ksmooth.push(1);
ksmooth.push(5);
ksmooth.push(9);
console.log(ksmooth);
console.log(ksmooth.mean());
console.log(ksmooth.stddev()*3);
```
# 关于作者

Baron Schwartz是下一代数据库监控解决方案VividCortex的创始人兼首席执行官。 他广泛谈论数据库性能，可伸缩性和开源的主题。 他是O'Reilly畅销书High Performance MySQL的作者，以及许多用于MySQL管理的开源工具。 他也是Oracle ACE并经常参与PostgreSQL社区。
Preetam Jinka是VividCortex的工程师，也是弗吉尼亚大学的本科生，他在那里学习统计学和时间序列。

# 致谢
我们要感谢George Michie，他为本书贡献了一些内容，并帮助我们澄清并保持适当的细节。
