---
title: 剑指几道算法题的思考
date: 2018-09-03 14:14:57
tags: 剑指几道算法题的思考
---

## 1、在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
<!--  more  -->
* 假如二维数组为 `{ {1, 2, 8, 9}, {2, 4, 9, 12}, {4, 7, 10, 13}, {6, 8, 11, 15} };`同时查找的值为`15`。
<!--  more  -->
  *  常规的思路为我们取到每个二维数组的值，然后遍历每个值，判断是否相等！同时我们知道会循环`16`次。代码如下
      ```
        private boolean lookUpInTwoDimensionalArrays(int[][] a, int num) {
        // 至少保证 长度至少为1，且还有元素，
        if (a == null || a.length < 1 || a[0].length < 1) {
            return false;
        }
        int b = 0;
        for (int i = 0; i < a.length; i++) {
            for (int j = 0; j < a[i].length; j++) {
                b++;
                if (num == a[i][j]) {
                    // 一共循环多少次：12
                    System.out.println(TAG + "一共循环多少次：" + b);
                    return true;
                }
            }
        }
        return false;
       }
     *  解题的思路如下        
        *  1、选取二维数组的中的第一个元素，比较最右边的值，如果相等，就查找结束
        *  2、选取二维数组的中的第一个元素，比较最右边的值，如果大于，查找的就是这个元素，同时角标减去1，然后继续查找
        *  3、选取二维数组的中的第一个元素，比较最右边的值，如果大于，查找的不是这个元素，就去查找下一个数组，然后继续查找![二维数组的查找过程.png](https://upload-images.jianshu.io/upload_images/5363507-a954c84b1c8c5551.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 最终的代码如下

```
 /**
     * 数组是递增的 ，从左到右  从上到下,那么数组一定是个正方形的样子
     *
     * @param arr 原始的数组
     * @param num 需要查找的数字
     * @return 是否找到了
     */
    private boolean newLookUpInTwoDimensionalArrays(int[][] arr, int num) {
        if (arr == null || arr.length < 1 || arr[0].length < 1) {
            return false;
        }
        int rowTotal = arr.length;// 数组的行数
        int colTolal = arr[0].length;// 数组的列数
        //开始的角标
        int row = 0;
        int col = colTolal - 1;
        int i = 0;
        while (row >= 0 && row < rowTotal && col >= 0 && col < colTolal) {
            // 是二维数组的 arr[0][arr[0].length-1] 的值，就是最右边的值
            i++;
            if (arr[row][col] == num) {
                System.out.println(TAG + "newLookUpInTwoDimensionalArrays 执行了多少次" + i);
                return true;
            } else if (arr[row][col] > num) {// 如果找到的值 比目标的值大的话，就把查找的列数减去1
                col--;//列数减去1 ，代表向左移动
            } else {// 比目标的num大的，就把行数加上1，然后往下移动
                row++;
            }
        }
        return false;
    }
```
* 如果查找的值为`15`，那么旧的代码会执行`15`次，新的代码只会执行`3`次。

## 2、请实现一个函数，把字符串中的每个空格替换成`%20`，例如“We are happy.”，则输出`We%20are%20happy.`.
* 第一种方法:先判断字符串中空格的数量。根据数量判断该字符串有没有足够的空间替换成`%20`。如果有足够空间，计算出需要的空间。根据最终需要的总空间，维护一个指针在最后。从后到前，遇到非空的就把该值挪到指针指向的位置，然后指针向前一位，遇到“ ”，则指针前移，依次替换为`02%`。
```
  public char[]  replaceBlank(char[] string, int usedLength) {
        // 判断输入是否合法
        System.out.println(TAG+"string.length="+string.length);
        System.out.println(TAG+"usedLength="+usedLength);
        if (string == null || string.length < usedLength) {
            return null;
        }

        // 统计字符数组中的空白字符数
        int whiteCount = 0;
        for (int i = 0; i < usedLength; i++) {
            if (string[i] == ' ') {
                whiteCount++;
            }
        }
        // 如果没有空白字符就不用处理
        if (whiteCount == 0) {
            return string;
        }
        // 计算转换后的字符长度是多少
        int targetLength = whiteCount * 2 + usedLength;
         //新的保存的字符串的数组
        char[] newChars = new char[targetLength];
        int tmp = targetLength; // 保存长度结果用于返回
//        if (targetLength > string.length) { // 如果转换后的长度大于数组的最大长度，直接返回失败
//            return -1;
//        }
        // todo  必须先做 这个，注意体会  --i  和 i-- 的区别
        usedLength--; // 从后向前，第一个开始处理的字符
        targetLength--; // 处理后的字符放置的位置
        // 字符中有空白字符，一直处理到所有的空白字符处理完
        while (usedLength >= 0 && usedLength < targetLength) {
            // 如是当前字符是空白字符，进行"%20"替换
            if (string[usedLength--] == ' ') {
                newChars[targetLength--] = '0';
                newChars[targetLength--] = '2';
                newChars[targetLength--] = '%';
            } else { // 否则移动字符
                newChars[targetLength--] = string[usedLength];
            }
            usedLength--;
        }
        return newChars;
    }
