# 剑指offer

## 1：赋值运算符函数

题目：如下类型为CMyString的声明，请为该类型添加赋值运算符函数。

```c++
class CMyString{
public:
    CMyString(char* pData=nullptr);
    CMyString(const CMyString& str);
    ~CMyString(void);
private:
    char* m_pData;
}
```

注意：

- 返回值类型是否声明为该类型的引用，并在函数结束前返回实例自身的引用（*this）。只有返回一个引用，才可以运行连续赋值。
- 是否把传入的参数类型声明为常量引用。如果传入的参数不是引用而是实例，那么从形参到实参会调用一次copy构造函数。
- 是否释放实例自身的内存。如果忘记在分配新内存之前释放自身已有的空间，则程序将出现内存泄漏。
- 判断传入的实参和当前的实例（*this）是不是同一个实例。如果是同一个，则不进行赋值操作，直接返回。如果事先不判断就进行赋值，那么再释放实例自身内存的时候就会导致严重的问题：当 *this和传入的参数是同一个实例时，一旦释放了自身的内存，传入的参数的内存也同时被释放了，因此也找不到需要赋值的内容了了。

经典解法：

```c++
CMyString& CMyString::operator=(const CMyString&str){
	if(str==&this)
        return *this;
    delete []m_pData;
    m_pData=nullptr;
    
    m_pData=new char[strlen(str.m_pData)+1];
    strcpy(m_pData,str.m_pData);
    return *this; 
}
```

异常安全解法：

在前面的函数中，分配内存之前先用delete释放了实例m_pData的内存。如果此时内存不足导致new char抛出异常，则m_pData将是一个空指针，这样容易导致程序崩溃。

在赋值运算符函数中实现异常安全，方法一，先用new分配新内存，再用delete释放已有的内存；方法二，先创建一个临时的实例，再交换临时实例和原来的实例。

```c++
CMyString& CMyString::operator=(const CMyString& str)
{
    if(thies!=&str)
    {
        CMyString strTemp(str);
        
        char* pTemp=strTemp.m_pData;
        strTemp.m_pData=m_pData;
        m_pData=pTemp;
    }
    return *this;
}
```

## 2：Singleton模式

题目：设计一个类，我们只能生成该类的一个实例

懒汉式：

```c++
class singleton   //实现单例模式的类  
{  
private:  
    singleton(){}  //私有的构造函数  
    static singleton* Instance;  
public:  
    static singleton* GetInstance()  
    {  
        if (Instance == NULL) //判断是否第一调用  
            Instance = new singleton();  
        return Instance;  
    }  
}; 
```

改进的懒汉式：

```c++
class singleton   //实现单例模式的类  
{  
private:  
    singleton(){}  //私有的构造函数  
    static singleton* Instance;  
      
public:  
    static singleton* GetInstance()  
    {  
        if (Instance == NULL) //判断是否第一调用  
        {   
            Lock(); //表示上锁的函数  
            if (Instance == NULL)  
            {  
                Instance = new singleton();  
            }  
            UnLock() //解锁函数  
        }             
        return Instance;  
    }  
};  
```

饿汉式：

```c++
class singleton   //实现单例模式的类  
{  
private:  
    singleton() {}  //私有的构造函数  
      
public:  
    static singleton* GetInstance()  
    {  
        static singleton Instance;  
        return &Instance;  
    }  
}; 
```

## 3：数组中重复的数字

题目一：找出数组中重复的数字

在一个长度为n的数组里所有数字都在0~N-1的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道重复了几次。找出数组中任意一个重复的数字。例如，如果输入长度为7的数组{2，3，1，0，2，5，3}，那么对应的输出应该是重复的数字2或者3。

```c++
bool duplicate(int numbers[], int length, int* duplication)
{
    if(numbers == nullptr || length <= 0)    //判断给定数组和长度是否符合要求
        return false;

    for(int i = 0; i < length; ++i)        //是否满足0~N-1的范围要求
    {
        if(numbers[i] < 0 || numbers[i] > length - 1)
            return false;
    }

    for(int i = 0; i < length; ++i)
    {
        while(numbers[i] != i)
        {
            if(numbers[i] == numbers[numbers[i]])
            {
                *duplication = numbers[i];
                return true;
            }
            // 交换numbers[i]和numbers[numbers[i]]             
            int temp = numbers[i];
            numbers[i] = numbers[temp];
            numbers[temp] = temp;
        }
    }

    return false;
}

```

思路：依次遍历数组，做值与下标号比较，不相同则交换至该值对应的下标号处，如果下次需要交换的值所对应的下标号已经有等于该下标号的值，说明有重复的数字存在。

题目二：不能修改数组找出重复的数字

在一个长度为n+1的数组里的所有数字都在1~n的范围内，所以数组中至少有一个数字是重复的。请找出数组中任一重复的数字，但不能改变输入的数组。

