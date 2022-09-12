# day8 电商数据可视化

后端提供数据接口 ---> 后端通过URL，把请求路由到一个函数，这个函数返回一个字典 ---> 浏览器就会获得一个JSON格式的数据  (服务器)
前端通过JavaScript获取网络请求 ---> fetch(axios更好) ---> 渲染网页

## 1.sql回顾

```sql
use data_viz;
select count(*) from tb_order;


-- 查询2019年GMV(Gross Merchandise Volume) - 交易流水- -订单金额求和
select sum(orderAmount)  from tb_order where payTime between '2019-1-01-01' and '2019-12-31';

-- 查询2019年销售额
select sum(payment)  from tb_order where payTime between '2019-1-01-01' and '2019-12-31';

-- 查询2019年的实际销售额 ---> 不包含退货的
select sum(payment)  from tb_order 
where payTime between '2019-1-01-01' and '2019-12-31' and chargeback='否';

-- 查询2019年用户客单价 -ARPPU（Average Revenue Per Paid User）---> 用户有重复值 实际支付的金额
select sum(payment)/(count(distinct userID))  from tb_order 
where payTime between '2019-1-01-01' and '2019-12-31' and chargeback='否';

-- 查询2019年每月的GMV
select sum(orderAmount)  from tb_order group by month(payTime);

-- 查询2019年各平台转化用户数量
select count(distinct userID) from tb_order group by platformType;

-- 查询2019年各渠道转化用户数量
select count(distinct userID) from tb_order group by chanelID;

```

## 2. 利用Python建立接口

    @app.route('/api/接口名称')
    def 接口函数名():
        conn = get_mysql_connection()  #  利用定义的函数从mysql中获取数据
        try:
            with conn.cursor(pymysql.cursors.DictCursor) as cursor:
                cursor.execute('mysql语句')
                row_dict = cursor.fetchone()    # 取值         
        except pymysql.MySQLError as err:
            print(err)
        finally:
            conn.close()
        return {'results': results}  # 把结果以JOSN格式输出

参考电商数据的main.py

## 3. 利用ECharts画图

html语言和ECharts进行渲染

    let 数据图形变量名 = echarts.init(document.querySelector('#gmv'), 'dark') # 初始化
                fetch('前面Python所建立的接口')
                    .then(resp => resp.json())
                    .then(json => {
                        let 数据图形变量名Option = {
                            title: {
                                text: '',
                                top: '5%',  # 距离上边缘向下移动图片高度的5%  
                                left: '5%'   # 距离图片左边缘为5%
                            },
                            legend: {
                                data: ['GMV'],
                                top: '5%'
                            },
                            xAxis: {
                                type: '横坐标数据类型',
                                data: json.横坐标数据
                            },
                            yAxis: {
                                type: '纵坐标数据类型',
                                data: json.纵坐标数据
                            },
                            grid: {
                                bottom: '10%'  # 
                            },
                            series: [
                                {
                                    name: 'GMV',
                                    type: 'line', # 折线 还可以选bar kline 很多中
                                    data: json.数据,
                                }
                            ]
                        }
                        数据图形变量名Chart.setOption(数据图形变量名Option) #根据ECharts设置 图形
                    })

#### 附录

##### main.py

