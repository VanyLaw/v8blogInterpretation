# [Elements kinds in V8](https://v8.dev/blog/elements-kinds) #

虽然js不会对数组中元素进行类型的约束，但用C++实现底层的时候会根据元素类型对数组进行分类。如
```
const array = [1, 2, 3];
// elements kind: PACKED_SMI_ELEMENTS
// 一开始数组只有小整型，C++使用SMI数组进行保存

array.push(4.56);
// 加入浮点数之后，将SMI数组转为double数组
// elements kind: PACKED_DOUBLE_ELEMENTS

array.push('x');
// 加入string之后，数组变为PACKED数组
```
**elements数组只能从特殊类型转为一般类型，不能由一般转回特殊类型。**
```
// 长度为5的数组在位置10添加一个数，将数组转为holey array（稀疏数组）
array[9] = 1;
console.log(array[8]);

// 在获取元素时，会先进行bounds check。8 >= 0 && 8 < array.length
// 然后调用hasOwnProperty(array, '8');
// hasOwnProperty(Array.prototype或array.__proto__, '8');
// hasOwnProperty(Object.prototype, '8');
```
**但对于一个Packed数组，V8不会按原型链往上查找，会直接返回。只有holey数组并本身没有该属性时，才会搜索原型链。**

```
const array = new Array(3);
// HOLEY_SMI_ELEMENTS

array[0] = 'a';
// 不是小整型，转为HOLEY_ELEMENTS

array[1] = 'b';
array[2] = 'c';
// 仍然是HOLEY_ELEMENTS
```
如果需要对某数组进行大量操作，最好是使用PACKED array。使用已知的初值去初始化数组，而不要用new的方式。

注意js中有2个0，+0和-0，+0是小整型，-0是double。同理NaN和Infinity都是double，不建议添加到数组中。
```
[1,2,+0];
// PACKED_SMI_ELEMENTS
[1,2,-0];
// PACKED_DOUBLE_ELEMENTS
```

如果数组中都是数字，那可以使用更具体的type array，如Int32 array等。

## new Array(n) ##
使用new方法创建大数组的时候，能预先分配出数组足够的空间。优化了数组的创建。
但同时这样会创建一个holey array，会减慢数组上的操作（Array prototype中的方法）。
V8会为数组预留16个元素位置作为buffer，当数组在不断增加的时候超过原预留大小，需要进行reallocate，将现有的数据重新拷贝到新的地址，并分配16个元素的buffer。
