# 算法比赛模板（一）


<!--more-->

## 工具类

### 线性筛素数

```c++
#include <iostream>
using namespace std;
#define SIZE 1000
int main()
{
	int	check[SIZE]	= { 0 };
	int	prime[SIZE]	= { 0 };
	int	pos		= 0;
	for ( int i = 2; i < SIZE; i++ )
	{
		/* 是素数 */
		if ( !check[i] )
			prime[pos++] = i;
		for ( int j = 0; j < pos && i * prime[j] < SIZE; j++ )
		{
			/* 筛掉 */
			check[i * prime[j]] = 1;
			if ( i % prime[j] == 0 )
				break;
		}
	}
	for ( int i = 0; prime[i] != 0; i++ )
		printf( "%d ", prime[i] );
	return(0);
}
```

### 以文本作为C++的数据输入

```c++
freopen( "DATA.in", "r", stdin );
freopen( "DATA.out", "w", stdout );
```

### 星期计算公式

```c++
#include <iostream>
using namespace std;


/*
 * 基姆拉尔森：w=(d+2*m+3*(m+1)/5+y+y/4-y/100+y/400)%7
 * w：0:星期一...依此类推
 */
int cal1( int y, int m, int d )
{
	if ( m == 1 || m == 2 )
		m += 12, y--;
	int w = (d + 2 * m + 3 * (m + 1) / 5 + y + y / 4 - y / 100 + y / 400) % 7;
	/* 结果加 1，返回 星期一：1 ——到星期日：7 */
	return(++w);
}


int main()
{
	int y, m, d;
	scanf( "%d %d %d", &y, &m, &d );
	int x = cal1( y, m, d );
	printf( "%dn", x );
	return(0);
}
```

### 大数阶乘（Java）

```c++
import java.io.*;
import java.util.*;
import java.math.BigInteger;
public class Main {
	public static void main( String[] args )
	{
		Scanner		cin	= new Scanner( System.in );
		int		n	= cin.nextint();
		BigInteger	ans	= BigInteger.ONE;
		for ( int i = 1; i <= n; i++ )
			ans = ans.multiply( BigInteger.valueOf( i ) );
		System.out.println( ans );
	}
}
```

### 全排列 函数

```c++
#include <iostream>
#include <algorithm>
using namespace std;

int main()
{
    int num[8] = { 1, 2, 3, 4, 5, 6, 7, 8 };
    do
    {
        cout    << num[0] << " " << num[1] << " " << num[2] << " " << num[3] << " " << num[4] << " "
            << num[5] << " " << num[6] << " " << num[7] << endl;
    }
    while ( next_permutation( num, num + 8 ) ); /* 全排列函数 */

    return(0);
}
```

## 经典例题

### 说反话

```c++
#include <iostream>
#include <stack>
using namespace std;
int main()
{
	stack<string>	v;
	string		s;
	while ( cin >> s )
		v.push( s );
	/* 入站 */
	cout << v.top();
	/* 站顶 */
	v.pop();
	/* 弹出站顶 */
	while ( !v.empty() )
	{
		cout << " " << v.top();
		v.pop();
	}
	return(0);
}
```

### 悄悄关注

```c++
/**
 * 新浪微博上有个“悄悄关注”，一个用户悄悄关注的人，不出现在这个用户的关注列表上，但系统会推送其悄悄关注的人发表的微博给该用户。现在我们来做一回网络侦探，根据某人的关注列表和其对其他用户的点赞情况，扒出有可能被其悄悄关注的人。
 * 输入格式：
 * 输入首先在第一行给出某用户的关注列表，格式如下：
 * 人数N 用户1 用户2 …… 用户N
 * 其中N是不超过5000的正整数，每个“用户i”（i=1, ..., N）是被其关注的用户的ID，是长度为4位的由数字和英文字母组成的字符串，各项间以空格分隔。
 * 之后给出该用户点赞的信息：首先给出一个不超过10000的正整数M，随后M行，每行给出一个被其点赞的用户ID和对该用户的点赞次数（不超过1000），以空格分隔。注意：用户ID是一个用户的唯一身份标识。题目保证在关注列表中没有重复用户，在点赞信息中也没有重复用户。
 * 输出格式：
 * 我们认为被该用户点赞次数大于其点赞平均数、且不在其关注列表上的人，很可能是其悄悄关注的人。根据这个假设，请你按用户ID字母序的升序输出可能是其悄悄关注的人，每行1个ID。如果其实并没有这样的人，则输出“Bing Mei You”。
 */
#include <iostream>
#include <algorithm>
#include <map>
using namespace std;

/* 结构体数组 */
struct pepole
{
    char    name[20];
    int    num;
}
peoples[10001];

/* 结构体排序 */
bool cmp( pepole x, pepole y )
{
    return(x.name > y.name); /*按姓名排序 */
}


int main()
{
    int            n, num;
    double            sum = 0;
    char            a[20];
    map<string, int>    m;

    cin >> n;
    for ( int i = 0; i < n; i++ )
    {
        cin >> a;
        m[a] = 1; /* map键值对赋值 */
    }

    cin >> num;
    for ( int i = 0; i < num; i++ )
    {
        scanf( "%s %d", &peoples[i].name, &peoples[i].num );
        sum += peoples[i].num;
    }
    sort( peoples, peoples + num, cmp ); /* 自定义排序 */
    sum /= num;

    int flag = 1;
    for ( int i = 0; i < num; i++ )
    {
        /* 大于平均值 且 不在关注列表中 */
        if ( peoples[i].num > sum && m[peoples[i].name] == 0 )
            printf( "%sn", peoples[i].name ), flag = 0;
    }
    if ( flag )
        printf( "Bing Mei Youn" );

    return(0);
}
```

### 括号配对
```c++
#include <iostream>
#include <stdio.h>
#include <string.h>
#include <stack>
using namespace std;
int main()
{
	int t, len;
	scanf( "%d", &t );
	getchar();
	while ( t-- )
	{
		char		s[10010];
		stack<char>	m;
		gets( s );
		len = strlen( s );
		m.push( s[0] );
		for ( int i = 1; i < len; i++ )
		{
			if ( m.size() == 0 )
				m.push( s[i] );
			else{
				if ( m.top() == '(' && s[i] == ')' )
					m.pop();
				else if ( m.top() == '[' && s[i] == ']' )
					m.pop();
				else
					m.push( s[i] );
			}
		}
		if ( m.empty() )
			printf( "Yes\n" );
		else
			printf( "No\n" );
	}
	return(0);
}
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
