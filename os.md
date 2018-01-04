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
