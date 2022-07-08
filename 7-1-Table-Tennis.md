# 7-1-Table-Tennis
PTA Practice during extra semester

#### 题目内容：

A table tennis club has N tables available to the public. The tables are numbered from 1 to N. For any pair of players, if there are some tables open when they arrive, they will be assigned to the available table with the smallest number. If all the tables are occupied, they will have to wait in a queue. It is assumed that every pair of players can play for at most 2 hours.

Your job is to count for everyone in queue their waiting time, and for each table the number of players it has served for the day.

One thing that makes this procedure a bit complicated is that the club reserves some tables for their VIP members. When a VIP table is open, the first VIP pair in the queue will have the priviledge to take it. However, if there is no VIP in the queue, the next pair of players can take it. On the other hand, if when it is the turn of a VIP pair, yet no VIP table is available, they can be assigned as any ordinary players.

### Input Specification:

Each input file contains one test case. For each case, the first line contains an integer `N` (≤10000) - the total number of pairs of players. Then `N` lines follow, each contains 2 times and a VIP tag: `HH:MM:SS` - the arriving time, `P` - the playing time in minutes of a pair of players, and `tag` - which is 1 if they hold a VIP card, or 0 if not. It is guaranteed that the arriving time is between 08:00:00 and 21:00:00 while the club is open. It is assumed that no two customers arrives at the same time. Following the players' info, there are 2 positive integers: `K` (≤100) - the number of tables, and `M` (< K) - the number of VIP tables. The last line contains `M` table numbers.

### Output Specification:

For each test case, first print the arriving time, serving time and the waiting time for each pair of players in the format shown by the sample. Then print in a line the number of players served by each table. Notice that the output must be listed in chronological order of the serving time. The waiting time must be rounded up to an integer minute(s). If one cannot get a table before the closing time, their information must NOT be printed.

### Sample Input:

```
10
20:52:00 10 0
08:00:00 20 0
08:02:00 30 0
20:51:00 10 0
08:10:00 30 0
08:12:00 10 1
20:40:00 13 0
08:01:30 15 1
20:53:00 10 1
20:54:00 10 0
3 1
2
```

### Sample Output:

```
08:00:00 08:00:00 0
08:01:30 08:01:30 0
08:02:00 08:02:00 0
08:12:00 08:16:30 5
08:10:00 08:20:00 10
20:40:00 20:40:00 0
20:51:00 20:51:00 0
20:52:00 20:52:00 0
20:53:00 20:53:00 0
4 3 2
```

#### 解题思路：

某乒乓球场每天服务时间是8:00 -- 21:00，有·K个桌子（编号 1-K），其中M个为会员预留，
某一天，有N个玩家要来打球，给出了他们的到达时间，玩耍时间，和他们是否是会员，
要求，输出这些玩家的 到达时间 开始被服务时间 玩耍时间。输出每个桌子服务的人数。

#### 需要注意的是：

21:00及之后到来的玩家无法被服务，这些玩家不用考虑；
由于排队等待导致21:00前还未能被服务的玩家也要在输出中排除；
每个乒乓球求桌子限制最多一次服务2个小时；
所有的输出顺序要按照玩家开始被服务时间的先后顺序。
vip玩家的服务优先级高于普通玩家。当没有会员来时，vip桌子也为普通用户服务。

#### 思路解析：

本题的基本思想就是就是先来先服务，也就是所有玩家按照到达的先后顺序排序然后逐个处理，只不过**vip可以" 插队 "** 就导致分类讨论比较复杂一点。

#### 框架整理：

1.创建结构体 `Customer` 保存玩家的 到达时间、开始被服务时间、玩耍时间、是否是会员。

```c++
struct Customer
{
	int arriveTime, PlayTime, isVIP;//所有时间用秒来计算
	int startTime;
    //到达时间，玩耍时间，是否为vip，开始服务时间
}Customers[10010];
```

2.定义各项数据。

```c++
int N, K, M;//人数，桌数，vip桌数
int tables[110][2];//[0]球桌将被占用时长，[1]是否是vip桌
int served[110];//球桌的服务数量
int openTime = 8 * 3600;//从8点开门开始
int endTime = 21 * 3600;//21点球馆关门
```

3.排队函数，分别按到达时间和开始时间排队

```c++
bool ArrivalQueue(Customer a, Customer b)
{
	return a.arriveTime < b.arriveTime;//按到达时间排队
}
bool StartQueue(Customer a, Customer b)
{
	return a.startTime < b.startTime;//按开始时间排队
}
```

4.寻找空桌子

```c++
int findEmptyTable(int isVip)//寻找空桌子
{
	int i;
	int emptyTable = -1;//初始化
	for (i = 0; i < K; i++)
	{
		if (tables[i][0] == 0)//发现空桌子，要判断是不是VIP桌
		{
			if (isVip == 1 && tables[i][1] != 1)
				continue;//跳过vip桌，实在没桌子了再提供vip桌
			emptyTable = i;
			break;
		}
	}
	return emptyTable;
}
```

5.寻找来了的vip客户

```c++
int findVip(int start, int StartTime)//寻找来了的vip客户
{
	int j, vip = -1;
	for (j = start; j < N && Customers[j].arriveTime <= StartTime; j++)
	{
		if (Customers[j].isVIP)
		{
			vip = j;//是vip就返回，不是就继续循环
			break;
		}
	}
	return vip;
}
```

#### 完整代码：（有注释）