```python
import random
import pymysql
from flask import Flask, redirect, request
from flask_cors import CORS
from utils import get_mysql_connection


app = Flask(__name__)
CORS(app)


@app.route('/')
def show_index():
    # 将请求重定向到static目录下的index.html
    return redirect('/static/index.html')


# API - Application Programming Interface - 应用程序编程接口
# 网络API（网络数据接口）- 请求这个URL就可以获得对应的数据（通常是JSON格式）
@app.route('/api/general_data')
def get_general_data():
    names = ('GMV', '销售额', '实际销售额', '客单价')
    divsors = (10000, 10000, 10000, 1)
    units = ('万元', '万元', '万元', '元')
    values = [0] * 4
    conn = get_mysql_connection()
    try:
        sql_queries = [
            'select sum(orderAmount) from tb_order',
            'select sum(payment) from tb_order',
            'select sum(payment) from tb_order where chargeback="否"',
            'select sum(payment) / count(distinct userID) from tb_order where chargeback="否"'
        ]
        with conn.cursor() as cursor:
            for i, query in enumerate(sql_queries):  # enumerate 返回index和值
                cursor.execute(query)
                values[i] = round(float(cursor.fetchone()[0]) / divsors[i], 2)
    except pymysql.MySQLError as err:
        print(err)
    finally:
        conn.close()
    results = [{'name': names[i], 'unit': units[i], 'value': values[i]} for i in range(4)]   
    # 列表的生成式语法
    return {'results': results}


@app.route('/api/gmv_by_month')
def get_gmv_by_month():
    conn = get_mysql_connection()
    months, gmvs = [], []
    try:
        with conn.cursor(pymysql.cursors.DictCursor) as cursor:
            cursor.execute('select month(orderTime) as month, sum(orderAmount) as gmv from tb_order group by month')
            row_dict = cursor.fetchone()
            while row_dict:
                months.append(f'{row_dict["month"]}月')
                gmvs.append(round(float(row_dict['gmv']) / 10000, 2))
                row_dict = cursor.fetchone()
    except pymysql.MySQLError as err:
        print(err)
    finally:
        conn.close()
    return {'x': months, 'y': gmvs}


@app.route('/api/channel_data')
def get_channel_data():
    conn = get_mysql_connection()
    try:
        with conn.cursor(pymysql.cursors.DictCursor) as cursor:
            cursor.execute('select chanelID as name, count(distinct userID) as value from tb_order group by chanelID order by value desc')
            results = cursor.fetchall()
    except pymysql.MySQLError as err:
        print(err)
    finally:
        conn.close()
    return {'results': results}


@app.route('/api/sales_data')
def get_sales_data():
    y1_data = [random.randrange(10, 41) for _ in range(6)]
    y2_data = [random.randrange(20, 51) for _ in range(6)]
    y3_data = [random.randrange(30, 41) for _ in range(6)]
    y4_data = [random.randrange(20, 31) for _ in range(6)]
    return {'y1': y1_data, 'y2': y2_data, 'y3': y3_data, 'y4': y4_data}


@app.route('/api/stock_data')
def get_stock_data():
    conn = pymysql.connect(host='localhost', port=3306,
                           user='guest', password='Guest.618',
                           database='stock', charset='utf8mb4')
    start = request.args.get('start', '2020-1-1')
    end = request.args.get('end', '2020-12-31')
    x_data, y_data = [], []
    try:
        with conn.cursor(pymysql.cursors.DictCursor) as cursor:
            cursor.execute(
                'select trade_date, open_price, close_price, low_price, high_price '
                'from tb_baba_stock where trade_date between %s and %s',
                (start, end)
            )
            row_dict = cursor.fetchone()
            while row_dict:
                x_data.append(row_dict['trade_date'].strftime('%Y-%m-%d'))
                y_data.append([
                    float(row_dict['open_price']), float(row_dict['close_price']),
                    float(row_dict['low_price']), float(row_dict['high_price'])
                ])
                row_dict = cursor.fetchone()
    except pymysql.MySQLError as err:
        print(err)
    finally:
        conn.close()
    return {'x': x_data, 'y': y_data}


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000, debug=True)

```

