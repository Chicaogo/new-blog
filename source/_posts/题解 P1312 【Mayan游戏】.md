---
title: 题解 P1312 【Mayan游戏】
date: 2018/11/7
categories: 题解
tag: DFS剪枝
---

# 题解 P1312 【Mayan游戏】

## 题面

**[过长已遮挡](https://www.luogu.org/problemnew/show/P1312)**

## 题意

**体面已经陈述题意**（这题没有考语文阅读理解）

## 题解

** 我还记得我曾经给自己找的锅，给某些人讲课的时候说过一句话：体面越长的题，越简单。** *这句话没有错，我会用接下来解决这道题的思路过程，来证明这句话。*

1. 首先我们知道存在这么几种操作：

  a. 交换操作

    b. 下沉操作

    c. 消除操作

    d. 搜索操作

    e. 玩完判断

    f. 拷贝操作（回溯）

2. 这时候可以看得出来这里面除了搜索操作，剩余的操作都是模拟，我们只需要分出多个函数进行模拟就好了。

3. 但是思路就是如此简单的一道题我却调了一个下午。（代码量是真的大，不压横150+行代码），还是我太菜了。

4. 因为数据量非常小，不用怎么优化也能过，但是写代码的时候一定要小心。

---

<!--more-->

### 下沉操作

每次交换完 or 消除完都要进行的操作。

1. 遍历图上的每一个点，如果有颜色，判断下面是否还有颜色

2. 没有颜色：立一个$flag$，向下一直找到呢个有颜色的点，然后利用$flag$下沉到有显色点的上方。

3. 有颜色：跳出当前列。

```cpp
void update()
{
    for(int i = 1;i <= 5;++i)
    {
        int flag = 0;
        for(int j = 1;j <= 7;++j)
        {
            if(!mapp[i][j]) flag++;
            else
            {
                if(!flag) continue;
                mapp[i][j - flag] = mapp[i][j];
                mapp[i][j] = 0;
            }
        }
    }
}
```

### 消除操作

每次下沉完都要进行的操作。根据题意会有三种情况：只有横着的三个；只有竖着的三个；横着三个交叉在竖着三个之间；横着三个和竖着三个同时出现但不相交。

但是我们可以把它们归纳到一种情况，就是只要有三个连续相同的出现就加到桶里面，然后再把桶扫一遍，全部染回没有颜色，这样就消除了。

这里呢还可以加一个小优化，立一个$flag$。因为根据题意，消除完还要进行下沉操作，但是如果我们没有可消除，就浪费了下沉的一些时间。

代码操作非常简单，只要根据上面的分析进行模拟就好了。

```cpp
bool reover()
{
    bool flag = 1;
    for(int i = 1;i <= 5;++i)
        for(int j = 1;j <= 7;++j)
        {
            if(i-1 >= 1 && i+1 <= 5 && mapp[i][j] && mapp[i][j] == mapp[i-1][j] && mapp[i][j] == mapp[i+1][j])
            {
                tong[i][j] = 1;
                tong[i-1][j] = 1;
                tong[i+1][j] = 1;
                flag = 0;
            }
            if(j-1 >= 1 && j+1 <= 7 && mapp[i][j] && mapp[i][j] == mapp[i][j-1] && mapp[i][j] == mapp[i][j+1])
            {
                tong[i][j] = 1;
                tong[i][j-1] = 1;
                tong[i][j+1] = 1;
                flag = 0;
            }
        }
    
    if(flag) return 0;

    for(int i = 1;i <= 5;++i)
        for(int j = 1;j <= 7;++j)
        {
            if(tong[i][j]) 
            {
                mapp[i][j] = 0;
                tong[i][j] = 0;
            }
        }
    
    return 1;
}
```

### 交换操作

根据题意可得，交换的流程是这样的：

交换两个点颜色 -> 下沉操作 -> 消除操作 -> 下沉操作 -> 消除操作

一直到不能再消除，交换结束。

```cpp
void move(int i,int j,int flag)
{
    swap(mapp[i][j],mapp[i+flag][j]);
    update();
    while(reover()) update();
}
```

### 玩完判断

只需要扫描最后一层，如果还有颜色，肯定是没有玩完。

```cpp
bool gameover()
{
    for(int i = 1;i <= 5;++i)
        if(mapp[i][1]) return 0;
    
    return 1;
}
```

### 拷贝操作（回溯）

留一个备份是为了方便我们搜索完一轮后，返回时恢复初值。

代码非常的简单容易理解。

```cpp
void move(int i,int j,int flag)
{
    swap(mapp[i][j],mapp[i+flag][j]);
    update();
    while(reover()) update();
}
```

### 搜索操作

这一块是最容易出错的，但是细心一些还是能过得，毕竟这个搜索也不需要优化。

1. 如果已经玩完了，直接输出答案，结束程序。

2. 如果搜索长度大于等于n了，不用再往后面搜了。

3. 留一个备份，方便一会回溯。

4. 1和2都不成立，就继续搜。分两种情况，左搜和右搜。

  a. 左搜：首先左边界肯定要大于等于1，左边的方块还没有颜色。

    b. 右搜：首先右边界肯定要小于等于5，相邻两颜色还不能相同。

5. **搜完一定要记得回溯**

```cpp
void dfs(int x)
{
    if(gameover())
    {
        for(int i = 1;i <= n;++i)
        {
            if(i != 1) cout << endl;
            cout << ans[i][1] <<" "<< ans[i][2] <<" "<< ans[i][3];
        }
        exit(0);
    }

    if(x == n+1) return;

    copy(x);

    for(int i = 1;i <= 5;++i)
        for(int j = 1;j <= 7;++j)
        {
            if(mapp[i][j])
            {
                if(i-1 >= 1 && mapp[i-1][j] == 0)
                {
                    move(i,j,-1);
                    ans[x][1] = i-1;
                    ans[x][2] = j-1;
                    ans[x][3] = -1;
                    
                    dfs(x+1);

                    for(int i = 1;i <= 5;++i)
                        for(int j = 1;j <= 7;++j)
                            mapp[i][j] = last[x][i][j];
                    
                    ans[x][1] = 0;
                    ans[x][2] = 0;
                    ans[x][3] = 0;
                }

                if(i+1 <= 5 && mapp[i][j] != mapp[i+1][j])
                {
                    move(i,j,1);
                    ans[x][1] = i-1;
                    ans[x][2] = j-1;
                    ans[x][3] = 1;
                    
                    dfs(x+1);

                    for(int i = 1;i <= 5;++i)
                        for(int j = 1;j <= 7;++j)
                            mapp[i][j] = last[x][i][j];
                    
                    ans[x][1] = 0;
                    ans[x][2] = 0;
                    ans[x][3] = 0;
                }
            }
        }
}
```

## 代码

这道题的代码挺考察耐心的，我调了两个小时才调出来AC100分的结果。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define il inline

int n;
int mapp[10][10],ans[10][10],last[10][10][10],tong[10][10];

il int read()
{
    int X=0,w=0; char ch=0;
    while(!isdigit(ch)) {w|=ch=='-';ch=getchar();}
    while(isdigit(ch)) X=(X<<3)+(X<<1)+(ch^48),ch=getchar();
    return w?-X:X;
}

il bool reover()
{
    bool flag = 1;
    for(int i = 1;i <= 5;++i)
        for(int j = 1;j <= 7;++j)
        {
            if(i-1 >= 1 && i+1 <= 5 && mapp[i][j] && mapp[i][j] == mapp[i-1][j] && mapp[i][j] == mapp[i+1][j])
            {
                tong[i][j] = 1;
                tong[i-1][j] = 1;
                tong[i+1][j] = 1;
                flag = 0;
            }
            if(j-1 >= 1 && j+1 <= 7 && mapp[i][j] && mapp[i][j] == mapp[i][j-1] && mapp[i][j] == mapp[i][j+1])
            {
                tong[i][j] = 1;
                tong[i][j-1] = 1;
                tong[i][j+1] = 1;
                flag = 0;
            }
        }
    
    if(flag) return 0;

    for(int i = 1;i <= 5;++i)
        for(int j = 1;j <= 7;++j)
        {
            if(tong[i][j]) 
            {
                mapp[i][j] = 0;
                tong[i][j] = 0;
            }
        }
    
    return 1;
}

il bool gameover()
{
    for(int i = 1;i <= 5;++i)
        if(mapp[i][1]) return 0;
    
    return 1;
}

il void copy (int x)
{
    for(int i = 1;i <= 5;++i)
        for(int j = 1;j <= 7;++j)
        {
            last[x][i][j] = mapp[i][j];
        }
}

il void update()
{   
    for(int i = 1;i <= 5;++i)
    {
        int flag = 0;
        for(int j = 1;j <= 7;++j)
        {
            if(!mapp[i][j]) flag++;
            else
            {
                if(!flag) continue;
                mapp[i][j - flag] = mapp[i][j];
                mapp[i][j] = 0;
            }
        }
    }
}

il void move(int i,int j,int flag)
{
    swap(mapp[i][j],mapp[i+flag][j]);
    update();
    while(reover()) update();
}

void dfs(int x)
{
    if(gameover())
    {
        for(int i = 1;i <= n;++i)
        {
            if(i != 1) cout << endl;
            cout << ans[i][1] <<" "<< ans[i][2] <<" "<< ans[i][3];
        }
        exit(0);
    }

    if(x == n+1) return;

    copy(x);

    for(int i = 1;i <= 5;++i)
        for(int j = 1;j <= 7;++j)
        {
            if(mapp[i][j])
            {
                if(i-1 >= 1 && mapp[i-1][j] == 0)
                {
                    move(i,j,-1);
                    ans[x][1] = i-1;
                    ans[x][2] = j-1;
                    ans[x][3] = -1;
                    
                    dfs(x+1);

                    for(int i = 1;i <= 5;++i)
                        for(int j = 1;j <= 7;++j)
                            mapp[i][j] = last[x][i][j];
                    
                    ans[x][1] = 0;
                    ans[x][2] = 0;
                    ans[x][3] = 0;
                }

                if(i+1 <= 5 && mapp[i][j] != mapp[i+1][j])
                {
                    move(i,j,1);
                    ans[x][1] = i-1;
                    ans[x][2] = j-1;
                    ans[x][3] = 1;
                    
                    dfs(x+1);

                    for(int i = 1;i <= 5;++i)
                        for(int j = 1;j <= 7;++j)
                            mapp[i][j] = last[x][i][j];
                    
                    ans[x][1] = 0;
                    ans[x][2] = 0;
                    ans[x][3] = 0;
                }
            }
        }
}

int main(int argc, char const *argv[])
{
    n = read();

    for(int i = 1;i <= 5;++i)
        for(int j = 1;j <= 8;++j)
        {
            int x = read();
            if(x == 0) break; 
            mapp[i][j] = x;
        }

    dfs(1);

    printf("-1\n");

    return 0;
}

```