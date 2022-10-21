# For循环

## 增强遍历数组

```
for(声明变量:表达式){
}
```

``` java
public class AddFor {
    public static void main(String[] args) {
        int a[]={2,4,5,7};
        for(int x:a){     //遍历数组，把数组a全部取出来赋值给 int x
            System.out.println(x);
        }
    }
}

```

# 可变参数

``` 
可变参数一个方法只能有一个，也必须是最后一个，任何普通的参数必须在它的之前申明
```

```java
package method;

public class demo04 {
    public static void main(String[] args) {
        demo04 text=new demo04();
//        text.changeMethod(1,2,3,4,5,8);
        text.printMax(1,2,23.1,50.56);
        printMax(new double[]{23.1,12.2,80.56});
    }
//    public void changeMethod(int ... i){ //可变长参数
//        for (int s:i){
//            System.out.println(i[s]);
//        }
//    }
    static void printMax(double ... num){
        if(num.length==0){
            System.out.println("请输入一个合适的数!!");
            return;
        }
        double result=num[0]; //比较
        for (int i = 0; i <num.length ; i++) {
            if(num[i]>num[0]){
                result=num[i];

            }
        }
        System.out.println("最大的数为："+result);
    }
}

```

# 递归

``` java
递归三大点：
1.确定递归函数的参数返回值
2.确定终止条件
3.确定单层递归的逻辑


BST(二分查找树)
前序：中左右
中序：左中右
后序：左右中
//前序
public void preOrder(TreeNode biTree){ 
    Stack <TreeNode> stack= new Stack<TreeNode>();
    while(biTree!= null || !stack.isEmpty()){
        while(node!=null){
            System.out.print(biTree.value+",");//中在前
            stack.push(biTree);
            biTree=biTree.left;
        }
        if(!stack.isEmpty()){
            biTree=stack.pop();
            biTree=biTree.right;
        }
    }
}


//中序
public void preOrder(TreeNode biTree){  
    Stack <TreeNode> stack= new Stack<TreeNode>();
    while(biTree!= null || !stack.isEmpty()){
        while(node!=null){
            stack.push(biTree);
            biTree=biTree.left;
        }
        if(!stack.isEmpty()){
            biTree=stack.pop();
            System.out.print(biTree.value+",");//中在中
            biTree=biTree.right;
        }
    }
}




//后续
public void preOrder(TreeNode biTree){  
    Stack <TreeNode> stack= new Stack<TreeNode>();
    while(biTree!= null || !stack.isEmpty()){
        while(node!=null){
            stack.push(biTree);
            biTree=biTree.left;
        }
        if(!stack.isEmpty()){
            biTree=stack.pop();
            biTree=biTree.right;
            System.out.print(biTree.value+",");//中在后
        }
    }
}
```



```


```

# 数组

## 选择排序

```java
package ArrayDemo;
/*
冒泡排序，比较数组中两个相邻的元素，如果第一个数比第二个数大，就交换它们位置
每一次比较会产生一个最大或者最小的数字
下一轮可以少一次排序，直到循环结束
 */

public class Arraydemo03 {
    public static void main(String[] args) {
        int[]n={2,8,1,5,32,14,8,45,100};
        int []sort=sort(n);//返回我们自己拍完的数组
        for (int a:sort) {
            System.out.print(a+", ");
        }

    }
    public static int[] sort(int []arrys){
        //临时变量
        int tem=0;

        //判断循环次数
        for (int i = 0; i < arrys.length-1; i++) {
            //如果第一个数比第二个数大就交换位置
            for (int j = 0; j < arrys.length-1-i; j++) {
                if(arrys[j+1]<arrys[j]){
                    tem=arrys[j];
                    arrys[j]=arrys[j+1];
                    arrys[j+1]=tem;
                }

            }
        }
        return arrys;
    }
}

```

# 构造器



