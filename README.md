# Interesting_Problems
一些有意思的问题
===============

1.leetcode #877 Stone Game
-------------------------

Alex and Lee play a game with piles of stones.  There are an even number of piles arranged in a row, and each pile has a positive integer number of stones piles[i].

The objective of the game is to end with the most stones.  The total number of stones is odd, so there are no ties.

Alex and Lee take turns, with Alex starting first.  Each turn, a player takes the entire pile of stones from either the beginning or the end of the row.  This continues until there are no more piles left, at which point the person with the most stones wins.

Assuming Alex and Lee play optimally, return True if and only if Alex wins the game.

Example 1:

Input: [5,3,4,5]
Output: true
Explanation: 
Alex starts first, and can only take the first 5 or the last 5.
Say he takes the first 5, so that the row becomes [3, 4, 5].
If Lee takes 3, then the board is [4, 5], and Alex takes 5 to win with 10 points.
If Lee takes the last 5, then the board is [3, 4], and Alex takes 4 to win with 9 points.
This demonstrated that taking the first 5 was a winning move for Alex, so we return true.
 

Note:

2 <= piles.length <= 500
piles.length is even.
1 <= piles[i] <= 500
sum(piles) is odd.

---------------
* 解法1 动态规划

因为探索空间比较大，遍历所有可能性会超时，考虑使用dp来存储已计算的状态

dp[i][j]表示在下标i到j之间，先手玩家可以比后手玩家获得的更多的石头。
这句话听起来比较拗口，公式表达会更简洁一些：
```
如果先手玩家选择piles[i]，那么有
  dp[i][j]= 先手玩家获得的石头-后手玩家获得的石头
          = { dp[i](先手玩家拿了piles[i]) + 先手玩家在i+1到j之间能拿到的所有石头 } - 后手玩家在i+1到j之间能拿到的所有石头
          = piles[i] - 后手玩家在i+1到j之间能拿到的所有石头 + 先手玩家在i+1到j之间能拿到的所有石头
          = piles[i] - (后手玩家在i+1到j之间能拿到的所有石头 - 先手玩家在i+1到j之间能拿到的所有石头)
          = piles[i] - dp[i+1][j]
同理如果先手玩家选择piles[j],那么有
  dp[i][j]= piles[j] - dp[i][j-1]

显然的，由于游戏里两名玩家都是尝试选择最优的方式赢得游戏，于是有
dp[i][j]=Math.max(piles[i] - dp[i+1][j], piles[j] - dp[i][j-1])
```

有了这个递推方程，我们的解法1就呼之欲出了：
```Java
    public boolean stoneGame(int[] piles){
        int [][] dp=new int[piles.length][piles.length];
        //从左上角出发，斜向折叠
        for (int j=0;j<piles.length;j++){
            for (int i=j;i>=0;i--){
            		if (i==j) dp[i][j]=piles[i];
            		else
            			dp[i][j]=Math.max(piles[i]-dp[i+1][j],piles[j]-dp[i][j-1]);
            }
        }
        return dp[0][piles.length-1]>0;
        
    }
```

值得注意的是这里的填充顺序，假设dp是一个4x4的矩阵，斜向从左上角开始折返填充,这里是 0 -> 5 -> 1 -> 10 -> 6 -> 2 -> 15 -> 11 -> 7 -> 3
```
0  1  2  3 
4  5  6  7
8  9  10 11
12 13 14 15
```
由于3是最后一个求的，于是显然有另一个顺序从右下角开始 15 -> 10 -> 11 ->5 -> 6 -> 7 -> 0 -> 1 -> 2 -> 3
可以写成
```Java      
        //从右下角出发，斜向折叠
        for (int i=piles.length-1;i>=0;i--){
        	for (int j=i;j<piles.length;j++){
        		if (i==j) dp[i][j]=piles[i];
        		else
        			dp[i][j]=Math.max(piles[i]-dp[i+1][j],piles[j]-dp[i][j-1]);
        	}
        }
```

存储空间进一步优化为一维数组的版本如下
```Java
    public boolean stoneGame(int[] piles){
    	int [] d=new int[piles.length];
        System.arraycopy(piles, 0, d, 0, piles.length);
        for (int j=0;j<piles.length;j++){
            for (int i=j;i>=0;i--){
        		if (i!=j)
        			d[i]=Math.max(piles[i]-d[i+1],piles[j]-d[i]);
        	}
        }
        return d[0]>0;
    }
```
--------------
* 解法2 递归+memoization
```Java
    public boolean stoneGame(int[] piles) {
    	cache=new int[piles.length][piles.length];
    	int sum=0;
    	
    	for (int i=0;i<piles.length;i++){
    		for (int j=0;j<piles.length;j++){
    			cache[i][j]=-1;
    		}
    		sum+=piles[i];
    	}
    		
    	int alex=maxStoneBetween(piles,0,piles.length-1);
    	int lee=sum-alex;
    	return alex>lee;
    }
    
    
    public int maxStoneBetween(int[] a, int head, int tail){
    	if (head>=tail){
    		return 0;
    	}else if (tail==head+1 && tail<a.length){
    		if (a[head]>a[tail]) return a[head];
    		else return a[tail];
    	}else{
    		if (cache[head][tail]!=-1) return cache[head][tail];
    		int res=Math.max(a[head]+Math.min(maxStoneBetween(a,head+2,tail),maxStoneBetween(a,head+1,tail-1)),
    						 a[tail]+Math.min(maxStoneBetween(a,head+1,tail-1), maxStoneBetween(a,head,tail-2)));
    		cache[head][tail]=res;
    		return res;
    	}
    }
```
--------------
* 解法3 数学

题中给出条件，
>石头的堆数是偶数堆，说明最终两个人可以拿到同样堆数的石头；
>石头的总数为基数，说明两个人拿到的两堆石头一定不会相等，因为显然 ``奇数 mod 2!=0``
>而玩家1可以保证先手，说明玩家1可以主动选择从奇数位开始拿并且拿走所有奇数位的石头堆，
                                   >或者从偶数位开始拿并且拿走所有偶数位的石头堆。
例如: 有石头堆
```
    [ 5 , 3 , 4 , 5 ]
位数  1   2   3   4
```
先手玩家如果选择拿所有奇数位的石头堆，那么他可以拿走1号位的'5'，盘面变成
```
    [   , 3 , 4 , 5 ]
位数  1   2   3   4
```
此时后手玩家无论选择2号位的'3'或者4号位的'5'都无法阻止先手玩家继续拿奇数位3号位的'4'

同理，先手玩家如果选择拿所有偶数位的石头堆，那么他可以拿走4号位的'5'
```
    [ 5 , 3 , 4 ,   ]
位数  1   2   3   4
```
此时后手玩家无论选择1号位的'5'或者3号位的'4'都无法阻止先手玩家继续拿偶数位2号位的'3'
不过在这个例子里选择拿偶数位的石头堆会输，所以先手玩家会选择奇数堆的石头

由此可知先手玩家的必胜法只需要预先计算所有奇数位堆的石头总数和所有偶数位堆的石头总数，然后选择去拿走总数更多的那一种拿法即可(拿走所有奇数位或者偶数位)
也就是说先手玩家必胜，于是有：
```Java
public boolean stoneGame(int[] piles){
    return true;
}
```