```c++
int getDuplication(const int* numbers, int length)
{
    if(numbers == nullptr || length <= 0)
        return -1;

    int start = 1;
    int end = length - 1;
    while(end >= start)
    {
        int middle = ((end - start) >> 1) + start;
        int count = countRange(numbers, length, start, middle);
        if(end == start)
        {
            if(count > 1)
                return start;
            else
                break;
        }

        if(count > (middle - start + 1))
            end = middle;
        else
            start = middle + 1;
    }
    return -1;
}

int countRange(const int* numbers, int length, int start, int end)
{
    if(numbers == nullptr)
        return 0;

    int count = 0;
    for(int i = 0; i < length; i++)
        if(numbers[i] >= start && numbers[i] <= end)
            ++count;
    return count;
}
```

思路：把从1~n的数字从中间的数字m分为两部分，前面一半为1~m，后面一半为m+1~n。如果后面一半为m+1~n。如果1~m的数字的数目超过m，那么这一半的区间里一定包含重复的数字；否则，另一半m+1~n的区间里一定包含重复的数字。可以继续把包含重复数字的区间一分为二，直到找到一个重复的数字。

## 4：二位数组中的查找

题目：在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

```c++
 bool Find(int target, vector<vector<int> > array) {
        int n=array.size();//行
        int m=array[0].size();
        int i=n-1,j=0;
        while(i>=0&&j<m)
        {
            if(array[i][j]==target)
                return 1;
            else if(array[i][j]<target)
                j++;
            else
                i--;
        }
        return 0;
    }
```

## 5：空格替换

题目：请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

```c++
//my resolution
void replaceSpace(char *str,int length) {
        int i = 0;
	    queue<string> que;
	    string tmp;
	    while (str[i] != '\0')
	    {
	    	if (str[i] == ' ')
	    	{
		    	que.push(tmp);
		    	que.push("%20");
		    	tmp = "";
		    	i++;
	    	}
            if(str[i] != ' '){
		        tmp += str[i];
		        i++;
            }
    	}
    	que.push(tmp);
        string ans;
    	while (que.size()!=0)
    	{
    		ans+=que.front();
    		que.pop();
    	}
        strcpy(str,ans.c_str());
	}
```

```c++
//standard
//双指针，从末尾开始复制
void ReplaceBlank(char str[], int length)
{
    if(str == nullptr && length <= 0)
        return;

    /*originalLength 为字符串str的实际长度*/
    int originalLength = 0;
    int numberOfBlank = 0;
    int i = 0;
    while(str[i] != '\0')
    {
        ++ originalLength;

        if(str[i] == ' ')
            ++ numberOfBlank;

        ++ i;
    }

    /*newLength 为把空格替换成'%20'之后的长度*/
    int newLength = originalLength + numberOfBlank * 2;
    if(newLength > length)
        return;

    int indexOfOriginal = originalLength;
    int indexOfNew = newLength;
    while(indexOfOriginal >= 0 && indexOfNew > indexOfOriginal)
    {
        if(str[indexOfOriginal] == ' ')
        {
            str[indexOfNew --] = '0';
            str[indexOfNew --] = '2';
            str[indexOfNew --] = '%';
        }
        else
        {
            str[indexOfNew --] = str[indexOfOriginal];
        }

        -- indexOfOriginal;
    }
}
```

## 6：从尾到头打印链表

题目：输入一个链表的头节点，从尾到头反过来打印出每个节点值。

```c++
vector<int> printListFromTailToHead(ListNode* head) {
        stack<int> stc;
        ListNode* tmp=head;
        while(tmp!=NULL)
        {
            stc.push(tmp->val);
            tmp=tmp->next;
        }
        vector<int> ans;
        while(!stc.empty())
        {
            ans.push_back(stc.top());
            stc.pop();
        }
        return ans;
    }
```

## 7：重建二叉树

题目：输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

```c++
TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> in) {
        int inlen=in.size();
        if(inlen==0)
            return NULL;
        vector<int> left_pre,right_pre,left_in,right_in;
        TreeNode* head=new TreeNode(pre[0]);
        int gen=0;      //记录根节点
        for(int i=0;i<inlen;i++){
			if(in[i]==pre[0]){
                gen=i;
                break;
            }
        }
        for(int i=0;i<gen;i++){
            left_in.push_back(in[i]);  //中序遍历中的左子树
            left_pre.push_back(pre[i+1]);//前序遍历中的左子树
        }
        for(int i=gen+1;i<inlen;i++){
            right_in.push_back(in[i]);//前序遍历中的右子树
            right_pre.push_back(pre[i]);//中序遍历中的右子树
        }
        head->left=reConstructBinaryTree(left_pre,left_in);
        head->right=reConstructBinaryTree(right_pre,right_in);
		return head;
    }
```