``` java
注意点：
1.本质是实例化对象
2.命名规则和类名相同
3.定义一个有参构造器之后想用无参使用Alt+Insert快捷键
4.this.xxx=xxx表示前面是当前类的，后面是传进来的

```

# Super&Object



```java
super注意点：
1.super调用父类的构造方法，必须在构造方法的第一个
2.super必须只能出现在子类的方法或者构造方法中
3.super，this不能同时调用构造方法

VS this
1.this本身代表的对象不同
2.super代表父类对象的引用
3.this没有继承也能使用，而super只能在继承条件下使用
4.this默认调用本类构造，而super调用父类构造


object：
1.object是所有类的父类
```

# 方法重写



```java
注意点：
1.重写都是方法的重写和属性无关，必须有继承关系
2.子类重写了父类的方法之和非static有关
3.重写只能是public
4.方法名必须相同，参数列表也是
5.修饰符可以扩大，不能缩小
6.抛出的异常：可以被缩小不能被扩大
总结：重写就是子类和父类必须一致，方法体不同
```

# 多态



```java
注意点：
1.多态是方法的多态，不是属性的多态
2.父类和子类有联系，类型转换异常 ClassCastException
3.存在条件：继承关系，方法要重写，父类引用指向子类对象  father f1=new son();
4.有继承关系
5.instanceof(类型转换)  能不能编译通过关键是看有没有父子关系
6.父类的引用指向对象
7.把子类转为父类，向上转型
8.把父类转为子类，向下转型；强制转换会丢失子类方法
```

# 抽象与接口

```java
注意点：
1.abstract的类代表抽象类
2.不能new这个抽象类，只能靠子类去实现他，并且子类要想继承抽象类就必须实现他的所有方法
3.抽象类里可以写普通方法，抽象方法必须在抽象类中
4.接口主要作用是定义方法让别人来实现的
5.public abstract
6.接口不能被实例化，接口没有构造方法
7.implements可以实现多个接口
```

# 集合

``` java
Java 集合分为3大类
1.Set：无序，不可重复的集合	
	Hashset：按hash算法来存储集合中的元素，不是线程安全，set存的值位置由hashcode决定
	    Set set=new HashSet();//不限制存储类型
        set.add("String");
        set.add(2);
        set.add(false);
        set.add(null);
        System.out.println(set.size());
        System.out.println("=======================");
        Iterator it=set.iterator();  //迭代器功能和foreach一样
        while (it.hasNext()){//迭代器中有下一个
            System.out.print(it.next()+" ");
        }
        //泛型
        Set<Integer> set2=new HashSet<Integer>();//指定存储的数据类型
        set2.add(2);

2.List：有序，可重复的集合
		List list=new ArrayList<>();
        for (int i = 0; i <10 ; i++) {
            list.add(i);
        }
        System.out.println(list.size());
        System.out.println(list);
        list.add(2,1);
        System.out.println(list);
        List list1=new ArrayList<>();
        list1.add(102);
        list1.add(202);
        list1.add(402);
        list.addAll(5,list1);
        System.out.println(list);
        System.out.println(list.get(0));
        System.out.println(list.contains(9));
        System.out.println(list.equals(2)==list.equals(1));
        System.out.println(list.indexOf(1));
        System.out.println(list.lastIndexOf(1));
3.Map：具有映射关系的集合
	   
	   Map map=new HashMap<>();
        map.put(1,"dsdasd");
        map.put(2,44);
        map.put(3,true);
        map.put(4,"q");

        System.out.println(map.get(2));//根据key去对应数值
        System.out.println(map.containsKey(4));//是否包含键
        System.out.println(map.containsValue(44));//包含值
        Set<Map.Entry> entrys=map.entrySet();//遍历
        for (Map.Entry en:entrys) {
            System.out.println(en.getKey()+","+en.getValue());
        }
4.Collections 工具类
	
	    List list=new ArrayList<>();
        for (int i = 0; i <5 ; i++) {
            list.add(i);
        }
        System.out.println(list);
        Collections.reverse(list); //反转
        System.out.println(list);
        System.out.println(Collections.max(list));//找最大
        Collections.swap(list,1,4); //交换1和4位置
        System.out.println(list);
        Collections.replaceAll(list,2,25);//取代2
        System.out.println(list);
```