```c++
#include <bits/stdc++.h>
using namespace std;

struct Customer
{
	int arriveTime, PlayTime, isVIP;//所有时间用秒来计算
	int startTime;
    //到达时间，玩耍时间，是否为vip，开始服务时间
}Customers[10010];

int N, K, M;//人数，桌数，vip桌数
int tables[110][2];//[0]球桌将被占用时长，[1]是否是vip桌
int served[110];//球桌的服务数量
int openTime = 8 * 3600;//从8点开门开始
int endTime = 21 * 3600;//21点球馆关门
bool ArrivalQueue(Customer a, Customer b)
{
	return a.arriveTime < b.arriveTime;//按到达时间排队
}
bool StartQueue(Customer a, Customer b)
{
	return a.startTime < b.startTime;//按开始时间排队
}
int findEmptyTable(int isVip)//寻找空桌子
{
	int i;
	int emptyTable = -1;//初始化
	for (i = 0; i < K; i++)
	{
		if (tables[i][0] == 0)//发现空桌子，要判断是不是VIP桌
		{
			if (isVip == 1 && tables[i][1] != 1)
				continue;//跳过vip桌，实在没桌子了再提供vip桌
			emptyTable = i;
			break;
		}
	}
	return emptyTable;
}
int findVip(int start, int StartTime)//寻找来了的vip客户
{
	int j, vip = -1;
	for (j = start; j < N && Customers[j].arriveTime <= StartTime; j++)
	{
		if (Customers[j].isVIP)
		{
			vip = j;//是vip就返回，不是就继续循环
			break;
		}
	}
	return vip;
}
int main()
{
	int i, j, h, m, s;
	cin >> N;
	for (i = 0; i < N; i++)
	{
		//格式化输入，scanf优于cin
		scanf("%d:%d:%d %d %d", &h, &m, &s, &Customers[i].PlayTime, &Customers[i].isVIP);
		Customers[i].arriveTime = h * 3600 + m * 60 + s;//记录到达时间
		Customers[i].startTime = endTime + 1;//记录开始时间
		if(Customers[i].PlayTime > 120)
            Customers[i].PlayTime = 120;//时间不超过两小时，超过两小时以两小时计算
		Customers[i].PlayTime *= 60;//按秒计算
	}
	cin >> K >> M;//K个球桌，M个VIP球桌
	for (i = 0; i < K; i++)
		tables[i][0] = tables[i][1] = served[i] = 0;//初始化球桌，全部为未服务状态
	for (i = 0; i < M; i++)
	{
		cin >> j;
		tables[j - 1][1] = 1;//读入vip桌子
	}
	sort(Customers, Customers + N, ArrivalQueue);//按到达时间排序
	int emptyTable, vipTable, fastestTable, vip;
	for (i = 0; i < N; i++)
	{	
		if (Customers[i].arriveTime > openTime)
		{
			for (j = 0; j < K; j++)
			{
				tables[j][0] -= (Customers[i].arriveTime - openTime);
                //所有table经过arriveTime-openTime时长
				if (tables[j][0] < 0)
					tables[j][0] = 0;//服务结束了，将桌子置于空闲
			}
			openTime = Customers[i].arriveTime;//如果当前时刻该顾客还未到，时间调到顾客到达时间
		}
		//寻找最快结束的桌子
		fastestTable = 0;
		for (j = 1; j < K; j++)
		{
			if (tables[j][0] < tables[fastestTable][0])
				fastestTable = j;
		}
		//时间调到最快结束桌子的结束时间
		openTime += tables[fastestTable][0];
		if (openTime >= endTime)//当有空桌时，已关门，后面的人不管了
		{
			N = i;
			break;
		}
		int t = tables[fastestTable][0];
		for (j = 0; j < K; j++)
			tables[j][0] -= t;
		//最快的桌子结束了，现在必有空桌

		vipTable = findEmptyTable(1);//isvip=1时说明有vip顾客
		vip = findVip(i, openTime);//vip顾客此时在等待
		//有vip空桌时且当前时间有vip顾客,vip顾客优先
		if (vipTable != -1 && vip != -1)
		{
			if (i != vip)
				for (j = vip; j > i; j--)
					swap(Customers[j], Customers[j - 1]);
			emptyTable = vipTable;
		}
		else//当前时刻，vip空桌和vip顾客不同时存在
		{
			emptyTable = findEmptyTable(0);
			if (emptyTable == -1)
				while (1);
		}
		Customers[i].startTime = openTime;//重置时间，再循环执行上述代码
		tables[emptyTable][0] = Customers[i].PlayTime;
		served[emptyTable]++;//桌子的服务人数+1
	}
	int wait;
	for (i = 0; i < N; i++)
	{
		if (Customers[i].startTime >= endTime)
			while (1);
		//格式化输出，该题目对格式化要求较高，printf优于cout输出
		printf("%02d:%02d:%02d %02d:%02d:%02d", Customers[i].arriveTime / 3600, (Customers[i].arriveTime / 60) % 60, Customers[i].arriveTime % 60, Customers[i].startTime / 3600, (Customers[i].startTime / 60) % 60, Customers[i].startTime % 60);
		wait = Customers[i].startTime - Customers[i].arriveTime;//顾客等待的时间
		wait = wait / 60 + (wait % 60) / 30;//时间格式化
		cout << ' ' << wait << endl;
	}
	for (i = 0; i < K - 1; i++)
		cout << served[i] << ' ';
	cout << served[i] << endl;//输出球桌服务的顾客数量
	return 0;
}


```



