# Interesting_Problems
一些有意思的问题

1. leetcode #877 Stone Game

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

解法1：
动态规划
因为探索空间比较大，遍历所有可能性会超时，考虑使用dp来存储已计算的状态

dp[i][j]表示在下标i到j之间，先手玩家可以比后手玩家获得的更多的石头。
这句话听起来比较拗口，公式表达会更简洁一些：
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

有了这个递推方程，我们的解法1就呼之欲出了：

boolean stoneGame(int[] piles){
  int [][] dp=new int[piles.length][piles.length];  
  for (int j=0;j<piles.length;j++){
       for (int i=j;i>=0;i--){
          if (i==j) dp[i][j]=piles[i];
          else
              dp[i][j]=Math.max(piles[i]-dp[i+1][j],piles[j]-dp[i][j-1]);
       }
  }
  return dp[0][piles.length-1]>0;
} 

值得注意的是这里的填充顺序，斜向从左上角开始折返填充,这里是 0 -> 5 -> 1 -> 10 -> 6 -> 2 -> 15 -> 11 -> 7 -> 3
0  1  2  3 
4  5  6  7
8  9  10 11
12 13 14 15

由于3是最后一个求的，于是显然有另一个顺序从右下角开始 15 -> 10 -> 11 ->5 -> 6 -> 7 -> 0 -> 1 -> 2 -> 3
可以写成
    for (int i=piles.length-1;i>=0;i--){
        	for (int j=i;j<piles.length;j++){
        		 if (i==j) dp[i][j]=piles[i];
        		 else
        			    dp[i][j]=Math.max(piles[i]-dp[i+1][j],piles[j]-dp[i][j-1]);
        	}
    }

解法2
递归+memoization

解法3
数学
