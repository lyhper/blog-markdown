---
title: 经典排序算法之JavaScript实现
categories: Algorithm
date: 2016-04-10
---
包含冒泡、选择、插入、归并、快速排序

```javascript
function ArrayList() {
	var array = [];
	this.insert = function(items) {
		array = items;
	};
	this.toString = function() {
		return array.join();
	};
	
	// 交换两个数的位置
	var swap = function(index1, index2) {
			var temp = array[index1];
			array[index1] = array[index2];
			array[index2] = temp;
		};
	
	// 冒泡排序
	this.bubbleSort = function() {
		var length = array.length;
		for (var i = 0; i < length; i++) {
			for (var j = 0; j < length - 1; j++) {
				if (array[j] > array[j + 1]) {
					swap(j, j + 1);
				}
			}
		}
	};

	// 选择排序
	this.selectionSort = function(){
		var length = array.length;
		var minIndex;

		for(var i = 0; i<length-1;i++){
			// 表示第几小
			minIndex = i;

			for(var j = i; j<length; j++){
				if(array[j] < array[minIndex]){
					minIndex = j;
				}
			}
			if(i !== minIndex){
				swap(i, minIndex);	
			}
		
		}
	};

	// 插入排序
	this.insertionSort = function(){
		var length = array.length;
		var j,temp;
		for(var i = 1;i<length; i++){
			j = i;
			// 待插入数
			temp = array[j];
			// 将待插入数与之前已经插入好位置的数进行比较
			while(j>0 && array[j-1] > temp){
				array[j] = array[j-1];
				j--;
			}
			// 将待插入数插入正确位置
			array[j] = temp;
		}
	};

	// 归并排序，O(nlogn),第二个n是n平方
	// 将原数组递归分解为只有一个元素的数组，再进行有序合并
	this.mergeSort = function(){
		array = mergeSortRec(array);
	};
	// 递归分解
	function mergeSortRec(array){
		var length = array.length;
		if(length === 1){
			return array;
		}
		var mid = Math.floor(length/2);
		var left = array.slice(0, mid);
		var right = array.slice(mid, length);
		return merge(mergeSortRec(left), mergeSortRec(right));
	}
	// 有序合并数组
	function merge(left, right){
		var result = [];
		var il=0,ir = 0;
		while(il<left.length && ir<right.length){
			if(left[il]<right[ir]){
				result.push(left[il++]);
			}else{
				result.push(right[ir++]);
			}
		}
		while(il < left.length){
			result.push(left[il++]);
		}
		while(ir < right.length){
			result.push(right[ir++]);
		}
		return result;
	}
	// 快速排序，O(nlogn),第二个n是n平方
	// 挖坑填数+分治法
	this.quickSort = function(){
		quick(array, 0, array.length - 1);
	}
	// left,right代表左右指针
	function quick(array, left, right){
		var index;
		if(array.length > 1){
			index = partition(array, left, right);
			if(left < index - 1){
				quick(array, left, index - 1);
			}
			if(right > index){
				quick(array, index, right);
			}
		}
	}
	// 将大于基准数的值放到右边，将小于基准数的值放到左边
	function partition(array, left, right){
		var pivot = array[Math.floor((left+right)/2)];
		var i = left, j = right;
		while(i <= j){
			while(array[i] < pivot){
				i++;
			}
			while(array[j] > pivot){
				j--;
			}
			if(i <= j){
				swapQuickSort(array, i, j);
				i++;
				j--;
			}
		}
		return i;
	}
	function swapQuickSort(array, index1, index2){
		var temp = array[index1];
		array[index1] = array[index2];
		array[index2] = temp;
	}
}
```