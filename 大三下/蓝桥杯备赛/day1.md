①查找第2025个质数
- sqrt(10),需要包含math.h头文件，返回double类型
- 经过测试，在蓝桥杯环境下循环中直接return会段错误，要先break;
- 提交之前一定先测一下能不能跑通！
②填棋盘
给出一个6×6的棋盘，若干白棋、黑棋，还有空白
返回一个数字字符串，为填完后的棋盘，满足下面的条件：
- 每一行（row）/列(col) 不同
- 每一行（row）/列（col）黑棋，白棋的个数相同，即为3
- 不存在连续三个相同的棋子连在一起，如 BBB or WWW
解题思路:递归，回溯

主循环调用backtrack(0)<---参数为0
backtrack函数内部

```cpp
bool backtrack(int pos)
{
	//终止条件 
	if(pos == 36){
		return 1;
	}
	
	int row = pos/6;
	int col = pos%6;
	if(store[row][col]!=2)
	{
		return backtrack(pos+1);
		//下一个位置	
	}
		for(int color = 0;color<=1;color++) //for循环开分支
	{
		//选中这个位置 
		store[row][col] = color;           //选择该位置后，判断valid
		if(isValid(row,col,color)){
			if(backtrack(pos+1))
			{
				//递归进行下一个 
				return 1;
			}
		}
		store[row][col] = 2;
		//回溯，撤销本次操作 
	} 
	return 0;
}
```


判断valid比较麻烦，是整个问题最struggle的部分

通过回溯的这种方式实际上化简了问题求解的复杂性，只需要在考虑**当前pos** 的可行性，
判断**放完当前之后**行（row）/列(col) 有没有超过三个，有没有连续三个。
以及当整行/列放完后for循环逐行与前面的对比，确保没有相同的行/列

```cpp
bool isValid(int row,int col,int color)
{
	// 检查行的黑白数量
	int row_white = 0, row_black = 0;
	for(int i = 0; i < 6; i++) {
    if(store[row][i] == 0) row_white++;
    else if(store[row][i] == 1) row_black++;
	}
	if(row_white > 3 || row_black > 3) return false;
	
	//检查列的黑白数量 
	int col_white = 0, col_black = 0;
	for(int i = 0;i<6;i++)
	{
	if(store[i][col]==0) col_white++;
	if(store[i][col]==1) col_black++;
	}
	if(col_white > 3 || col_black > 3) return false; 
	
	
	// 检查行是否有3个连续同色
	for(int i = 0; i < 4; i++) {  
    if(store[row][i] != 2 && 
       store[row][i] == store[row][i+1] && 
       store[row][i] == store[row][i+2]) {
        return false;
    }
	}

	// 检查列是否有3个连续同色
	for(int i = 0; i < 4; i++) {
    if(store[i][col] != 2 && 
       store[i][col] == store[i+1][col] && 
       store[i][col] == store[i+2][col]) {
        return false;
    }
	}
	
		// 检查行是否与之前某行重复
	if(row_black + row_white == 6){
	
	for(int r = 0; r < row; r++) {  // 只检查之前的行
	    bool same = true;
	    for(int i = 0; i < 6; i++) {
	        if(store[r][i] != store[row][i]) {
	            same = false;
	            break;
	        }
	    }
	    if(same) return false;  // 发现重复
	}
}


if(col_black+col_white==6){

	// 检查列是否与之前某列重复
	for(int c = 0; c < col; c++) {  // 只检查之前的列
	    bool same = true;
	    for(int i = 0; i < 6; i++) {
	        if(store[i][c] != store[i][col]) {
	            same = false;
	            break;
	        }
	    }
	    if(same) return false;  // 发现重复
	}
}

return 1;
}


```

③抽奖
简单题，难度与第一题相同

④判断节点颜色
```cpp

#include <iostream>
using namespace std;
#include<vector>
#include<math.h>
vector<int>store(30000,0);

int judge(int row,int n)

{
return store[pow(2,row-1)+n-1];
}
int main()
{
  store[1] = 1;
  for(int i = 2;i<30000;i++)
  {
    if(i%2==0)
      store[i] = store[i/2];
    else
    {
      store[i] = !store[i/2]; 
    }  
  }
  int all_num = 0; cin >> all_num;
  for(int i = 0; i < all_num;i++)
  {
    int row = 0; int n = 0; cin >> row >> n;
    if(judge(row,n))
    {
      cout<<"RED"<<endl;
    } 
    else
    {
      cout<<"BLACK"<<endl;
    }
  }
  return 0;

}
```

那部分分的技巧：看清楚数据的范围！满足前60% 80%的数据范围
转而使用递归的方式

```cpp
#include <iostream>
using namespace std;
int judge(int row,int n)

{
   if(row == 1 && n==1)
   {
     return 1;
   }
   else
   {
     if(n%2==1)
     return judge(row-1,n/2);
     else
     return !judge(row-1,n/2); 
   }
}
int main()
{
  int all_num = 0; cin >> all_num;
  for(int i = 0; i < all_num;i++)
  {
    int row = 0; int n = 0; cin >> row >> n;
    if(judge(row,n))
    {
      cout<<"RED"<<endl;
    } 
    else
    {
      cout<<"BLACK"<<endl;
    }
  }
  return 0;
}
```