## 8：二叉树的下一个节点

题目：给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。

```c++
//my resolution
vector<TreeLinkNode*> vec;
    TreeLinkNode* GetNext(TreeLinkNode* pNode)
    {
        TreeLinkNode* tmp=pNode;
        while(pNode->next!=NULL)
            pNode=pNode->next;
        in(pNode);
        vec.push_back(NULL);
        int n=vec.size();
        for(int i=0;i<n;i++)
            if(vec[i]==tmp)
                return vec[i+1];
        return NULL;
        
    }
    void in(TreeLinkNode* pNode){      
        if(pNode)
        {
            in(pNode->left);   
            vec.push_back(pNode);       
            in(pNode->right);
        }
    }
```

```c
//分两种情况，有没有右子节点
TreeLinkNode* GetNext(TreeLinkNode* pNode)
    {
        if(pNode==NULL) return NULL;
        if(pNode->right!=NULL)
        {
            pNode=pNode->right;
            while(pNode->left!=NULL)
                pNode=pNode->left;
            return pNode;
        }
        while(pNode->next!=NULL)
        {
            if(pNode->next->left==pNode) 
                return pNode->next;
            pNode=pNode->next;
        }
         
        return NULL;
    }
```

## 9：两个栈实现队列

```c++
class Solution
{
public:
    void push(int node) {
        stack1.push(node);
    }

    int pop() {
        if(stack2.empty())
        while(!stack1.empty())
        {
            stack2.push(stack1.top());
            stack1.pop();
        }
        int ans=stack2.top();
        stack2.pop();
        return ans;     
    }

private:
    stack<int> stack1;
    stack<int> stack2;
};
```

## 10：斐波那契数列

题目：求斐波那契数列的第n项

```c++
//递归实现
long long Fibonacci_Solution1(unsigned int n)
{
    if(n <= 0)
        return 0;
    if(n == 1)
        return 1;
    return Fibonacci_Solution1(n - 1) + Fibonacci_Solution1(n - 2);
}
```

```c++
//非递归实现，时间复杂度O(n)
long long Fibonacci_Solution2(unsigned n)
{
    int result[2] = {0, 1};
    if(n < 2)
        return result[n];

    long long  fibNMinusOne = 1;
    long long  fibNMinusTwo = 0;
    long long  fibN = 0;
    for(unsigned int i = 2; i <= n; ++ i)
    {
        fibN = fibNMinusOne + fibNMinusTwo;

        fibNMinusTwo = fibNMinusOne;
        fibNMinusOne = fibN;
    }

     return fibN;
}
```

## 11：旋转数组的最小数字

题目：把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非减排序的数组的一个旋转，输出旋转数组的最小元素。 例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。

```c++
 //类似于二分查找
 int minNumberInRotateArray(vector<int> rotateArray) {
        int size=rotateArray.size();
        if(size==0)
            return 0;    
        int left=0;
        int right=size-1;
        int mid=0;
        while(rotateArray[left]>=rotateArray[right])  
        {
            if(right-left==1) 
            {
                mid=right;
                break;
            }
            mid=(left+right)>>1;   
            if(rotateArray[left]<=rotateArray[mid])
                left=mid;
            else
                right=mid;
        }
        return rotateArray[mid];
    }
```

## 12：矩阵中的路径

题目：请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。 例如 a b c e s f c s a d e e 这样的3 X 4 矩阵中包含一条字符串"bcced"的路径，但是矩阵中不包含"abcb"路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。

```c++
//回溯法 
bool hasPath(const char* matrix, int rows, int cols, const char* str)
{
    if(matrix == nullptr || rows < 1 || cols < 1 || str == nullptr)
        return false;

    bool *visited = new bool[rows * cols];
    memset(visited, 0, rows * cols);

    int pathLength = 0;
    for(int row = 0; row < rows; ++row)
    {
        for(int col = 0; col < cols; ++col)
        {
            if(hasPathCore(matrix, rows, cols, row, col, str,
                pathLength, visited))
            {
                return true;
            }
        }
    }

    delete[] visited;

    return false;
}

bool hasPathCore(const char* matrix, int rows, int cols, int row,
    int col, const char* str, int& pathLength, bool* visited)
{
    if(str[pathLength] == '\0')
        return true;

    bool hasPath = false;
    if(row >= 0 && row < rows && col >= 0 && col < cols
        && matrix[row * cols + col] == str[pathLength]
        && !visited[row * cols + col])
    {
        ++pathLength;
        visited[row * cols + col] = true;

        hasPath = hasPathCore(matrix, rows, cols, row, col - 1,
            str, pathLength, visited)
            || hasPathCore(matrix, rows, cols, row - 1, col,
                str, pathLength, visited)
            || hasPathCore(matrix, rows, cols, row, col + 1,
                str, pathLength, visited)
            || hasPathCore(matrix, rows, cols, row + 1, col,
                str, pathLength, visited);

        if(!hasPath)
        {
            --pathLength;
            visited[row * cols + col] = false;
        }
    }

    return hasPath;
}
```

