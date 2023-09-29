# 基础语法
* lambda 表达式
<br>parameters -> expression
<br>或 
<br>parameters -> { statements; }

* 方法引用 类名::方法名
# 一、流的定义
* 操作数据源(集合、数组等)所生成的元素序列
* stream会返回一个新stream
* stream操作时延迟执行的,当需要结果时才执行
# 二、筛选与切片
* .filter(lambda 表达式,或方法引用) 从流中排除元素
* .limit(n) 限制获取n个元素
* .skip(n) 跳过前n个元素
* 类对象.distinct() 去重,但该类必须重写equals()方法与hashCode() 方法
# 三、映射
* .map(lambda 表达式,或方法引用)
* .flatMap(lambda 表达式,或方法引用)
<br>扁平化处理,将Stream\<List>转换为Stream\<T>

# 四、collect
* .collect(Collectors.方法)
<br>这里的方法包括
<br>Collectors.toList();//ArrayList
<br>Collectors.toSet();//HashSet
<br>Collectors.toCollection(所需集合类型::new);
<br>Collectors.toMap(Function1, Function2, mergeFunction);//Function1的结果为key,Function2的结果为value,mergeFunction有重复值时解决冲突的方法
<br>Collectors.partitioningBy(分组条件)//key为true和false;value为满足和不满足条件的数据列表
<br>Collectors.groupingBy(类::get方法)//key为get方法获取的属性,value为类
<br>Collectors.joining(字符串);//将stream中的元素用特定的连接符拼接

# 五、reduce约束
* .reduce(lambda 表达式)//可以实现求和、求最大值、求最小值
# 六、排序
* .sorted()
# 七、计算两个list中的差集
* 集合1.stream().filter(it -> !集合2.contains(it)).collect(Collectors.toList());