# 枚举和注解

``` java
public class enumTest {
    public static void main(String[] args) {
        Serson spring = Serson.SPPRING;
        Serson spring1 = Serson.SPPRING;//两个对象都指向同一个类，单例调用
        System.out.println(spring.equals(spring1));
        System.out.println(spring1.name);
        spring.Test();
    }
    enum Serson implements  eumnTest{//枚举
        SPPRING("春天","春心荡漾"),//相当于调用构造方法
        SUMMER("夏天","热"),
        AUTUIN("秋天","秋风刹刹"),
        WINNER("冬天","温暖花开");
        private String name,desc;

        Serson() {//无参
        }

        Serson(String name, String desc) {//有参构造器
            this.name = name;
            this.desc = desc;
        }

        @Override
        public void Test() {//枚举类实现接口方法
            System.out.println("这是枚举类实现接口的Test方法");
        }
    }

}
interface  eumnTest{//接口
    void Test();
}

```

# IO流

## 文件流(文件)

### 文件字节输入输出流

```java
package com.luo.File;

import java.io.FileInputStream;
import java.io.FileOutputStream;

public class FileStream {
    public static void main(String[] args) {
//        FileStream.fileInputStream();
        FileStream.fileOutputStream();
    }
    public static void fileInputStream(){//读
        try {
            FileInputStream fileinput=new FileInputStream("D:\\JavaCode\\NewProject\\src\\com\\luo\\File\\a.txt");
            int len=0;
            byte []b=new byte[10];//设置比特数组接受读取的数据
            while ((len=fileinput.read(b))!=-1){
                System.out.println(new String(b,0,len));//转成字符输出
            }
            fileinput.close();
        }catch (Exception e){

        }


    }
    public static void fileOutputStream(){//写
        try {
            FileOutputStream fileout=new FileOutputStream("D:\\JavaCode\\NewProject\\src\\com\\luo\\File\\b.txt");
            String s="1213465";
            fileout.write(s.getBytes());//缓冲放入内存
            fileout.flush();//执行写入命令
            fileout.close();//关闭字符流
        }catch (Exception e){

        }

    }
}

```



## 缓冲流(内存)



``` java

```



# 两数之和

```java
给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。
你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。
你可以按任意顺序返回答案

class Solution {
    public int[] twoSum(int[] nums, int target) {
        int len=nums.length;
        Map<Integer,Integer> hasmap=new HashMap<Integer,Integer>(len-1);
        hasmap.put(nums[0], 0);
        for(int i=1;i<len;i++){
            if(hasmap.containsKey(target-nums[i])){
                return new int[]{hasmap.get(target-nums[i]),i};

            }
            hasmap.put(nums[i], i);
        }
        return new int[0];
    }
}
```

# 回文数

```java
给你一个整数 x ，如果 x 是一个回文整数，返回 true ；否则，返回 false 。
回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。
    例如，121 是回文，而 123 不是。

class Solution {
    public boolean isPalindrome(int x) {
        int scan=x;
        boolean t=true;
        int a=0;
        int b=0;
        if(scan<0){
//            System.out.print(f);
            t=false;
        }
        while(scan!=0){
            a=scan%10;
            b=b*10+a;
            scan=scan/10;
        }
        if(b==x &&x>=0){
 //           System.out.print(t);
            t=true;
        }else{
 //           System.out.print(f);
            t=false;
        }
        return t;
    }
}
```

# 整数转罗马

