# rust feature list

- [rust feature list](#rust-feature-list)
  - [rust本身的feature如何实现](#rust本身的feature如何实现)
  - [如何在rust中实现C的feature(rust-for-linux项目)](#如何在rust中实现c的featurerust-for-linux项目)
  - [rust-for-linux对linux常用功能的实现情况](#rust-for-linux对linux常用功能的实现情况)

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
         - 所以rust-for-linux实现了"kernel::init_static_sync!"这个宏，但是需要迁移到最新的rust-for-linux上才可以；

3. 复杂结构体的初始化

   - 问题解释
     - 结构体成员较多的情况下，直接写一个结构体就显得比较臃肿，需要写大量的代码；如何优雅写一个初始化函数，减轻实例化时的痛苦就成了关键问题；
   - 解决方法
     - 用关联函数new初始化结构体，可以让大家少写一部分初始化需求一样的成员；
       - 这个方法的问题，每一次初始化一个实例时，都需要指定所有的需要初始化的成员变量，代码太长而且参数记不住，如果可以让初始化有默认参数就更好了；
     - 用build pattern解决
       - 这个方法可以有任意长度的默认参数，但是实在是太代码臃肿了；
     - 用default方法
       - 这个方法目前是效果最好的;
   - 参考链接
     - [结构体初始化的三种方法](https://zhuanlan.zhihu.com/p/112202164)
     - [default文档](https://doc.rust-lang.org/std/default/trait.Default.html)
     - [default的几个例子](https://rust-unofficial.github.io/patterns/idioms/ctor.html)

4. Rc<RefCell<sth>>的使用

   - 问题解释

     - 1.用Rc以及RefCell包裹一个结构体作为结构体指针，如何通过这些智能指针来修改结构体成员；
     - 2.如何在修改成员之后再次使用该结构体？

   - 解决方法

     - 1.需要通过borrow_mut方法获取一个可变引用再进行修改；

       ```rust
       let a = s{
           v:3,
       }; // 结构体s有一个类型为i32的成员v
       let p1 = Rc::new(RefCell::new(a));
       let mut p2 = p1.borrow_mut();
       p2.v = 5;
       ```

       

     - 2.由于不能同时存在可变引用和不可变引用（虽然RefCell能保证编译通过，但仍会在运行时panic），所以需要在修改完成后，手动将可变引用释放；

       ```rust
       let a = s{
           v:3,
       }; // 结构体s有一个类型为i32的成员v
       let p1 = Rc::new(RefCell::new(a));
       let mut p2 = p1.borrow_mut();
       p2.v = 5;
       drop(p2); //没有这句则无法再使用p1.clone()或p1.borrow()
       //下面再使用p1
       ```

       

## 如何在rust中实现C的feature(rust-for-linux项目)

1. 如何在rust中实现for_each_online_cpu的功能

   - 问题解释：
     - 一般的宏都可以直接通过helpers.c里面的函数进行包裹，以被rust函数调用，但是for_each_online函数比较特殊，里面是一个for循环，很难在不改变逻辑的情况下通过helpers.c包裹；
   - 解决方法
     - 目前在rust-for-linux的官方论坛上求助，在不改变逻辑的情况下是无法直接包裹的；
     - 所以决定直接改变这个宏的逻辑，返回一个数组，成员是online的cpu index；
       - 目前根据论坛上的意见，我用迭代器实现了这个宏，相关代码放在了cpumask.rs里面；
     - 官方论坛上还提到的方法有在rust侧重写整个for_each_online_cpu逻辑，或者采用callback函数实现（整个方法目前不太明确）；

2. C中的结构体位域如何在rust中实现

   - 问题解释
     - rust中没有方法给struct指定字段长度，现有的方法都比较麻烦；
   - 解决方法
     - 暂时停滞；

3. C中的union如何在rust中实现

   - 问题解释

     - rust中是有union的，但是不太好用，需要所有的成员类型全部实现了copy的trait，对于复杂结构的union时，rust就不太好实现；

   - 解决方法

     - rust中的enum可以基本实现union的内容，给enum内的元素套一个壳子，内部是想要实现的内容；

       ```rust
       enum EvlMonitorState_item {
           gate(EvlMonitorState_item_gate),
           event(EvlMonitorState_item_event),
       }
       ```

4. 结构体指针成员

   - 问题解释
     - 结构体成员是指针类型，c的裸指针在rust中不太安全；
   - 解决方法
     - 根据需求用智能指针加锁包裹起来；
     - 不需要多线程时，可使用Rc<RefCell<sth>>来进行包裹；
     - 需要多线程时，可使用Arc来代替Rc；

5. 函数指针成员

   - 问题解释
     - 直接使用fn(sth)->sth作为函数指针会遇到无法初始化的问题；
   - 解决方法
     - 使用Option包裹一层，在初始化时使用None；

## rust-for-linux对linux常用功能的实现情况

1. linux中的常用函数和宏

   | 函数/宏               | 实现情况                                 |
   | --------------------- | ---------------------------------------- |
   | for_each_possible_cpu | kernel/cpumask.rs::PossibleCpusIndexIter |
   | for_each_online_cpu   | kernel/cpumask.rs::OnlineCpusIndexIter   |
   | for_each_present_cpu  | kernel/cpumask.rs::PresentCpusIndexIter  |
   | per_cpu               | 没有实现                                 |
   | per_cpu_ptr           | 没有实现                                 |

2. linux中的模块功能

   | 模块        | 函数/宏       | 实现情况                         |
   | ----------- | ------------- | -------------------------------- |
   | cpumask相关 |               |                                  |
   |             | cpulist_parse | kernel/cpumask.rs::cpulist_parse |
   |             | cpumask_copy  | kernel/cpumask.rs::cpumask_copy  |
   | 内存相关    |               |                                  |
   |             | vmalloc       | kernel/vmalloc.rs::c_vmalloc     |
   |             | vfree         | kernel/vmalloc.rs::c_vfree       |
   |             | kzalloc       | kernel/vmalloc.rs::c_kzalloc     |
   |             | kfree         | kernel/vmalloc.rs::c_kzfree      |

3. Dovetail相关函数和宏

   | 函数/宏        | 实现情况                          |
   | -------------- | --------------------------------- |
   | running_inband | kernel/premmpt.rs::running_inband |
   | PAGE_ALIGN     | kernel/mm.rs::page_aligned        |

