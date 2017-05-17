##有意义的命名
###名副其实
变量，函数或类的命名应该告诉我们：  
1. **它为什么会存在**  
2. **它做什么事**  
3. **应该怎么用**

简洁性&模糊度  
可能3，4行的代码，但是模糊度很高,如此不断的改进  

```
public List<int[]> getThem() {  
	List<int[]> list1 = new ArrayList<int[]>();
	for(int[] x : theList){
		if(x[0] == 4){
			list1.add(x);
		}
	}
	return list1
}

public List<int[]> getFlaggedCells() {  
	List<int[]> flaggedCells = new ArrayList<int[]>();
	for(int[] cell : gameBoard){
		if(cell[STATUS_VALUE] == FLAGGED){
			flaggedCells.add(cell);
		}
	}
	return flaggedCells
}

public List<Cell> getFlaggedCells() {  
	List<Cell> flaggedCells = new ArrayList<Cell>();
	for(Cell cell : gameBoard){
		if(cell.isFlagged()){
			flaggedCells.add(cell);
		}
	}
	return flaggedCells
}
```

###避免误导
accountList，如果容器真的是list类型，没问题，否则，accounts，accountGroup都是更好的选择。

###做有意义的区分
不要废话，`Product`，`ProductInfo`，`ProductData`就是无意义的区分。

###使用读的出来的名称
###使用可搜索的名称
###避免使用编码
1. 匈牙利语标记法
2. 成员前缀：把类和函数做到足够小，消除对于前缀的依赖
3. 接口和实现：IXyzFactory和XyzFactory，I是一个干扰，我也无需让人知道给他们的是个接口，XyzFactory和XyzFactoryImpl就很好

###避免思维映射
###类名
类名，对象名应该是名词或者名词短语

###方法名
应该是动词或动词短语

###别用双关语

###使用解决方案领域名称
毕竟我们面对的都是程序员

###添加有意义的语境

##函数