```
 * 第二种方法，非常的消耗时间和内存。

```
 private String idoWorkReplace(String str, String tagStr) {
        if (str == null || str.length() < 1) {
            return "";
        }
        String[] split = str.split(" ");
        StringBuffer stringBuffer = new StringBuffer();
        for (String s : split) {
            stringBuffer.append(s);
            stringBuffer.append(tagStr);
        }
        CharSequence charSequence = stringBuffer.subSequence(0, stringBuffer.length() - tagStr.length());
        return charSequence.toString();
    }
```
* 第三种方法，调用`replace`方法，如果仅仅来讲替换字符串的话，是最好的方法，关键的方法是`indexOf(this, targetStr, lastMatch)`.
```

 StringBuffer sb = new StringBuffer();
        sb.append(str);
        // todo  最节能的方法
        sb.toString().replace(" ", "%20");
```
* 第四种方法和第一种方法异曲同工:先统计空白的数量，然后计算出有多少空白的长度，然后设置新的长度`StringBuffer`，通过插入最大角标的地方，不断的插入，就可以了
```
 private String replaceSpaces(String string) {
        //判断是否 输入合法
        if (string == null || string.length() < 1) {
            return "";
        }
        char[] chars = string.toCharArray();
        // 统计有多少的空白的数组
        int whiteCount = 0;
        for (int i = 0; i < chars.length; i++) {
            if (chars[i] == ' ') {
                whiteCount++;
            }
        }
        if (whiteCount == 0) {
            return string;
        }
        //最原本的长度
        int indexold = string.length() - 1;
        // 转换完成后的长度
        int newlength = string.length() + whiteCount * 2;
        //新的长度
        int indexnew = newlength - 1;
        StringBuffer stringBuffer = new StringBuffer(string);
        //设置新的buffer的长度
        stringBuffer.setLength(newlength);
        for (; indexold >= 0 && indexold < newlength; indexold--) {
            // 原来的 字符串的最后一位为空格
            if (string.charAt(indexold) == ' ') {
                stringBuffer.setCharAt(indexnew--, '0');
                stringBuffer.setCharAt(indexnew--, '2');
                stringBuffer.setCharAt(indexnew--, '%');
            } else {//不为空的话，就直接放进去 就行了
                stringBuffer.setCharAt(indexnew--, string.charAt(indexold));
            }
        }
        return stringBuffer.toString();
    }
```

## 3、输入个链表的头结点，从尾到头反过来打印出每个结点的值。
* 数据结构
```
 public static class ListNode {
        int val;
        ListNode next;
        public ListNode(int  v){
            this.val=v;
        }
    }
```
* 初始化
```
     ListNode listNode = new ListNode(10);
     ListNode listNode1 = new ListNode(11);
     ListNode listNode2 = new ListNode(12);
     ListNode listNode3 = new ListNode(13);
      ListNode listNode4 = new ListNode(14);
        listNode.next=listNode1;
        listNode1.next=listNode2;
        listNode2.next=listNode3;
        listNode3.next=listNode4;
```
* 第一种方法的详情,非常像二叉树的遍历，也是最简单的方法。
```
   public static ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        arrayList.clear();
        if (listNode != null) {
            printListFromTailToHead(listNode.next);//指向下一个节点
            arrayList.add(listNode.val);//将当前节点的值存到列表中
        }
        return arrayList;
    }
```
* 第二种方法,利用`Stack`的特点，`Stack`类：继承自`Vector`，实现一个后进先出的栈,不太明白的同学可以看这里[常用集合的原理分析](https://www.jianshu.com/p/a5f638bafd3b);
```
private static void doWhat(ListNode listNode) {
        Stack<ListNode> stack = new Stack<>();
        while (listNode!=null){
            stack.push(listNode);
            listNode=listNode.next;
        }
        System.out.println(" stack "+stack.size());
//        for (int i=0;i<stack.size();i++){
//            System.out.println("每个该打印的元素 ::"+stack.get(i).val);
//        }
        ListNode tmp;
        while (!stack.empty()){
            // 移除堆栈顶部的对象，并作为此函数的值返回该对象。
            tmp = stack.pop();
            System.out.println("tmp="+tmp.val);
          //  System.out.println("每个该打印的元素 ："+tmp.val);
        }
    }