## 13：机器人的运动范围

题目：地上有一个m行和n列的方格。一个机器人从坐标（0,0）的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？

```c++
//回溯法
int movingCount(int threshold, int rows, int cols)
{
    if(threshold < 0 || rows <= 0 || cols <= 0)
        return 0;

    bool *visited = new bool[rows * cols];
    for(int i = 0; i < rows * cols; ++i)
        visited[i] = false;

    int count = movingCountCore(threshold, rows, cols,
        0, 0, visited);

    delete[] visited;

    return count;
}

int movingCountCore(int threshold, int rows, int cols, int row,
    int col, bool* visited)
{
    int count = 0;
    if(check(threshold, rows, cols, row, col, visited))
    {
        visited[row * cols + col] = true;

        count = 1 + movingCountCore(threshold, rows, cols,
            row - 1, col, visited)
            + movingCountCore(threshold, rows, cols,
                row, col - 1, visited)
            + movingCountCore(threshold, rows, cols,
                row + 1, col, visited)
            + movingCountCore(threshold, rows, cols,
                row, col + 1, visited);
    }

    return count;
}

bool check(int threshold, int rows, int cols, int row, int col,
    bool* visited)
{
    if(row >= 0 && row < rows && col >= 0 && col < cols
        && getDigitSum(row) + getDigitSum(col) <= threshold
        && !visited[row* cols + col])
        return true;

    return false;
}

int getDigitSum(int number)
{
    int sum = 0;
    while(number > 0)
    {
        sum += number % 10;
        number /= 10;
    }

    return sum;
}
```

## 14：剪绳子

题目：给你一根长度为n绳子，请把绳子剪成m段（m、n都是整数，n>1并且m≥1）。每段的绳子的长度记为k[0]、k[1]、……、k[m]。k[0]* k[1] * … * k[m]可能的最大乘  积是多少？例如当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此 时得到最大的乘积18。                                      |

```c++
//动态规划
int maxProductAfterCutting_solution1(int length)
{
    if(length < 2)
        return 0;
    if(length == 2)
        return 1;
    if(length == 3)
        return 2;

    int* products = new int[length + 1];
    products[0] = 0;
    products[1] = 1;
    products[2] = 2;
    products[3] = 3;

    int max = 0;
    for(int i = 4; i <= length; ++i)
    {
        max = 0;
        for(int j = 1; j <= i / 2; ++j)
        {
            int product = products[j] * products[i - j];
            if(max < product)
                max = product;

            products[i] = max;
        }
    }

    max = products[length];
    delete[] products;

    return max;
}
```

```c++
//贪婪算法
int maxProductAfterCutting_solution2(int length)
{
    if(length < 2)
        return 0;
    if(length == 2)
        return 1;
    if(length == 3)
        return 2;

    // 尽可能多地减去长度为3的绳子段
    int timesOf3 = length / 3;

    // 当绳子最后剩下的长度为4的时候，不能再剪去长度为3的绳子段。
    // 此时更好的方法是把绳子剪成长度为2的两段，因为2*2 > 3*1。
    if(length - timesOf3 * 3 == 1)
        timesOf3 -= 1;

    int timesOf2 = (length - timesOf3 * 3) / 2;

    return (int) (pow(3, timesOf3)) * (int) (pow(2, timesOf2));
}
```

## 15：二进制中1的个数

```c++
//解法一
int  NumberOf1(int n) {
         int count=0;
         unsigned int flag=1;
         while(flag)
         {
             if(n&flag)
                 count++;
             flag=flag<<1;   //一共需要移动32次
         }
         return count;
     }
```

```c++
//解法二
int NumberOf1_Solution2(int n)
{
    int count = 0;

    while (n)
    {
        ++count;
        n = (n - 1) & n;
    }

    return count;
}
```

## 16：数值的整数次方

题目：给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。

```c++
bool g_InvalidInput = false;
double Power(double base, int exponent)
{
    g_InvalidInput = false;

    if (equal(base, 0.0) && exponent < 0)
    {
        g_InvalidInput = true;
        return 0.0;
    }

    unsigned int absExponent = (unsigned int) (exponent);
    if (exponent < 0)
        absExponent = (unsigned int) (-exponent);

    double result = PowerWithUnsignedExponent(base, absExponent);
    if (exponent < 0)
        result = 1.0 / result;

    return result;
}

double PowerWithUnsignedExponent(double base, unsigned int exponent)
{
    if (exponent == 0)
        return 1;
    if (exponent == 1)
        return base;

    double result = PowerWithUnsignedExponent(base, exponent >> 1);
    result *= result;
    if ((exponent & 0x1) == 1)
        result *= base;

    return result;
}

bool equal(double num1, double num2)   //判断两个double是否相等
{
    if ((num1 - num2 > -0.0000001) && (num1 - num2 < 0.0000001))
        return true;
    else
        return false;
}
```

