# 数字三角形_POJ1163

## 1.  [题目链接](http://poj.org/problem?id=1163)
<font size=4>
The Triangle


| **Time Limit:** 1000MS       |      | **Memory Limit:** 10000K |
| ---------------------------- | ---- | ------------------------ |
| **Total Submissions:** 64081 |      | **Accepted:** 38340      |

Description

![image](F7AA902B42FB4E5999376F0221E44663)

(Figure 1)

Figure 1 shows a number triangle. Write a program that calculates the highest sum of numbers passed on a route that starts at the top and ends somewhere on the base. Each step can go either diagonally down to the left or diagonally down to the right.

Input

Your program is to read from standard input. The first line contains one integer N: the number of rows in the triangle. The following N lines describe the data of the triangle. The number of rows in the triangle is > 1 but <= 100. The numbers in the triangle, all integers, are between 0 and 99.

Output

Your program is to write to standard output. The highest sum is written as an integer.

Sample Input

```
5
7
3 8
8 1 0 
2 7 4 4
4 5 2 6 5
```

Sample Output

```
30
```

Source

[IOI 1994](http://poj.org/searchproblem?field=source&key=IOI+1994)

## 2.  解题思路

用二维数组a存放数字三角形，二维数组maxSum保存已经计算出的结果（记忆化，否则会超时）

a[i] [j]：第 i 行第 j 个数字（i, j从1开始算）

maxSum[i] [j]：从a[i] [j] 到底边的各条路径中，最佳路径的数字之和。

问题：求maxSum(1, 1)

从a[i] [j] 出发，下一步只能走a[i+1] [j] 或者a[i+1] [j+1]。故对于N行的三角形：

~~~java
if (i == n) {
    maxSum[i][j] = a[i][j];
} else {
    int a = maxSum(i + 1, j);
    int b = maxSum(i + 1, j + 1);
    maxSum[i][j] = Math.max(a, b) + a[i][j];
}
~~~

### 3. 记忆递归性动归算法：

~~~java
import java.util.Scanner;

public class Main {
    private static int n;
    private static int[][] a;
    private static int[][] maxSum;

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        n = sc.nextInt();
        a = new int[n + 1][];
        maxSum = new int[n + 1][n + 1];
        for (int i = 1; i <= n ; i++) {
            a[i] = new int[i + 1];
            for (int j = 1; j <= i; j++) {
                a[i][j] = sc.nextInt();
            }
        }
        System.out.println(maxSum(1, 1));
    }

    private static int maxSum(int i, int j) {
        if (maxSum[i][j] != 0) {
            return maxSum[i][j];
        }
        if (i == n) {
            maxSum[i][j] = a[i][j];
        } else {
            int x = maxSum(i + 1, j);
            int y = maxSum(i + 1, j + 1);
            maxSum[i][j] = Math.max(x, y) + a[i][j];
        }
        return maxSum[i][j];
    }
}
~~~

## 4. 解题思路2（递归转成递推）

maxSum数组的最后一行值等于a数组最后一行的值，故用双重for循环，从倒数第二行开始往第一行进行递推，递归后maxSum[1] [1] 即为所求的值。

## 5. 递归转成递推算法：

~~~java
import java.util.Scanner;

public class Main {
    private static int n;
    private static int[][] a;
    private static int[][] maxSum;

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        n = sc.nextInt();
        a = new int[n + 1][];
        maxSum = new int[n + 1][n + 1];
        for (int i = 1; i <= n ; i++) {
            a[i] = new int[i + 1];
            for (int j = 1; j <= i; j++) {
                a[i][j] = sc.nextInt();
            }
        }
        for (int i = 1; i <= n; i++) {
            maxSum[n][i] = a[n][i];
        }
        for (int i = n - 1; i >= 1; i--) {
            for (int j = 1; j <= i; j++) {
                int x = maxSum[i+1][j];
                int y = maxSum[i+1][j+1];
                maxSum[i][j] = Math.max(x, y) + a[i][j];
            }
        }
        System.out.println(maxSum[1][1]);
    }
}
~~~