上述递归方式会报内存超限，转为非递归方式：
最终解决方式：
```cpp
#include<iostream>
using namespace std;
typedef long long ll;
int judge(int row, ll n)
{
    //从2——>1如果被整除的话 就是同色的，不被整除就是异色的
    int count = 0;
    while (row != 1)
    {
        if ((n+1) % 2 != 0)
        {
            count++;
        } 
        n =(n+1) /2;
        row--;
    }
    return count % 2 == 0 ? 1 : 0;
}
 
int main()
{
    int all_num = 0;
    cin >> all_num;
    for (int i = 0; i < all_num; i++)
    {
        int row = 0;
        ll n = 0;
        cin >> row >> n;
        if (judge(row, n))
        {
            cout << "RED" << endl;
        }
        else
        {
            cout << "BLACK" << endl;
        }
    }
    return 0;
}
```

# 逆元相关问题

- 排列组合问题：
	正常数学：
	**N! ÷ a! ÷ b! ÷ c!**

- 模运算代码：
	f[N] × g[a] × g[b] × g[c]

如何计算逆元？
当 **p 是质数**，且 **x 不是 p 的倍数** 时：

$$x^{p−1}≡1(mod p)$$

**x 的 (p-1) 次方，模 p 一定等于 1**

$$x*x^{p−2}≡1(mod p)$$
**x的逆元是x的p-2次方**

$$inv(x)=x^{p−2}(modp)$$

举例子如下：
	让 p = 7（质数）
	求 **x = 2 的逆元**
	公式：
	inv (2) = 2^(7-2) = 2^5 = 32
	32 mod 7 = 4
	验证：
	2 × 4 = 8


$$(n-1)! = \frac{n!}{n}$$
两边取**逆元**：
$$\frac{1}{(n-1)!} = n*\frac{1}{n!}$$
翻译成代码变量
g[i-1]=(i)×g[i]%mod


预处理完毕后的计算技巧：关注核心计算部分
```cpp
#include <iostream>

using namespace std;

  

typedef long long LL;

const int N = 5e5 + 10, mod = 1e9 + 7;  // 数据范围与模数

  

LL n;                  // 输入的数字总个数

LL cnt[N];             // 统计每个数字出现的次数

LL f[N], g[N];         // f:阶乘数组  g:阶乘的逆元数组

  

// 快速幂：求 a^b % p

LL qpow(LL a, LL b, LL p)

{

    LL ret = 1;

    while(b)

    {

        if(b & 1) ret = ret * a % p;  // 二进制位为1，乘上当前a

        a = a * a % p;                // a平方

        b >>= 1;                      // b右移一位

    }

    return ret;

}

  

// 预处理：提前算出 1~5e5 的阶乘 和 阶乘逆元

void init()

{

    int max_n = 5e5;

    f[0] = 1;  // 0! = 1

  

    // 求阶乘 f[i] = i!

    for(int i = 1; i <= max_n; i++)

        f[i] = f[i - 1] * i % mod;

  

    // 费马小定理求最大阶乘的逆元

    g[max_n] = qpow(f[max_n], mod - 2, mod);

  

    // 倒推求所有阶乘逆元

    for(int i = max_n - 1; i >= 0; i--)

        g[i] = (i + 1) * g[i + 1] % mod;

}

  

int main()

{

    init();  // 预处理阶乘和逆元

  

    cin >> n;  // 输入数字总个数

  

    // 统计每个数字出现的次数

    for(int i = 1; i <= n; i++)

    {

        int x; cin >> x;

        cnt[x]++;

    }

  

    // 题目要求：选走两个数，剩下 n-2 个数

    LL real_n = n - 2;

  

    // 计算初始答案 S = (real_n)! / (cnt[1]! * cnt[2]! * ...)

    // 除法用乘逆元实现：S = f[real_n] * 乘积(g[cnt[i]])

    LL S = f[real_n];

    for(int i = 1; i < N; i++)

        S = S * g[cnt[i]] % mod;

  

    LL ans = 0;  // 最终答案

  

    // 试除法枚举 real_n 的所有约数对 (i, j)

    for(int i = 1; i <= real_n / i; i++)

    {

        if(real_n % i != 0) continue;  // 不是约数跳过

  

        LL j = real_n / i;  // i * j = real_n

  

        // 数组中没有 i 或 j，无法组成矩阵，跳过

        if(!cnt[i] || !cnt[j]) continue;

  

        // i == j 时，需要至少 2 个 i

        if(i == j && cnt[i] < 2) continue;

  

        // ---------------- 核心计算 ----------------

        // 选走一个 i 和一个 j 后的方案数

        // 公式：S * cnt[i] * cnt[j]

        // 代码用阶乘等价实现：S * f[cnt[i]] * f[cnt[j]] / ( (cnt[i]-1)! * (cnt[j]-1)! )

  

          if(i == j)

        {

            // 选两个相同的：C(cnt,2) = cnt*(cnt-1)/2

            LL add = S * cnt[i] % mod;

            add = add * (cnt[i]-1) % mod;

            ans = (ans+add)%mod;

        }

        else

        {

            // 直接乘！

            LL add = S * cnt[i] % mod;

            add = add * cnt[j] % mod;

  

            ans = (ans + add + add) % mod;

        }

    }

  

    cout << ans << endl;  // 输出总方案数

    return 0;

}
```