## 17：打印从1到最大的n位数

题目：输入数字n，按顺序打印出从1到最大的n位十进制数。比如输入3，打印出1、2、3一直到最大的3位数999。

难点：当n极大的情况，因此借助字符串模拟数字加

```c++
//方法一
void Print1ToMaxOfNDigits_1(int n)
{
    if (n <= 0)
        return;

    char *number = new char[n + 1];
    memset(number, '0', n);
    number[n] = '\0';

    while (!Increment(number)) //每次加1，溢出返回
    {
        PrintNumber(number);
    }

    delete[]number;
}

// 字符串number表示一个数字，在 number上增加1
// 如果做加法溢出，则返回true；否则为false
bool Increment(char* number)
{
    bool isOverflow = false;
    int nTakeOver = 0;  //有无溢出
    int nLength = strlen(number);

    for (int i = nLength - 1; i >= 0; i--)     //number高位存放数字低位
    {
        int nSum = number[i] - '0' + nTakeOver;
        if (i == nLength - 1)
            nSum++;      //最低位加一

        if (nSum >= 10)
        {
            if (i == 0)
                isOverflow = true;  //最高位溢出
            else
            {
                nSum -= 10;
                nTakeOver = 1;
                number[i] = '0' + nSum;
            }
        }
        else
        {
            number[i] = '0' + nSum;
            break;
        }
    }

    return isOverflow;
}
```

```c++
//方法二：全排列方法
void Print1ToMaxOfNDigits_2(int n)
{
    if (n <= 0)
        return;

    char* number = new char[n + 1];
    number[n] = '\0';

    for (int i = 0; i < 10; ++i)
    {
        number[0] = i + '0';
        Print1ToMaxOfNDigitsRecursively(number, n, 0);
    }

    delete[] number;
}

void Print1ToMaxOfNDigitsRecursively(char* number, int length, int index)
{
    if (index == length - 1)
    {
        PrintNumber(number);
        return;
    }

    for (int i = 0; i < 10; ++i)
    {
        number[index + 1] = i + '0';
        Print1ToMaxOfNDigitsRecursively(number, length, index + 1);
    }
}
```

```c++
//打印字符串
void PrintNumber(char* number)
{
    bool isBeginning0 = true;
    int nLength = strlen(number);

    for (int i = 0; i < nLength; ++i)
    {
        if (isBeginning0 && number[i] != '0')
            isBeginning0 = false;

        if (!isBeginning0)
        {
            printf("%c", number[i]);
        }
    }

    printf("\t");
}
```

## 18：删除列表的节点

