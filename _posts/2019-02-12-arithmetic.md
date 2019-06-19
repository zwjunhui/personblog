---
layout: post
title: 一些简单算法
date: 2019-02-12
tag: arithmetic
---

### 冒泡算法
**简单介绍：**
    重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。
```java

      // 传递参数为一个数组int[] sourceArray，排列为从小到大的顺序
         // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);
 
        for (int i = 1; i < arr.length; i++) {
            // 设定一个标记，若为true，则表示此次循环没有进行交换，也就是待排序列已经有序，排序已经完成。
            boolean flag = true;
 
            for (int j = 0; j < arr.length - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    int tmp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = tmp;
 
                    flag = false;
                }
            }
 
            if (flag) {
                break;
            }
        }
        return arr;

```

### 斐波那契数列
**简单介绍：**
    斐波那契数列，又称黄金分割数列、因数学家列昂纳多·斐波那契以兔子繁殖为例子而引入，故又称为“兔子数列”，指的是这样一个数列：1、1、2、3、5、8、13、21、34、……在数学上，斐波纳契数列以如下被以递推的方法定义：F(1)=1，F(2)=1, F(n)=F(n-1)+F(n-2)（n>=3，n∈N*）
    第一个月初有一对刚诞生的兔子
    第二个月之后（第三个月初）它们可以生育
    每月每对可生育的兔子会诞生下一对新兔子
    兔子永不死去
    ![图片](/images/posts/arithmetic/1560925667.jpg)

```java
    public class Test {
    public static long[] generateFibonaccis(int n) {
        long[] fib = new long[n];
        fib[0] = 1;
        fib[1] = 1;
        for (int i = 2; i < n; i++) {
            fib[i] = fib[i - 2] + fib[i - 1];
        }
        return fib;
    }
    public static void main(String[] args) {
        int n = 10;
        long[] fib = generateFibonaccis(n);
        for (int i = 0; i < n; i++) {
            System.out.print(Long.toUnsignedString(fib[i]) + " ");
        }
    }
}
```


### 二分查找
**简单介绍**
    二分搜索只对有序数组有效。二分搜索先比较数组中比特素和目标值。如果目标值与中比特素相等，则返回其在数组中的位置；如果目标值小于中比特素，则搜索继续在前半部分的数组中进行。如果目标值大于中比特素，则搜索继续在数组上部分进行。由此，算法每次排除掉至少一半的待查数组。

```java
public static int binarySearch(int[] arr, int start, int end, int hkey){
    //递归方式
    if (start > end)
        return -1;

    int mid = start + (end - start)/2;    //防止溢位
    if (arr[mid] > hkey)
        return binarySearch(arr, start, mid - 1, hkey);
    if (arr[mid] < hkey)
        return binarySearch(arr, mid + 1, end, hkey);
    return mid;  

    //while循环方式
    int result = -1;

    while (start <= end){
        int mid = start + (end - start)/2;    //防止溢位
        if (arr[mid] > hkey)
            end = mid - 1;
        else if (arr[mid] < hkey)
            start = mid + 1;
        else {
            result = mid ;  
            break;
        }
    }

    return result;

}
```

### 快速排序
**简单介绍：**
通过一趟排序将要排序的数据分割为两部分，第一部分所有数据比第二部分的所有数据小，按照这种思路将两部分数据再次分别进行快速排序，可以使用递归完成，最终使得整个数据序列有序。
1、选取一个基准数，一般选第0个元素。
2、将比基准数小的移动到左侧，比基准数大的移动到右侧，相等的不移动，此时基准数位置为K。
3、对左右两侧重复步骤1和步骤2，直到左右侧细分到只有一个元素。
**快排的难点也就是在第2步，怎么移动各个数据？**
<1> 首先从数列的右边开始往左找，设下标为i，也就是i--操作，找到第一个比基准数小的值，让他与基准数交换；
<2> 接着开始从左往右找，设下标为j，也就是j++，找到第一个比基准数大的值，让他与基准数交换位置；
<3> 重复1和2，直到i和j相遇时结束，最后基准值所在位置为k。

```java
public class QuickSort {
    private int[] array;
    public QuickSort(int[] array){
        this.array = array;
    }
    
    public void printSort(){
        for (int i = 0; i < array.length; i++) {
            System.out.println(array[i]);
        }
    }
    
    public void sort(){
        quicksort(array,0,array.length -1);
    }
    
    private void quicksort(int[] array,int begin,int end){
        if(begin<end){  //i和j没相遇之前比较各数据与基准值大小
            int base = array[begin];  //取第一个值为基准值
            int i = begin;  //左标记为i
            int j = end;    //右标记为j
            
            //一趟排序，找到比基准值大的在基准值右，比基准值小的在基准值左
            while(i<j){
                //从右往左扫描
                while(i<j && array[j]>base){ //从右往左扫，如果元素比基准值大
                    j--;  //则右边标记--，直到找到第一个比基准值小的，停止扫描
                }
                if(i<j){
                    array[i]=array[j];  //交换右扫描第一个比基准值小的数
                    i++;  //i标记右移一位
                }
                //从左往右扫描
                while(i<j && array[i]<base){//从左往右扫，如果元素比基准值小
                    i++;  //则左标记++，直到找到第一个比基准值大的，停止扫描
                }
                if(i<j){
                    array[j]=array[i];  //交换左扫描第一个比基准值大的数
                    j--;  //j标记左移一位
                }
            }  //此时基准值左右两侧相对有序
            
            array[i] = base;  //此时i为中间位置k
            
            quicksort(array,begin,i-1);  //左侧按照快排思路，递归
            
            quicksort(array,i+1,end);    //右侧按照快排思路，递归
        }
    }    
}
```