```
## 4、输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。这个题的官方的答案，我本人持有严重的怀疑，也有可能是我根本没有找到正确的答案！

* 认识下什么二叉树：二叉树是每个结点最多有两个子树的树结构。通常子树被称作“左子树”`（left subtree）`和“右子树”`（right subtree）`。二叉树常被用于实现二叉查找树和二叉堆。树的数据结构能够同时具备数组查找快的优点以及链表插入和删除快的优点

* 代码结构
```
public interface Tree {
    //查找节点
     Node find(int key);
    //插入新节点
     boolean insert(int data);
    //中序遍历
     void infixOrder(Node current);
    //前序遍历
     void preOrder(Node current);
    //后序遍历
     void postOrder(Node current);
    //查找最大值
     Node findMax();
    //查找最小值
     Node findMin();
    //删除节点
     boolean delete(int key);
}
```
* Node
```
public class Node {
    int data;   //节点数据
    Node leftChild; //左子节点的引用
    Node rightChild; //右子节点的引用

    public Node(int data){
        this.data = data;
    }
    //打印节点内容
    public void display(){
        System.out.println(data);
    }

    @Override
    public String toString() {
        return super.toString();
    }
}
```
* 插入数据的过程

```
      BinaryTree bt = new BinaryTree();
       // 第一个插入的结点是 根节点
       bt.insert(50);
       bt.insert(20);
       bt.insert(80);
       bt.insert(10);
       bt.insert(30);
       bt.insert(60);
       bt.insert(90);
       bt.insert(25);
       bt.insert(85);
       bt.insert(100);
```
* insert 代码

```
//插入节点
    public boolean insert(int data) {
        Node newNode = new Node(data);
        if (root == null) {//当前树为空树，没有任何节点
            root = newNode;
            return true;
        } else {
            Node current = root;
            Node parentNode = null;
            while (current != null) {
                parentNode = current;
                if (current.data > newNode.data) {//当前值比插入值大，搜索左子节点
                    current = current.leftChild;
                    if (current == null) {//左子节点为空，直接将新值插入到该节点
                        parentNode.leftChild = newNode;
                        return true;
                    }
                } else {
                    current = current.rightChild;
                    if (current == null) {//右子节点为空，直接将新值插入到该节点
                        parentNode.rightChild = newNode;
                        return true;
                    }
                }
            }
        }
        return false;
    }
```

![二叉树插入数据的过程.png](https://upload-images.jianshu.io/upload_images/5363507-0b41b46b1356dcc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![二叉树最终的结构.png](https://upload-images.jianshu.io/upload_images/5363507-1bce3daeb9b826cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 最终的图解如下

![二叉树 insert 图解.jpg](https://upload-images.jianshu.io/upload_images/5363507-69f85ae2eea40afe.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 二叉树的前序遍历：根节点>>左子树>>右子树
```
  public void preOrder(Node current) {
        if (current != null) {
            System.out.print(current.data + " ");
            infixOrder(current.leftChild);
            infixOrder(current.rightChild);
        }
    }
```

* 1、根节点50，查找左节点，找到20，然后找到10，输出10，然后找到25 ，然后30 。到这里输出的结果是 50 10 20 25 30
* 2、查找根节点的50的右节点，然后找出80，找出80的左节点60.接着80，查找80的右节点95，输出85 然后 90 ，最后输出100
* 3、最终的输出的结果50 10 20 25 30 60 80 85 90 100

![前序遍历.png](https://upload-images.jianshu.io/upload_images/5363507-1e5c416724d9f985.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* 二叉树中序遍历：首先遍历左子树，然后访问根结点，最后遍历右子树。 左子树 ---》根节点----》 右子树
```
  public void infixOrder(Node current) {
        if (current != null) {
            infixOrder(current.leftChild);
            System.out.print(current.data + " ");
            infixOrder(current.rightChild);
        }
    }
