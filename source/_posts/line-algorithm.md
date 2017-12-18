---
title: 线性选择第k小元素
---
# 线性选择第k小元素

## 1、	问题描述
给定的线性集中n个元素和一个正整数k(1≤k≤n)，要求在线性时间内（即时间复杂度为O(n)）找出这n个元素中第k小的元素。
## 2、	算法设计思想
可以使用分治策略解决此问题。
算法思想如下：该算法是借助快速排序的划分思想，首先将n个数分为5个一组，分解成了(n + 5)/5个找中位数的子问题。再找出中位数的中位数，将该元素作为基准，划分原数组，可将其划分成两边至少有3*(n - 5)/10 个元素比中位数的中位数小，当n >= 75的时候，3*(n - 5)/10 >= n/4 ，所以划分后得到的数组都至少缩短1/4。再继续在缩短的数组里面找出原问题要找的第k小元素，若基准p > k，则在前面的数组中找第k小元素，否则则在后面的数组找第k – p小的元素。最终得到要找元素的位置，得到值。
## 3、	算法过程描述
以
```
[176 134 177 183 185 ||180 124 144 153 137 ||172 117 187 171 163 ||129 134 116 197 137 ||106 173 145 156 104 ||142 178 127 171 188 ||152 138 177 155 117 ||109 119 119 118 157 ||173 111 164 145 165 ||129 192 140 151 185 ||117 192 152 155 155 ||123 139 135 148 149 ||134 152 127 108 198 ||173 118 163 196 179 ||113 197 125 116 115 ||158 168 145 105 156 ||124 188 127 104 120 ||195 106 162 145 187 ||193 180 141 188 142 ||189 170 178 102 181]
```

这100个数为例。**找出第五小元素。**

(1)	划分成20个组，双竖线为一个组，找出对应的所有组的中位数

`
[177 144 171 134 145 171 152 119 164 151 155 139 134 173 116 156 124 162 180 178]
`

(2)	将他们放在数组的前n/5位，再找出他们中位数的中位数为151。以这个数为基准进行划分。划分结果为
***
[117 116 119 124 134 134 139 144 145 151 102 137 142 141 145 106 127 129 120 104 116 106 145 105 125 127 142 124 115 113 117 138 137 118 134 109 118 127 119 108 111 145 149 148 135 129 140 123 117 **151** 192 152 185 155 192 180 173 163 165 153 157 176 177 152 198 155 163 172 179 196 188 178 187 173 197 156 185 180 158 168 178 177 173 171 188 171 164 183 187 195 162 156 155 188 193 152 170 197 181]
***
灰色元素为基准元素。

(3)	划分后基准元素在49的位置，因为5<49,所以在前半部分查找。

(4)	因为75 > 49所以直接进行快速排序得到最后结果第五小的元素为
## 4、算法实现及运行结果

```
#include <iostream>
using namespace std;

void swapNum(int num[],int i,int j){
	int temp = num[i];
	num[i] = num[j];
	num[j] = temp;
}

//以pivot为标兵元素(pivot在start位置上)，对num数组进行划分，返回pivot的位置
int partition(int num[],int start,int end,int pivot){
	int i = start;
	int j = end;
	while(true){
		while(num[i] <= pivot && i < end){
			++i;
		}
		while(num[j] > pivot){
			--j;
		}
		if(i >= j){
			break;
		}
		swapNum(num,i,j);
	}
	num[start] = num[j];
	num[j] = pivot;
	return j;
}
//快速排序
void qsort(int num[],int start,int end){
	//选取基准元素，小的放基准元素左边，大的放右边
	if(start < end){
		int pivot = partition(num,start,end,num[start]);
		//递归以基准元素位置将数组分成 [start,start+j] 和[j + 1,end],j为基准元素最后所在位置
		qsort(num,start,pivot - 1);//左半部分进行递归
		qsort(num,pivot + 1,end);//右半部分进行递归
	}
}


//找出num数组中第k小的元素，返回在数组中的位置
int select(int num[],int start,int end,int k){
	if(end - start + 1< 75){
		qsort(num,start,end);
		return start + k - 1;
	}
	int groupNum =(end - start + 5) / 5;//算出总共有5个元素一组总共有几个组
	for(int i = 0;i < groupNum;i++){
		//分出来5个元素一组找出中位数
		int left = start + (i * 5);
		int right = (start + (i * 5) + 4) > end ? end : start + i * 5 + 4;
		int midIndex = select(num, left,right , (right - left)/2 + 1);
//		qsort(num,left,right);
		swapNum(num,start + i,midIndex);//将中位数放在前n/5位上
	}
	cout<< "所有中位数为: ";
	for(int i = 0;i < start + groupNum;i++){
		cout<<num[i] << " ";
	}
	cout<<endl;
	//找出中位数的中位数作为标兵元素
	int pivot = select(num, start, start + groupNum, (groupNum + 1)/ 2 );
	//划分后分解成子问题，缩小规模。
	int p = partition(num, start, end, num[pivot]);
	if(p == k){
		return p;
	}
	else if(p > k){//在前半部分找
		return select(num, start, p - 1, k);
	}
	else{//在后半部分找
		return select(num, p + 1, end, k-p);	
	}
}
int main() {
	int n;
	
	cin >>  n;
	int num[n];
	int k = 0;
	int max = 0;
	int min = 100;
	for(int i = 0 ; i < n ; i++){
//		cin >> num[i];
		num[i] = rand()%(max - min + 1) + min;//生成范围[min,max]的随机数
	} 
	cout << "输入数据为：" << endl;
	for(int i = 0 ; i < n ; i++){
		cout << num[i] << " ";
	} 
	
	cout << "输入要找出第几小的元素";
	cin >> k;
	int index = select(num,0,n - 1,k);
	int result = num[index];
	cout << "元素值为" << result << endl;
	
	
	qsort(num,0,n - 1);//排序后可以直接验证第k小元素是否正确
	int flag = num[k - 1] == result ? true : false;
	if(flag){
		cout << "查找结果正确" << endl;
	}
	else{
		cout << "查找结果错误" << endl;
	}	
}
```

运行结果如下:
![](media/15073801079025/15073836975284.jpg)

测试数据随机生成可自行进行测试

## 5、算法复杂度分析及算法改进
（1）	时间复杂度
当n < 75的时候直接进行排序，得到第k位的数即为结果，当n >= 75时，将问题划分成为n/5个找中位数并找出中位数的中位数和在划分后的数组中找出第k小的数两个子问题，所有划分找中位数的时间复杂度为Cn，所以时间复杂度为：
![](media/15073801079025/15073836664832.jpg)

解递归公式可得时间复杂度为T(n) = O(n)

（2）空间复杂度
算法都是在原数组进行操作。无需辅助空间。

（3）算法改进
除了每组分为5个和小于75进行排序，有没有更好的其他划分选择能让规模变的更小？


**如有勘误留言在评论进行修改。**

