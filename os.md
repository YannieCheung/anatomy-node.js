###os.cpus()
返回一个对象数组，包含了每一个逻辑CPU核心的信息。

数组中每个对象包含以下属性
* model core的基本信息
* speed 速度，单位MHz
* times 在以下各种模式下CPU所花费掉的毫秒数
    - user 
    - nice
    - sys
    - idle
    - irq
其中一个core如下面示例
```
{
    model: 'Intel(R) Core(TM) i7-4600U CPU @ 2.10GHz',
    speed: 2693,
    times:{
        user: 10110175,
        nice: 0,
        sys: 6049484,
        idle: 166312775,
        irq: 50825
    }
}
```