```

 * 1、传入根节点 值为50 ，查找50的左节点为20不为null，再次查找20的左节点，为10，也不为null，然后在查找10的左节点，为null ，输出10，第一个节点输出完成为 10
 * 2、接下来，查找的10的右节点，为null，不进入输出语句
 * 3、这下输出语句20，查找20右节点，查找到值为30，第一步查找的值为左节点，查找到的值为25，查找右节点为null
 * 4、到这里输出的结果为  10  20 25 30
 * 5、这样子50左节点就全部查找完了
 * 6、接下来查找50的右节点，查找右节点80，然后查找80的左节点，查找到的值为60,60没有右节点，继续查找80的右节点，到这里 输出的结果10 20 25 30 50 60 80
 * 7、查找80右节点，查到到值为90，接着查找90的左节点，查到到的值85，接着85的左右节点为null，返回直接查找90的右节点，查找到了100，
 * 8、最终输出的结果，10 20 25 30 50 60 80 85 90 100

![二叉树的中序遍历.png](https://upload-images.jianshu.io/upload_images/5363507-0e19a066dddcb01b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 后序遍历:在二叉树中，先左后右再根，即首先遍历左子树，然后遍历右子树，最后访问根结点。
    * 1、根节点50，查找左节点20，在继续查找20的左节点10,10没有左右节点，打印10,然后查找到20的右节点30,30继续查找到25，第一次输出为 10 20 25 30
    *  2、最终输出10 20 25 30 60 80 85 90 100 50

![二叉树的后序遍历.png](https://upload-images.jianshu.io/upload_images/5363507-c7600a265ea5bcaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 查找最大值或者是最小值
```
 //找到最大值
    public Node findMax() {
        Node current = root;
        Node maxNode = current;
        while (current != null) {
            maxNode = current;
            current = current.rightChild;
        }
        return maxNode;
    }

    //找到最小值
    public Node findMin() {
        Node current = root;
        Node minNode = current;
        while (current != null) {
            minNode = current;
            current = current.leftChild;
        }
        return minNode;
    }
```

* 例如：前序遍历序列`｛50 10 20 25 30 60 80 85 90 100｝`和中序遍历序列`｛10 20 25 30 50 60 80 85 90 100}`， 重建二叉树并输出它的头结点。
* 原始节点的遍历结果,可以很清楚的看到，有多少左右节点，那个节点是那个的节点等等的信息。
```
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node50  是那边啊 根
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node20  是那边啊 左
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node10  是那边啊 左
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node30  是那边啊 右
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node25  是那边啊 左
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node80  是那边啊 右
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node60  是那边啊 左
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node90  是那边啊 右
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node85  是那边啊 左
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node100  是那边啊 右
```
* 解决方法一：
```
 public static Node reConstructBinaryTree(int[] pre, int[] in) {
        if (pre == null || in == null || pre.length != in.length)//如果先序或者中序数组有一个为空的话，就无法建树，返回为空
            return null;
        else {
            return reBulidTree(pre, 0, pre.length - 1, in, 0, in.length - 1);
        }
    }

    /**
     * @param pre
     * @param startPre
     * @param endPre
     * @param in
     * @param startIn
     * @param endIn
     * @return
     */
    private static Node reBulidTree(int[] pre, int startPre, int endPre, int[] in, int startIn, int endIn) {
        if (startPre > endPre || startIn > endIn)//先对传的参数进行检查判断
            return null;
        int root = pre[startPre];//数组的开始位置的元素是跟元素
        int locateRoot = locate(root, in, startIn, endIn);//得到根节点在中序数组中的位置 左子树的中序和右子树的中序以根节点位置为界

        if (locateRoot == -1) //在中序数组中没有找到跟节点，则返回空
            return null;
        Node treeRoot = new Node(root);//创建树根节点
        treeRoot.leftChild = reBulidTree(pre, startPre + 1, startPre + locateRoot - startIn, in, startIn, locateRoot - 1);//递归构建左子树
        treeRoot.rightChild = reBulidTree(pre, startPre + locateRoot - startIn + 1, endPre, in, locateRoot + 1, endIn);//递归构建右子树
        return treeRoot;
    }

    //找到根节点在中序数组中的位置，根节点之前的是左子树的中序数组，根节点之后的是右子树的中序数组
    private static int locate(int root, int[] in, int startIn, int endIn) {
        for (int i = startIn; i < endIn; i++) {
            if (root == in[i])
                return i;
        }
        return -1;
    }

