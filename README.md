# pytrader
封装一些量化交易中需要用到的功能，最初是为了配合wondertrader（wtpy）使用。

功能分为两大类，一类是用于交易盘口数据的可视化，提供盘口交易需要的基本数据；一类是用于计算实盘量化策略信号，以ndarray零拷贝和numba进行加速。

### 目前实现的：
9种成交类型判断：多开、空开、多平、空平、多换、空换、双开、双平、未知。

三种成交方向判断：主买、主卖、中性。

多空净量计算。遵循业内主流算法，目前与文华财经保持一致。

### 待实现：
** 基于形状统计分析的形态相似度检测，用于提供形态学信号。

** 用于实时计算多空与形态信号分发给策略使用，实现策略业务逻辑与信号运算逻辑分离。准备基于于webscoket实现。


# 函数功能：
### 成交类型与方向计算
从tick数据计算成交类型与成交方向等盘口数据。
```
def get_accurate_trade_direction(
        today_ticks: np.ndarray,  # 当日所有tick的结构化数组
        preday_tick: np.ndarray = None,  # 提供昨日tick可以计算竞价方法（但是其实没什么用）
        exchange: str = "default", #  交易所代码
        full_output: bool = True,  # 是否返回完所有字段
        label: bool = False  # 是否将9个成交类型、三个成交方向从数字映射为文本
) -> np.ndarray：#  该接口返回数据完整，可用于盘口可视化，}```
```
```
def get_accurate_direction_only(
        today_ticks: np.ndarray,
        preday_tick: np.ndarray = None,
        exchange: str = "default",
        label: bool = False  
) -> np.ndarray:# 只计算成交方向，返回-1,0,1三种成交方向。用于实时量化策略计算。
```

### 多空计算引擎

从tick数据计算并返回指定周期的多空信号，CPU缓存级优化。支持1s、5s、15s、30s、1min、5min、15min、30min、1h、day、week、month。虽然大周期的没啥用就是了。
一般推荐numba模式，1分钟周期的5万条tick计算平均耗时0.000274s，50万条耗时0.005170s。超大数据量时推荐polars模式。
```
def calculate_net_volume(
        direction_result: np.ndarray,
        freq: str, # 计算周期
        engine: str = "auto",  # "auto", "polars", "numba"三个计算引擎
        engine_threshold: int = 700000  # 阈值参数
) -> np.ndarray:  自动分发引擎
```
可以根据性能测试结果手动选择合适引擎：
![image](https://github.com/user-attachments/assets/0e3227b9-9d3b-4013-b654-c715758dab35)


### Tick级数据重采样
这部分就没啥说的了，组合了下polars的功能。

### 形态分析
基于统计形状分析方法实现的多周期、自定义时窗的形态相似度检测.