```java
字符          数值
I             1
V             5
X             10
L             50
C             100
D             500
M             1000

I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。


class Solution {
    public String intToRoman(int num) {
        int a=0,b=0,c=0,d=0;
        String qianwei="";
        String baiwei="";
        String shiwei="";
        String gewei="";
        if(num>3999||num<0){
            return "";
        }
        if(num>=1&&num<=3999){
            a=num%10; //个
            b=(num/10)%10;//十位
            c=(num/100)%10;//百位
            d=num/1000;//千位
            if(d==0){
                qianwei="";
            }else if(d==1){
                qianwei="M";

            }else if(d==2){
                qianwei="MM";

            }else if(d==3){
                qianwei="MMM";
            }
            switch (c){
                case 0:
                    baiwei="";
                    break;
                case 1:
                    baiwei="C";
                    break;
                case 2:
                    baiwei="CC";
                    break;
                case 3:
                    baiwei="CCC";
                    break;
                case 4:
                    baiwei="CD";
                    break;
                case 5:
                    baiwei="D";
                    break;
                case 6:
                    baiwei="DC";
                    break;
                case 7:
                    baiwei="DCC";
                    break;
                case 8:
                    baiwei="DCCC";
                    break;
                case 9:
                    baiwei="CM";
                    break;
            }
            switch (b){
                case 0:
                    shiwei="";
                    break;
                case 1:
                    shiwei="X";
                    break;
                case 2:
                    shiwei="XX";
                    break;
                case 3:
                    shiwei="XXX";
                    break;
                case 4:
                    shiwei="XL";
                    break;
                case 5:
                    shiwei="L";
                    break;
                case 6:
                    shiwei="LX";
                    break;
                case 7:
                    shiwei="LXX";
                    break;
                case 8:
                    shiwei="LXXX";
                    break;
                case 9:
                    shiwei="XC";
                    break;
            }
            switch (a){
                case 0:
                    gewei="";
                    break;
                case 1:
                    gewei="I";
                    break;
                case 2:
                    gewei="II";
                    break;
                case 3:
                    gewei="III";
                    break;
                case 4:
                    gewei="IV";
                    break;
                case 5:
                    gewei="V";
                    break;
                case 6:
                    gewei="VI";
                    break;
                case 7:
                    gewei="VII";
                    break;
                case 8:
                    gewei="VIII";
                    break;
                case 9:
                    gewei="IX";
                    break;

            }

            }
        return qianwei+baiwei+shiwei+gewei;        
    }
}
```

# 罗马转整数

```java
I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。


class Solution {
    public int romanToInt(String s) {
        int sum = 0;
        char change[] = s.toCharArray();
        for (int i = 0; i < s.length(); i++) {
            switch (change[i]) {
                case 'I':
                    if (i + 1 < s.length()) {
                        switch (change[i + 1]) {
                            case 'V':
                                sum += 4;
                                i++;
                                break;
                            case 'X':
                                sum += 9;
                                i++;
                                break;
                            default:
                                sum += 1;
                        }
                    }else
                        sum=sum+1;
                        break;
                case 'V':
                    sum+=5;
                    break;
                case 'X':
                    if(i+1<s.length()){
                        switch (change[i+1]){
                            case 'L':
                                sum+=40;
                                i++;
                                break;
                            case 'C':
                                sum+=90;
                                i++;
                                break;
                            default:
                                sum+=10;
                        }
                    }else
                        sum+=10;
                        break;
                case 'L':
                    sum+=50;
                    break;
                case 'C':
                    if(i+1<s.length()){
                        switch (change[i+1]){
                            case 'D':
                                sum+=400;
                                i++;
                                break;
                            case 'M':
                                sum+=900;
                                i++;
                                break;
                            default:
                                sum+=100;
                                break;
                        }
                    }else
                        sum+=100;
                        break;
                case 'D':
                    sum+=500;
                    break;
                case 'M':
                    sum+=1000;
                    break;
            }
        }
        return sum; 
    }
}
```

# 最长公共前缀