```
* 但是这个的输出结果如下，很明显没有重建二叉树成功，它的执行流程如上面的代码，由于个人认为没有重建，所以就没有输出示意图
```
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 新新----的中序遍历的开始
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node50  是那边啊 根
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node10  是那边啊 左
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node20  是那边啊 右
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node25  是那边啊 右
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node60  是那边啊 右
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node80  是那边啊 右
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node85  是那边啊 右
09-02 05:51:53.177 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node90  是那边啊 右
09-02 05:51:53.185 2590-2590/com.android.interview I/System.out: 新新----的中序遍历的结束
```
* 解决方法二:解题思路也是不断地遍历，代码如下
```
 public static Node reConstructBinaryTreeNew(int[] pre, int[] in) {
        if (pre.length == 0 || in.length == 0)
            return null;
         Node node = new Node(pre[0]);
        for (int i = 0; i < pre.length; i++) {
            if (pre[0] == in[i]) {
                node.leftChild = reConstructBinaryTreeNew(Arrays.copyOfRange(pre, 1, i + 1), Arrays.copyOfRange(in, 0, i));
                node.rightChild = reConstructBinaryTreeNew(Arrays.copyOfRange(pre, i + 1, pre.length), Arrays.copyOfRange(in, i + 1, in.length));
                break;
            }
        }
        return node;
    }
```
*  输出的结果和方法一的结果是一样的。二叉树的结果还是不一样。
```
09-02 05:51:53.185 2590-2590/com.android.interview I/System.out: 新新----的中序遍历的开始------------
09-02 05:51:53.185 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node50  是那边啊 根
09-02 05:51:53.185 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node10  是那边啊 左
09-02 05:51:53.186 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node20  是那边啊 右
09-02 05:51:53.194 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node25  是那边啊 右
09-02 05:51:53.211 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node30  是那边啊 右
09-02 05:51:53.212 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node60  是那边啊 右
09-02 05:51:53.212 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node80  是那边啊 右
09-02 05:51:53.212 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node85  是那边啊 右
09-02 05:51:53.213 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node90  是那边啊 右
09-02 05:51:53.213 2590-2590/com.android.interview I/System.out: 这个值是什么啊 Node100  是那边啊 右
09-02 05:51:53.214 2590-2590/com.android.interview I/System.out: 新新----的中序遍历的结束------------
```

*  解决方法三：由前序遍历的第一个节点可知根节点。根据根节点，可以将中序遍历划分成左右子树。在前序遍历中找出对应的左右子树，其第一个节点便是根节点的左右子节点。按照上述方式递归便可重建二叉树。
```
public class Test {  
    /** 
     * 请实现一个函数，把字符串中的每个空格替换成"%20"，例如“We are happy.“，则输出”We%20are%20happy.“。 
     * 
     * @param string     要转换的字符数组 
     * @param usedLength 已经字符数组中已经使用的长度 
     * @return 转换后使用的字符长度，-1表示处理失败 
     */  
    public static int replaceBlank(char[] string, int usedLength) {  
        // 判断输入是否合法  
        if (string == null || string.length < usedLength) {  
            return -1;  
        }  
  
        // 统计字符数组中的空白字符数  
        int whiteCount = 0;  
        for (int i = 0; i < usedLength; i++) {  
            if (string[i] == ' ') {  
                whiteCount++;  
            }  
        }  
  
        // 计算转换后的字符长度是多少  
        int targetLength = whiteCount * 2 + usedLength;  
        int tmp = targetLength; // 保存长度结果用于返回  
        if (targetLength > string.length) { // 如果转换后的长度大于数组的最大长度，直接返回失败  
            return -1;  
        }  
  
        // 如果没有空白字符就不用处理  
        if (whiteCount == 0) {  
            return usedLength;  
        }  
  
        usedLength--; // 从后向前，第一个开始处理的字符  
        targetLength--; // 处理后的字符放置的位置  
  
        // 字符中有空白字符，一直处理到所有的空白字符处理完  
        while (usedLength >= 0 && usedLength < targetLength) {  
            // 如是当前字符是空白字符，进行"%20"替换  
            if (string[usedLength] == ' ') {  
                string[targetLength--] = '0';  
                string[targetLength--] = '2';  
                string[targetLength--] = '%';  
            } else { // 否则移动字符  
                string[targetLength--] = string[usedLength];  
            }  
            usedLength--;  
        }  
  
        return tmp;  
    }  
}  
```
* 这三种的方式的结果也是一样的，我自己认为二叉树没有重构成功，如果有大佬明白的，可以指点下，谢谢了!

* 谢谢以下资料给与我的帮助
  * [Java数据结构和算法（十）——二叉树](https://www.cnblogs.com/ysocean/p/8032642.html)
  * [android_interview](https://github.com/Shimingli/android_interview)


                                                                                                                                                                 