这次的讨论围绕list.h展开，收获很大。<br />
**1、为什么要将`List_head`结构体定义在`types.h`中**  <br />
  我最开始认为是更容易维护，就是把处于一个层级的定义在一起，管理起来更加方便，
当需要淘汰掉某一个或者增加新的东西时更方便，这就类似于代码中的宏一样，一改全改。  <br />
**2、关于使用`const`修饰函数参数**    <br />
  以前编写c程序时，唯一使用到**const**的地方就是预定义一些常量字符串，方便后面使用。而在函数定义时，参数类型的选择依据是**是否要对该变量的值
或其指向的值进行修改**，如果需要修改，则选择传递地址，否则传递变量本身，这种做法的目的是**防止对数据的错误操作**，但带来的负担是，对栈空间
的使用量非常大，而栈空间是有限的，以前也想过这个问题，但一直不知道如果全部传递指针的话，如何避免对数据的错误操作。
今天看了源码，找到了解决这个问题的关键---**const**关键字修饰函数参数。下面这段代码的目的是对一个数组进行排序，并显示结果，但不更改原数据：
2.1当未用**const**修饰时，编译通过，但对原始数据进行了更改:

```
void sort_and_show(int * array, int count) {
    int i, j;
    int numOfNotSorted = count;
    int hasSwap = 1;
    int temp;
    
    for(i = 0; i < count && hasSwap; i++, numOfNotSorted--) {
        hasSwap = 0;
        for(j = 0; j < numOfNotSorted - 1; j++) {
            if(array[j] < array[j + 1]) {
                temp = array[j];
                array[j] = array[j + 1];
                array[j + 1] = temp;
                hasSwap = 1;
            }
        }
    }
    show_array(array, count);
}
```
2.2使用**const**修饰时，编译未通过：
```
void sort_and_show(const int * array, int count) {
    int i, j;
    int numOfNotSorted = count;
    int hasSwap = 1;
    int temp;

    for(i = 0; i < count && hasSwap; i++, numOfNotSorted--) {
        hasSwap = 0;
        for(j = 0; j < numOfNotSorted - 1; j++) {
            if(array[j] < array[j + 1]) {
                temp = array[j];
                array[j] = array[j + 1];//××××××××××××××××××××××××××××××××
                array[j + 1] = temp;
                hasSwap = 1;
            }
        }
    }
    show_array(array, count);
}
```
”×“所在行有错误，错误提示为：
```
error:assignment of read-only location '*(array +(sizetype)((long unsigned int)j * 4ul))'
```
意思非常明确，array指针所指向的空间的数据是read-only，即不允许修改，所以“×”所在行的数据修改操作是非法的，也就说<br />
  `const int * array`  <==> `(const int) *array`
所以以后在编写c语言程序时，传递指针，用**const**修饰来限制操作，需要在源数据的基础上进行操作，但不允许破坏源数据时，就复制一份再操作。
<br />
**3、总结** <br />
  内核源码读起来比较困难，大量的宏，完成了很多习惯性上用函数解决的问题，而且.h文件层层相扣，要不断的去追踪到源头。还有许多其他的收获，
  比如我也一直在纠结编程语言的选择，感觉东西很多学不玩，老师通过list.h带出了“表”这种结构，进而引申到了所有表例如数据库操作等等，启示
  要**抓住最根本的东西，不要贪多**。
