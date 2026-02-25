```cpp
#include <iostream>
#include <chrono>
#include <thread>
#include <vector>
#include <algorithm>
#include <numeric>
#include <cstdlib>   //for"atof"

using namespace std;

class timer{
private:
   double interval;

public:
   timer (double x):interval(x){}
   //打印时间戳：1.获取当前时间 2.转换成时间长度 3.转换成时间戳 4.打印

   static void print_timestamp(){       //调用不需要依赖于任何实例对象
        auto now=chrono::system_clock::now();//这里的std::chrono是命名空间，system_clock是类,now是静态成员函数，属于类本身，不需要对象就可以调用
        //auto自动推断类型，相当于chrono::system_clock::time_point,依次是命名空间，顶层类，嵌套类型
        //::是作用域解析运算符, .是成员访问运算符
```
[::与.的功能对比](https://github.com/xrainy677/first_markdown_class/blob/main/symbol_meanings.md)
```cpp
        auto duration_since_epoch=now.time_since_epoch();  //时间长度的auto对应seconds,milliseconds......
        long long timestamp=chrono::duration_cast<chrono::milliseconds>(duration_since_epoch).count();
        /*1.模板————“函数的函数”，比如duration_cast<chrono::milliseconds>就是生成将时间长度转换为毫秒级时间戳的函数，换一个模板参数就可能生成转换成秒级、微秒级时间戳的函数
          再比如vector<int>,就是指定<int>生成“存int的数组类”
          所以duration_cast<chrono::milliseconds>生成了一个函数，后面的(duration)是函数参数
          2.系统默认的时间长度是以纳秒这种极小单位为单位的，经过一系列操作以后，变成以毫秒为单位的时间长度，但不改变其是时间长度的本质，因而可以调用count*/
        cout<<"当前时间戳为（以毫秒为单位）："<<timestamp<<endl;
   }

   double get_interval() const {
        return interval;
   }

   static chrono::high_resolution_clock::time_point get_concrete_time(){   //命名空间::类（高精度到纳秒级）::数据类型（类似于int） 函数名
         return chrono::high_resolution_clock::now();
   }

   static double get_concrete_duration(const chrono::high_resolution_clock::time_point& start,const chrono::high_resolution_clock::time_point& end){
           auto concrete_duration=chrono::duration_cast<chrono::nanoseconds>(end-start);
           return concrete_duration.count()/1000000000.0;  //不直接转换成seconds,否则直接截断到秒，精度下降
   }
};

double get_percentiles(const vector<double>& v,double p)
{
    if(p<0.0||p>100.0)
    {
        cerr<<"输入错误百分位数"<<endl;
        return -1;
    }
    else if(p<100.0){
    double order_number=p/100.0*(v.size()-1);      //-1对应vector中的下标,一定要除100.0，否则会保留到整数位！！！
    size_t integer=static_cast<size_t>(order_number);      //size_t是非负，范围比unsigned int大
                                                           //static_cast是显示类型转换运算符，语法：static_cast<目标类型>(待转换的变量/值)
    double fractional_part=order_number-integer;

    double target=v[integer]+(v[integer+1]-v[integer])*fractional_part;  //出现integer+1,考虑越界问题
    return target;
    }
    else{
        return v[v.size()-1];
    }
}

double get_average(const vector <double>&v){
    double sum=accumulate(v.begin(),v.end(),0.0);      //begin()返回指向v中首元素的迭代器，end()返回指向容器最后一个元素的下一个位置的迭代器，0.0是初始和，不写0是为了保证结果是double类
    return  sum/v.size();
}


int main(int argc,char* argv[])  //命令行（人和计算机对话的文本界面），argc是命令行参数的总个数（包含程序名），argv[]存储命令行参数的字符串数组
{
    if(argc!=2){
       cerr<<"输入示例：./timer 1.0(1.0为打印时间戳间隔时间）"<<endl;
       return 1;
    }

    double my_interval=atof(argv[1]);   //argv[0]是"./timer",argv[1]是"1.0"(char类型)

    if(my_interval<=0){
      cerr<<"interval必须大于零！"<<endl;
      return 1;
    }

    timer T1(my_interval);
    vector<double> actual_duration;

    cout<<"第1次打印：";
    timer::print_timestamp();
    for(int i=1;i<101;i++)
    {
        auto start=timer::get_concrete_time();
        this_thread::sleep_for(chrono::duration<double>(T1.get_interval()));     
        cout<<"第"<<i+1<<"次打印：";
        timer::print_timestamp();
        auto end=timer::get_concrete_time();       //保证统计的是实际打印间隔时间而不是休眠时间
        double current_duration=timer::get_concrete_duration(start,end);
        actual_duration.push_back(current_duration);
    }
    sort(actual_duration.begin(),actual_duration.end());
        //this_thread也是一个命名空间，表示当前线程
        //duration是一个模板，此处也可以写成sleep_for(chrono::seconds(1)),但这种方式只能表示整数，不能表示1.5之类的
        //这个模板默认单位是秒
        /*duration 模板的完整定义是：template <typename Rep, typename Period = ratio<1>>
                                     class duration;
Rep：数值存储类型（比如 int/double）；Period：时间单位（默认是 ratio<1>，即「1 秒」）。
所以 duration<double>(1.0) 等价于：
duration<double, ratio<1>>(1.0)
ratio<1>：表示 “1 个基本单位 = 1 秒”；
如果你想定义毫秒，需要手动指定 ratio<1,1000>（1/1000 秒 = 1 毫秒）：
duration<double, ratio<1,1000>>(500.0); // 表示500毫秒（0.5秒）*/

    cout<<"平均间隔时间："<<get_average(actual_duration)<<endl;
    cout<<"五十百分位数："<<get_percentiles(actual_duration,50)<<endl;
    cout<<"八十百分位数："<<get_percentiles(actual_duration,80)<<endl;
    cout<<"九十百分位数："<<get_percentiles(actual_duration,90)<<endl;
    cout<<"九十五百分位数："<<get_percentiles(actual_duration,95)<<endl;
    cout<<"九十九百分位数："<<get_percentiles(actual_duration,99)<<endl;
    return 0;
}
```