```java
编写一个函数来查找字符串数组中的最长公共前缀。如果不存在公共前缀，返回空字符串 ""。
示例 1：
输入：strs = ["flower","flow","flight"]
输出："fl"
    
示例 2：
输入：strs = ["dog","racecar","car"]
输出：""
解释：输入不存在公共前缀。

class Solution {
    public String longestCommonPrefix(String[] strs) {
        int n=strs.length;
        String substr=strs[0];
        String s="";
        if(strs==null||strs.length==0){
            return "";
        }
        if(n==1){
            return substr;
        }
        for (int i=1;i<n;i++){
            int sum=0;
            s=strs[i];
            while(sum<substr.length()&&sum<s.length()&& substr.charAt(sum)==s.charAt(sum)){
                sum++;
            
            }
            substr=s.substring(0,sum);
        }
        return substr;   

    }

}
```



# 有效的括号

```java
给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。
有效字符串需满足：
    左括号必须用相同类型的右括号闭合。
    左括号必须以正确的顺序闭合。
    每个右括号都有一个对应的相同类型的左括号。

class Solution {
    public boolean isValid(String s) {
        while(true){
            int n=s.length();
            s=s.replace("()",""); //每次循环必定会削去一对，
            s=s.replace("{}","");
            s=s.replace("[]","");
            if(s.length()==n){
                return n==0;
                }
        }
        
    }
}
```

# 删除有序数组中的重复项

```java
给你一个 升序排列 的数组 nums ，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。元素的 相对顺序 应该保持 一致 。

由于在某些语言中不能改变数组的长度，所以必须将结果放在数组nums的第一部分。更规范地说，如果在删除重复项之后有 k 个元素，那么 nums 的前 k 个元素应该保存最终结果。

将最终结果插入 nums 的前 k 个位置后返回 k 。

class Solution {
    public int removeDuplicates(int[] nums) {
       int numbers =0;
       List list=new ArrayList<>();
       for(int i=0;i<nums.length;i++){
           if(!list.contains(nums[i])){
               list.add(nums[i]);
           }
       }
       Object []newarr=list.toArray();
       Arrays.sort(newarr);
       numbers=newarr.length;
    //    System.out.println(numbers);
       System.out.println(Arrays.toString(newarr));
       return numbers;

}
}


class Solution {
    public int removeDuplicates(int[] nums) {
        if(nums.length<2){
            return nums.length;
        }
        int j=0;
        for(int i=1;i<nums.length;i++){
            if(nums[j]!=nums[i]){
                nums[++j]=nums[i];
            }
        }
        return ++j;
    }
}
```

# 移除元素

```java
给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。


class Solution {
    public int removeElement(int[] nums, int val) {
        int len=0;
        for(int i=0;i<nums.length;i++){
            if(nums[i]!=val){
                nums[len++]=nums[i];
            }
        }
        return len;
}
}
```

# 搜索插入位置（二分查找算法）

```java

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 O(log n) 的算法。

//二分算法
class Solution {
    public int searchInsert(int[] nums, int target) {
        int l = 0, r = nums.length;
        while (l < r) {
            int mid = l + r >> 1;
            if (nums[mid] < target) l = mid + 1;    // 只需找到第一个 >=target 的位置
            else r = mid;
        }
        return l;
    }
}

###################################################################################################


class Solution {
    public int searchInsert(int[] nums, int target) {
        int len=0;
        for(int i=0;i<nums.length;i++){
            if(nums[i]==target) {
                len = i;
                break;
            }else if(nums[i]<=target){
                len=i+1;
            }
        }
        System.out.println(len);
        System.out.println(Arrays.toString(nums));
        return len;
    }
}
```

# 整数转罗马

```java

```

# 整数转罗马

```java

```

# 整数转罗马

```java

```

# 整数转罗马

```java

```

# 整数转罗马

```java

```

# 整数转罗马

```java

```

