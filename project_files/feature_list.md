# rust feature list

- [rust feature list](#rust-feature-list)
  - [rust本身的feature如何实现](#rust本身的feature如何实现)
  - [如何在rust中实现C的feature(rust-for-linux项目)](#如何在rust中实现c的featurerust-for-linux项目)

## rust本身的feature如何实现

1. 如何在结构体中放引用
    - 问题解释：
         - 在rust中如果结构体的成员变量有引用的话，需要对其标注生命周期，这样在多个结构体互相嵌套的情况下，就会导致生命周期标注的工作量爆炸
    - 解决方法
        - 使用rc/arc包裹成员变量；

2. 如何共享全局变量
    - 问题解释
        - 在一个项目中，会经常出现一个模块初始化一个全局的静态变量，希望另一个模块可以使用并修改（如果是只读的就没有问题）；但是rust出于并发安全的考虑，不允许在safe的代码中操作static mut；
    - 解决方法
        - 全局的静态变量声明为static mut，每次使用时采用unsafe包裹；这种方法适用于实际使用过程中并不会出现多线程并发的情况；
        - 采用锁或者原子变量，这种方法适用于会发生多线程并发的情况；
            - 如果这个变量的类型可以采用原子类型，那就采用原子类型；
            - 采用锁包裹，以达到同步的目的，但是此时会遇到const function的问题，需要采用lazy_static，而rust-for-linux里面又没有；
              - 好消息是此时可以采用core中的synclazy函数，坏消息是synclazy在rust-for-linux也没有；

## 如何在rust中实现C的feature(rust-for-linux项目)

1. 如何在rust中实现for_each_online_cpu的功能
    - 问题解释：
        - 一般的宏都可以直接通过helpers.c里面的函数进行包裹，以被rust函数调用，但是for_each_online函数比较特殊，里面是一个for循环，很难在不改变逻辑的情况下通过helpers.c包裹；
    - 解决方法
        - 目前在rust-for-linux的官方论坛上求助，在不改变逻辑的情况下是无法直接包裹的；
        - 所以决定直接改变这个宏的逻辑，返回一个数组，成员是online的cpu index；
        - 官方论坛上还提到的方法有在rust侧重写整个for_each_online_cpu逻辑，或者采用callback函数实现（整个方法目前不太明确）；