#### html-index.html

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>首页</title>
        <style>
            /* 通配符选择器 */
            * {
                margin: 0;
                padding: 0;
            }
            body {
                background-color: black;
            }
            h1 {
                text-align: center;
                color: white;
            }
            #general {
                width: 840px;
                margin: 20px auto;
                text-align: center;
            }
            #general>li {
                /* float: left; */
                display: inline-block;
                margin: 10px 20px;
                list-style: none;
                color: white;
            }
            #general>li>img, #general>li>div {
                vertical-align: middle;
            }
            .detail {
                font-size: 12px;
            }
            #chart>div {
                float: left;
                width: 480px;
                height: 320px;
                margin: 10px 50px;
                border: 1px solid white;
            }
            #chart {
                width: 1200px;
                margin: 0 auto;
            }
            #cal {
                width: 520px;
                margin: 10px auto;
                text-align: center;
            }
            #cal>input, #cal>button {
                margin: 0 4px;
                border: none;
                outline: none;
            }
            #cal>button {
                color: white;
                background-color: darkcyan;
                width: 58px;
                height: 22px;
            }
        </style>
    </head>
    <body>
        <h1>电商订单数据可视化项目</h1>
        <hr>
        <div id="app">
            <ul id="general">
                <li v-for="item in data_items">
                    <img src="images/medal.png" width="36">
                    <div style="display: inline-block">
                        <div class="detail">{{ item.value | numberFormat }}{{ item.unit }}</div>
                        <div class="detail">{{ item.name }}</div>
                    </div>
                </li>
            </ul>
        </div>
        <!-- 如果需要在前端生成统计图表，最好的选择是ECharts和D3.js -->
        <!-- 创建一个用于绘图的div，通过CSS指定好绘图区域的宽度和高度 -->
        <div id="chart">
            <div id="gmv"></div>
            <div id="bar"></div>
            <div id="channel"></div>
            <div id="kline"></div>
        </div>
        <!-- <script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.14/vue.min.js"></script> -->
        <script src="/static/js/vue.min.js"></script>
     <script>
         // 通过JavaScript代码获取JSON数据，然后交给Vue.js渲染页面
            let app = new Vue({
                el: '#app',
                data: {
                    data_items: []
                },
                filters: {
                    numberFormat(x) {
                        return x.toLocaleString()
                    }
                },
                created() {
                    fetch('/api/general_data')
                        .then(resp => resp.json())
                        .then(json => this.data_items = json.results)
                }
            })
     </script>
        <!-- 第一步：引入ECharts的JS文件 -->
        <script src="/static/js/echarts.min.js"></script>
        <!-- <script src="https://cdn.bootcdn.net/ajax/libs/echarts/5.1.2/echarts.min.js"></script> -->
        <!-- 使用ECharts库实现图表渲染 -->
        <script>
            let gmvChart = echarts.init(document.querySelector('#gmv'), 'dark')
            fetch('/api/gmv_by_month')
                .then(resp => resp.json())
                .then(json => {
                    let gmvOption = {
                        title: {
                            text: 'GMV走势',
                            top: '5%',
                            left: '5%'
                        },
                        legend: {
                            data: ['GMV'],
                            top: '5%'
                        },
                        xAxis: {
                            type: 'category',
                            data: json.x
                        },
                        yAxis: {
                            type: 'value',
                            min: 400,
                            max: 1200,
                            interval: 100
                        },
                        grid: {
                            bottom: '10%'
                        },
                        series: [
                            {
                                name: 'GMV',
                                type: 'line',
                                data: json.y,
                                markPoint: {
                                    data: [
                                        {type: 'max', name: '最大值'},
                                        {type: 'min', name: '最小值'}
                                    ]
                                },
                                markLine: {
                                    data: [
                                        {type: 'average', name: '平均值'}
                                    ]
                                }
                            }
                        ]
                    }
                    gmvChart.setOption(gmvOption)
                })

            let channelChart = echarts.init(document.querySelector('#channel'), 'dark')
            fetch('/api/channel_data')
                .then(resp => resp.json())
                .then(json => {
                    let channelOption = {
                        title: {
                            text: '渠道获客占比',
                            top: '5%',
                            left: '5%'
                        },
                        tooltip: {
                            trigger: 'item'
                        },
                        // legend: {
                        //     type: 'scroll',
                        //     orient: 'vertical',
                        //     right: 10,
                        //     top: 20,
                        //     bottom: 20
                        // },
                        series: [
                            {
                                name: '获客渠道',
                                type: 'pie',
                                radius: ['40%', '70%'],
                                center: ['50%', '55%'],
                                avoidLabelOverlap: false,
                                label: {
                                    show: true
                                },
                                // emphasis: {
                                //     label: {
                                //         show: true,
                                //         fontSize: '18',
                                //         fontWeight: 'bold'
                                //     }
                                // },
                                labelLine: {
                                    show: true
                                },
                                data: json.results
                            }
                        ]
                    }
                    channelChart.setOption(channelOption)
                })

            // 将指定的div处理成绘图用的画布
            let barChart = echarts.init(document.querySelector('#bar'), 'dark')
            fetch('/api/sales_data')
                .then(resp => resp.json())
                .then(json => {
                    // 准备图表需要使用到的数据
                    let barOption = {
                        title: {
                            text: '大区销售',
                            top: '5%',
                            left: '5%'
                        },
                        legend: {
                            data:['华东区', '华南区', '华北区', '西南区'],
                            top: '5%',
                            left: '25%',
                        },
                        xAxis: {
                            data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
                        },
                        yAxis: {
                        },
                        grid: {
                            top: '25%',
                            bottom: '10%'
                        },
                        series: [
                            {
                                name: '华东区',
                                type: 'bar',
                                data: json.y1
                            },
                            {
                                name: '华南区',
                                type: 'bar',
                                data: json.y2
                            },
                            {
                                name: '华北区',
                                type: 'bar',
                                data: json.y3
                            },
                            {
                                name: '西南区',
                                type: 'bar',
                                data: json.y4
                            }
                        ]
                    }
                    // 把数据加到图表上
                    barChart.setOption(barOption)
                })

            let klineChart = echarts.init(document.querySelector('#kline'), 'dark')
            fetch('/api/stock_data')
                .then(resp => resp.json())
                .then(json => {
                    let klineOption = {
                        title: {
                            text: '公司股票',
                            top: '5%',
                            left: '5%'
                        },
                        xAxis: {
                            data: json.x
                        },
                        yAxis: {
                            scale: true,
                            splitArea: {
                                show: true
                            }
                        },
                        dataZoom: [
                            {
                                show: true,
                                type: 'slider',
                                top: '90%',
                                start: 0,
                                end: 100
                            }
                        ],
                        series: [
                            {
                                type: 'k',
                                data: json.y
                            }
                        ]
                    }
                    klineChart.setOption(klineOption)
                })
        </script>
    </body>
</html>
```
