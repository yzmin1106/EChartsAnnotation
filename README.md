# ECharts的Java注解框架 EChartsAnnotation
  用原生Java注解来映射ECharts的`Option`类，提供Annotation到`JSON`的转换功能。
  
## 思路 How it works
`1`在后台使用Annotation来标注Bean类  
`2`注解处理器转换成JSON树  
`3`使用JSON系列化工具包（fastjson/gson）输出到前端页面  

## 注解 Annotation
基于ECharts3.0制作  
[Option定义](http://echarts.baidu.com/documents/cn/option.json?_v_=1453695515722 "点击下载JSON文件")  
总共有3700+个注解！这里生成的注解只能用于标记Bean类的域`Field`  
根据JSON树的叶子节点的Type属性中的不同类型，3700多注解分成6种不同类型:  

JS类型|Java类型|默认值|文件名后缀|备注
-------|--------|------|----------|----
boolean|boolean|false|Boolean|布尔类型
Color|int|0|Hex|Web颜色映射成Java整型，由于常用16进制表示所以后缀是Hex
number|Number|0|Number|抽象类Number是int、double等基本类型的父类，一律转为double
string|String|""|String|字符串类型
Array|List|无|Array|数组类型一律转成泛型List
Function|Object|无|Function|由于Java不支持函数类型，所以需要实现fastjson接口JSONAware的toJSONString方法输出字符串的“函数”
*|Object|无|All|参考Function类型

按注解的参数个数分，可以分为两种：  
######标记注解
标记注解没有参数。如果被标记的元素为`null`或者等于默认值，注解处理器将不会输出任何东西  
######单值注解
单值注解只有一个参数。该参数的类型跟注解名件名末尾的单词有关。  
由于许多叶子节点有多种类型，所以一个叶子节点可能生成多个注解，但是注解处理器把这些注解当作同一个注解输出。  
在单值注解的参数不为空和被标记的元素不为`null`且不等于默认值的情况下，注解处理器会优先输出被注解元素的值。  
## 注解处理器 AnnotationProcessor
######注解处理器专用的注解：  
`SingleChart`和`DuplexChart`用来标记需要被转换成JSON的Java Bean类，  
这两个注解不同之处在于处理`series`、`dataZoom`和`visualMap`下面的注解时，  
同一个类多次同个注解文件，前者把重复出现的注解合并处理；  
后者通过`AddSeries`、`AddDataZoom`和`AddVisualMap`三种注解提取被`SingleChart`标记的Java文件中的  
`series`、`dataZoom`和`visualmap`,并添加到当前被`DouplexChart`标记的文件中。
######SingleChart
参数 `exportTo` 默认值是"",不等于默认值时向本地磁盘写入"exportTo".json  
参数 `extendsFrom` 默认值是"",不等于默认值时,将继承自"extendsFrom.json"  
######DupelxChart
参数 `exportTo` 默认值是"",不等于默认值时向本地磁盘写入"exportTo".json  
参数 `extendsFrom` 默认值是"",不等于默认值时,将继承自"extendsFrom.json"  
######AddSeries
只能用于`DuplexChart`的域`Field`，若域不为null且域的类源文件被`SingleChart`注解标记，  
将会提取源文件中`series`下的注解并添加到`DuplexChart`  
######AddDataZoom
只能用于`DuplexChart`的域`Field`，若域不为null且域的类源文件被`SingleChart`注解标记，  
将会提取源文件中`dataZoom`下的注解并添加到`DuplexChart`   
######AddVisualMap
只能用于`DuplexChart`的域`Field`，若域不为null且域的类源文件被`SingleChart`注解标记，  
将会提取源文件中`visualMap`下的注解并添加到`DuplexChart`   
## 如何使用 Get Started
`phraseSingleChart`和`phraseDuplexChart`已合并到`parseChart`，不用再区分两者。  
`1`添加EChartsAnnotation到项目  
maven  
```XML
<dependency>
  <groupId>cn.edu.gdut.zaoying</groupId>
  <artifactId>EChartsAnnotation</artifactId>
  <version>1.0.2</version>
  <type>pom</type>
</dependency>
```
gradle  
```Groovy
compile('cn.edu.gdut.zaoying:EChartsAnnotation:1.0.2')
```
jar  
[下载Jar包](https://github.com/zaoying/EChartsAnnotation/blob/master/out/artifacts/EChartsAnnotaion/EChartsAnnotaion.jar "点击下载EChartsAnnotation.jar")  
`2`增加LineChart折线图
```Java
@SingleChart(exportTo = "templates/lineChart.json")
public class LineChart {
    @NameString
    String name;
    @DataArray
    double[] data;
    @cn.edu.gdut.zaoying.Option.xAxis.NameString("横轴")
    String xAxisName;
    @TypeString("category")
    String type;
    @cn.edu.gdut.zaoying.Option.xAxis.DataArray
    int[] xAxisData=new int[]{1,2,3,4};
    @cn.edu.gdut.zaoying.Option.yAxis.NameString("纵轴")
    String yAxisName;
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double[] getData() {
        return data;
    }

    public void setData(double[] data) {
        this.data = data;
    }

}
```
之前的例子因为少了`xAxis`和`yAxis`没法正常显示，这是ECharts的要求。  
`3`调用图表处理器
```Java
public class EChartsTest {
    public static void main(String[] args) {
      LineChart lineChart=new LineChart();
      lineChart.name="线性表一";
      lineChart.data=new double[]{1,2,3,4};
      Object json=EChartsAnnotationProcessor.parseChart(lineChart);
      System.out.print(JSON.toJSONString(json));
    }
}
```
`4`编写更复杂的组合图表  
######编写条形图
```Java
import cn.edu.gdut.zaoying.Option.series.bar.DataArray;
import cn.edu.gdut.zaoying.Option.series.bar.NameString;
import cn.edu.gdut.zaoying.SingleChart;

@SingleChart
public class BarChart {
    @NameString
    String name;
    @DataArray
    double[] data;
}
```
######CombinedChart组合图表
```Java
@DuplexChart
public class CombinedChart {
    @TextString
    String title;
    @BackgroundColorHex(value = 0xfff)
    int backgroundColor;
    @AddSeries
    LineChart lineChart;
    @AddSeries
    LineChart lineChart2;
    @AddSeries
    BarChart barChart;

    public CombinedChart(String title) {
        this.title = title;
        lineChart = new LineChart();
        lineChart.setName("折线图");
        lineChart.setData(new double[]{1,2,3,4});
        lineChart2 = new LineChart();
        lineChart2.setName("折线图二");
        lineChart2.setData(new double[]{3,6,8,9});
        barChart = new BarChart();
        barChart.setName("条形图");
        barChart.setData(new double[]{5,6,7,8});
    }
}
```
`5`调用图表处理器
```Java
public class EChartsTest {
    public static void main(String[] args) {
        Object json=EChartsAnnotationProcessor.parseChart(new CombinedChart("组合图表"));
        System.out.print(JSON.toJSONString(json));
    }
}
```
`6`使用继承
######编写view.json
```javascript
{
  "title":{"text":"组合图表"},
  "backgroundColor":"#fff"
}
```
######修改CombinedChart.java
```Java
@DuplexChart(extendFrom = "view.json")//view.json放在同一个目录，不然要加上完整路径
public class CombinedChart {
//    @TextString
//    String title;
//    @BackgroundColorHex(value = 0xfff)
//    int backgroundColor;
    @AddSeries
    LineChart lineChart;
    @AddSeries
    Line2Chart line2Chart;
    @AddSeries
    BarChart barChart;

    public CombinedChart(String title) {
//        this.title = title;//使用view.json提供的值
        lineChart = new LineChart();
        lineChart.setName("折线图");
        lineChart.setData(new double[]{1,2,3,4});
        line2Chart = new Line2Chart();
        line2Chart.setName("折线图二");
        line2Chart.setData(new double[]{1,2,3,4});
        barChart = new BarChart();
        barChart.setName("条形图");
        barChart.setData(new double[]{5,6,7,8});
    }
}
```
######调用图表处理器
```Java
public class EChartsTest {
    public static void main(String[] args) {
        Object json=EChartsAnnotationProcessor.parseChart(new CombinedChart("组合图表"));
        System.out.print(JSON.toJSONString(json));
    }
}
```
`7`导出json文件  
导出json文件供其它图表`继承`
```Java
@DuplexChart(exportTo = "templates/view.json")//调用图表处理器解析的同时，导出json文件
@SingleChart(exportTo = "templates/view.json")//不建议和DuplexChart注解在同一个类中使用
```
`8`添加Function
```Java
import com.alibaba.fastjson.JSONAware;
public class Function implements JSONAware{//实现JSONAware接口
    String method;
    String arguments;
    String body;
    
    public Function(String method, String arguments, String body) {
        this.method = method;
        this.arguments = arguments;
        this.body = body;
    }
    @Override
    public String toJSONString() {
        return "function "+method+" ("+arguments+"){\n"+body+"\n}";
    }
}
```
`9`测试
```Java
import com.alibaba.fastjson.JSON;

public class ECharts {
    public static void main(String[] args) {
        Function function=new Function("toString","str","alert();");
        System.out.println(JSON.toJSONString(function));
    }
}
```
##写在最后
项目进度已基本完成，后期除了bug fix，不会再有大修改。请放心集成！
`Taglib`项目
  [电梯](https://github.com/zaoying/EChartsTaglib "EChartsTaglib")