题目一：在O(1）时间内删除链表节点

给定单链表的头指针和一个节点指针，定义一个函数在O(1)时间内删除该节点。

注意：考虑删除节点在链表的不同位置，链表中间，链表尾部，单一节点。

```c++
void DeleteNode(ListNode** pListHead, ListNode* pToBeDeleted)
{
    if(!pListHead || !pToBeDeleted)
        return;

    // 要删除的结点不是尾结点
    if(pToBeDeleted->m_pNext != nullptr)
    {
        ListNode* pNext = pToBeDeleted->m_pNext;  //将下一个节点复制到被删节点
        pToBeDeleted->m_nValue = pNext->m_nValue;
        pToBeDeleted->m_pNext = pNext->m_pNext;
 
        delete pNext;      //释放删除节点的下一个节点
        pNext = nullptr;
    }
    // 链表只有一个结点，删除头结点（也是尾结点）
    else if(*pListHead == pToBeDeleted)
    {
        delete pToBeDeleted;
        pToBeDeleted = nullptr;
        *pListHead = nullptr;
    }
    // 链表中有多个结点，删除尾结点
    else
    {
        ListNode* pNode = *pListHead;
        while(pNode->m_pNext != pToBeDeleted)
        {
            pNode = pNode->m_pNext;            
        }
 
        pNode->m_pNext = nullptr;
        delete pToBeDeleted;
        pToBeDeleted = nullptr;
    }
}
```

题目二：删除链表中重复节点

在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留。 例如，链表1->2->3->3->4->4->5 处理后为 1->2->5

```c++
 ListNode* deleteDuplication(ListNode* pHead)
    {
        if(pHead==NULL||pHead->next==NULL) 
            return pHead;
        
        ListNode* newHead=new ListNode(-1);
        newHead->next=pHead;
        ListNode* p=pHead;
        ListNode* pre=newHead;
        ListNode* next=NULL;
        while(p!=NULL&&p->next!=NULL)
        {
            next=p->next;
            if(p->val==next->val)
            {
                while(next!=NULL&&next->val==p->val)
                    next=next->next;
                pre->next=next;
                p=next;
            }
            else
            {
                pre=p;
                p=p->next;
            }
        }
        return newHead->next;
        
    }
```

## 19：正则表达式匹配

> 题目：请实现一个函数用来匹配包括'.'和 ' * '  的正则表达式。模式中的字符  '.' 表示任意一个字符，而 '* ' 表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab * ac *a"匹配，但是与"aa.a"和"ab * a"均不匹配

```c++
bool match(const char* str, const char* pattern)
{
    if(str == nullptr || pattern == nullptr)
        return false;

    return matchCore(str, pattern);
}

bool matchCore(const char* str, const char* pattern)
{
    if(*str == '\0' && *pattern == '\0')
        return true;

    if(*str != '\0' && *pattern == '\0')
        return false;

    if(*(pattern + 1) == '*')
    {
        if(*pattern == *str || (*pattern == '.' && *str != '\0'))
            // 进入有限状态机的下一个状态
            return matchCore(str + 1, pattern + 2)
            // 继续留在有限状态机的当前状态 
            || matchCore(str + 1, pattern)
            // 略过一个'*' 
            || matchCore(str, pattern + 2);
        else
            // 略过一个'*'
            return matchCore(str, pattern + 2);
    }

    if(*str == *pattern || (*pattern == '.' && *str != '\0'))
        return matchCore(str + 1, pattern + 1);

    return false;
}
```

## 20：表示数值的字符串

题目：请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。 但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。

```c++
bool isNumeric(char* string)
    {
        if(string==NULL)
            return false;
        if(*string=='+'||*string=='-')
            string++;
        if(*string=='\0')
            return false;
        int dot=0,num=0,nume=0;//分别用来标记小数点、整数部分和e指数是否存在
        while(*string!='\0'){
            if(*string>='0'&&*string<='9')
            {  
                string++;
                num=1;
            }
            else if(*string=='.'){
                if(dot>0||nume>0)
                    return false;
                string++;
                dot=1;
            }
            else if(*string=='e'||*string=='E')
                {
                  if(num==0)
                      return false;
                  string++;
                  nume++;
                  if(*string=='+'||*string=='-')
                      string++;
                 if(*string=='\0')
                     return false;
                }
            else
                return false;
        }
        return true;
    }
```

## 21：调整数组顺序使奇数位于偶数前面

> 题目：输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分。

```c++
void Reorder(int *pData, unsigned int length, bool (*func)(int))
{
    if(pData == nullptr || length == 0)
        return;

    int *pBegin = pData;
    int *pEnd = pData + length - 1;

    while(pBegin < pEnd) 
    {
        // 向后移动pBegin
        while(pBegin < pEnd && !func(*pBegin))
            pBegin ++;

        // 向前移动pEnd
        while(pBegin < pEnd && func(*pEnd))
            pEnd --;

        if(pBegin < pEnd)
        {
            int temp = *pBegin;
            *pBegin = *pEnd;
            *pEnd = temp;
        }
    }
}
bool isEven(int n)
{
    return (n & 1) == 0;
}
```



## 22：链表中倒数第k个节点

> 题目：输入一个链表，输出该链表中倒数第k个结点。

```c++
//双指针，指针距离为k
ListNode* FindKthToTail(ListNode* pListHead, unsigned int k)
{
    if(pListHead == nullptr || k == 0)
        return nullptr;

    ListNode *pAhead = pListHead;
    ListNode *pBehind = nullptr;

    for(unsigned int i = 0; i < k - 1; ++ i)
    {
        if(pAhead->m_pNext != nullptr)  //注意
            pAhead = pAhead->m_pNext;
        else
        {
            return nullptr;
        }
    }

    pBehind = pListHead;
    while(pAhead->m_pNext != nullptr)
    {
        pAhead = pAhead->m_pNext;
        pBehind = pBehind->m_pNext;
    }

    return pBehind;
}
```

## 23：链表中环的入口节点

给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。

```c++
ListNode* EntryNodeOfLoop(ListNode* pHead)
    {
        /*  断链
        if(pHead==NULL||pHead->next==NULL) return NULL;
        ListNode* fast=pHead->next;
        ListNode* slow=pHead;
        while(fast!=NULL)
        {
            slow->next=NULL;
            slow=fast;
            fast=fast->next;
        }
        return slow;
        */
        // 2倍速度追赶
        if(pHead==NULL||pHead->next==NULL) return NULL;
        ListNode* fast=pHead;
        ListNode* slow=pHead;
        ListNode* cur=pHead;
        while(fast!=NULL)
        {
            fast=fast->next->next;
            slow=slow->next;
            if(fast==slow)
            {
                while(cur!=slow)
                {
                    cur=cur->next;
                    slow=slow->next;
                } 
                return cur;
            }
            
        }
        return NULL;
    }
```

## 24：反转链表

输入一个链表，反转链表后，输出新链表的表头。

```c++
ListNode* ReverseList(ListNode* pHead) {
        if(pHead==nullptr||pHead->next==nullptr) return pHead;      
        ListNode* pre=nullptr;
        ListNode* cur=pHead;
        while(cur!=nullptr)
        {
            ListNode* next=cur->next;
            cur->next=pre;
            pre=cur;
            cur=next;
        }
        return pre;
    }
```

## 25：合并两个排序的链表

> 输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

```c++
  //非递归
	ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
    {
        if(pHead1==nullptr)
            return pHead2;
        if(pHead2==nullptr)
            return pHead1;
        if(pHead1==nullptr&&pHead2==nullptr) return nullptr;
        ListNode* newpHead=new ListNode(-1);
        ListNode* cur=newpHead;
        while(pHead1!=nullptr&&pHead2!=nullptr)
        {
            if(pHead1->val<=pHead2->val)
            {
                cur->next=pHead1;
                pHead1=pHead1->next;
            }
            else
            {
                cur->next=pHead2;
                pHead2=pHead2->next;
            }
            
            cur=cur->next;
            
        }
        if(pHead1==nullptr)
        {
            cur->next=pHead2;
         }
         else if(pHead2==nullptr)
         {
             cur->next=pHead1;
         }
       return newpHead->next;
        
    }
```

```c++
//递归
	ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
    {
            ListNode* result=NULL;
            if(pHead1==NULL)
            	return pHead2;
            else if(pHead2==NULL)
           		return pHead1;
            if(pHead1->val<=pHead2->val)
            {
                result=pHead1;
                result->next=Merge(pHead1->next,pHead2);
            }
            else
            {
                result=pHead2;
                result->next=Merge(pHead1,pHead2->next);
            }
         
        	return result;
    }
```

## 26：树的子结构

> 输入两棵二叉树A，B，判断B是不是A的子结构。

```c++
	bool HasSubtree(TreeNode* pRoot1, TreeNode* pRoot2)
    {
        bool result=false;
        if(pRoot1!=NULL&&pRoot2!=NULL){
            if(pRoot1->val==pRoot2->val)
                result=Tree1subTree2(pRoot1, pRoot2);
            if(!result)
                result=HasSubtree(pRoot1->left,pRoot2);
            if(!result)
                result=HasSubtree(pRoot1->right,pRoot2);            
        }
		return result;
    }
    bool Tree1subTree2(TreeNode* pRoot1, TreeNode* pRoot2){
        if(pRoot2==NULL)
            return true;
        if(pRoot1==NULL)
            return false;
        if(pRoot1->val!=pRoot2->val)
        	return false;
        return Tree1subTree2(pRoot1->left,pRoot2->left)&&Tree1subTree2(pRoot1->right,pRoot2->right);
    }
```

## 27：二叉树的镜像

> 操作给定的二叉树，将其变换为源二叉树的镜像。

```c++
void Mirror(TreeNode *pRoot) 
{     
        if(pRoot==nullptr) return;
        if(pRoot->left==nullptr&&pRoot->right==nullptr) return;
         TreeNode *tmp=pRoot->left;
         pRoot->left=pRoot->right;
         pRoot->right=tmp;      
        if(pRoot->left!=nullptr)
            Mirror(pRoot->left);
        if(pRoot->right!=nullptr)
            Mirror(pRoot->right);
    }
```

```c++
void Mirror(TreeNode *pRoot) {
    //非递归方法
    if(pRoot==NULL)
        return;
    stack<TreeNode *> s;
    s.push(pRoot);
    while(s.size()){
        TreeNode *node=s.top();
        s.pop();
        if(node->left!=NULL||node->right!=NULL)
        {
            TreeNode *tmp=node->left;
            node->left=node->right;
            node->right=tmp;
        }
        if(node->left) s.push(node->left);
        if(node->right) s.push(node->right);
    }
}
```

## 28：对称的二叉树

> 请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

```c++
	bool isSymmetrical(TreeNode* pRoot)
    {
        if(pRoot==NULL)
            return true;
        return comRoot(pRoot->left,pRoot->right);
    }
    bool comRoot(TreeNode *left,TreeNode *right)
    {
        if(left==NULL) return right==NULL;
        if(right==NULL) return false;
        if(left->val!=right->val) return false;
        return comRoot(left->right,right->left)&&comRoot(left->left,right->right);
    }
```

## 29：顺时针打印矩阵

> 输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

```c++
	vector<int> printMatrix(vector<vector<int> > matrix) 
    {
		int row = matrix.size();
        int col = matrix[0].size();
        vector<int> res;
         
        // 输入的二维数组非法，返回空的数组
        if (row == 0 || col == 0)  return res;
         
        // 定义四个关键变量，表示左上和右下的打印范围
        int left = 0, top = 0, right = col - 1, bottom = row - 1;
        while (left <= right && top <= bottom)
        {
            // left to right
            for (int i = left; i <= right; ++i)  res.push_back(matrix[top][i]);
            // top to bottom
            for (int i = top + 1; i <= bottom; ++i)  res.push_back(matrix[i][right]);
            // right to left
            if (top != bottom)
            for (int i = right - 1; i >= left; --i)  res.push_back(matrix[bottom][i]);
            // bottom to top
            if (left != right)
            for (int i = bottom - 1; i > top; --i)  res.push_back(matrix[i][left]);
            left++,top++,right--,bottom--;
        }
        return res;
    }
```

## 30：包含min函数的栈

> 定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。

```c++
 	//注意时间复杂度
	stack<int> stack1,stack2;
     
    void push(int value) {
        stack1.push(value);
        if(stack2.empty())
            stack2.push(value);
        else if(value<=stack2.top())
        {
            stack2.push(value);
        }
    }
     
    void pop() {
        if(stack1.top()==stack2.top())
            stack2.pop();
        stack1.pop();
         
    }
     
    int top() {
        return stack1.top();       
    }
     
    int min() {
        return stack2.top();
    }
```

## 31：栈的压入、弹出序列

> 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。

```c++
 bool IsPopOrder(vector<int> pushV,vector<int> popV) {
        int n=pushV.size();
        int i=0,j=0;
        stack<int> s; 
        for(i;i<n;i++)
        {
            s.push(pushV[i]);
            while(j<n&&s.top()==popV[j])
            {
                s.pop();
                j++;
            }
        }
        return s.empty();
    }
```

## 32：打印二叉树

> 题目一（从上到下打印二叉树）：从上往下打印出二叉树的每个节点，同层节点从左至右打印。

```
vector<int> PrintFromTopToBottom(TreeNode* root) {
        vector<int> vec;
        if(root==nullptr) 
            return vec;
        queue<TreeNode*> que;
        que.push(root);
        while(!que.empty())
        {
            TreeNode* tmp=que.front();
            vec.push_back(tmp->val);
            if(tmp->left!=nullptr) que.push(tmp->left);
            if(tmp->right!=nullptr) que.push(tmp->right);
            que.pop();
        }
        return vec;

    }
```

> 题目二（按层打印二叉树）：从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。

```c++
vector<vector<int> > Print(TreeNode* pRoot) {
        vector<vector<int> > res;
        if(pRoot==NULL) return res;
        queue<TreeNode*> que;        
        que.push(pRoot);
        while(!que.empty())
        {    
            vector<int> vec;
            int s=que.size();
            for(int i=0;i<s;i++)
            {
                TreeNode* tmp=que.front();
                vec.push_back(tmp->val);
                que.pop();                         
                if(tmp->left!=NULL)
                    que.push(tmp->left);
                if(tmp->right!=NULL)
                    que.push(tmp->right);
            }
            res.push_back(vec);
        }
}
```

> 题目三（之字形打印）

```c++
vector<vector<int> > Print(TreeNode* pRoot) {
        vector<vector<int> > res;
        if(pRoot==NULL) return res;
        queue<TreeNode*> que;        
        que.push(pRoot);
        bool flag=0;
        while(!que.empty())
        {    
            vector<int> vec;////
            int s=que.size();
            for(int i=0;i<s;i++)
            {
                TreeNode* tmp=que.front();
                vec.push_back(tmp->val);
                que.pop();                         
                if(tmp->left!=NULL)
                    que.push(tmp->left);
                if(tmp->right!=NULL)
                    que.push(tmp->right);
            }
            if(flag)
                reverse(vec.begin(),vec.end());
            res.push_back(vec);
            flag=!flag;

        }
        return res;
    }
```

	bool VerifySquenceOfBST(vector<int> sequence) {
	    int size=sequence.size();
	    if(size==0) 
	        return false;
	    int root=sequence[size-1];
	    int i=0;
	    vector<int> seq_left;
	    for(;i<size-1;i++)
	    {
	        if(root<sequence[i])
	            break;
	        seq_left.push_back(sequence[i]);         
	    }
	    int j=i;
	    vector<int> seq_right;
	    for(;j<size-1;j++)
	    {
	        if(root>sequence[j])
	            return false;
	        seq_right.push_back(sequence[j]);
	    }
	    bool left=true,right=true;
	    if(i>0)
	        left=VerifySquenceOfBST(seq_left);
	    if(i<size-1)
	        right=VerifySquenceOfBST(seq_right);
	    return left&&right；        
	}