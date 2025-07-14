# C++项目开发经验总结

------------

## 开始之前

1. 首先要把项目需求**搞明白**。这个”搞明白“是真的要搞明白，不是简单了解下需要做什么，大致需要分为几个模块就可以了。而是需要进行深入的分析。

2. 任务划分要清晰，各个部分需要完成的功能也要协调清楚，各个模块之间的关系是什么？相互之间的接口如何定义？模块之间的通信是通过函数实参还是共享变量？





## 函数 or 接口的编写

- 编写函数时，需要明白每个函数需要做什么事。要明确哪些事该在这个函数里面做，哪些事该在另一个函数/单独写一个函数来做。函数命名很大程度上就指明了函数的主要任务。

- 一个优雅的接口函数不应该有冗长的参数列表，可以设置一组函数，把参数藏在函数名中。

- 一些比较复杂的难以阅读的判定操作，可以把它们写成内联函数，同时给一些易于理解的名字
  
  ```cpp
      bool currentlyAtBerth()const{if(status_ == 1 && berth_id_ != -1) return true; return false;}
      bool currentlyAtVspot()const{if(status_ == 1 && berth_id_ == -1) return true; return false;}
      bool headingToVspot()const{if(status_ == 0 && berth_id_ == -1) return true; return false;}
      Berth* headingToBerth()const{if(status_ == 0 && berth_id_ != -1) return berth_; return nullptr;}
  ```

## 注意的问题

#### 注释代码时要注意不要干蠢事

这样注释代码只能祝你debug的时候不要冒火

```cpp
    if(count_added_goods > 0)
        //log_ << "+++++ added " <<  goods_num << " new goods" << std::endl;
```


