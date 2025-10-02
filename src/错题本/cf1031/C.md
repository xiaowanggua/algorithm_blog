# C. Smilo and Minecraft

The boy Smilo is playing Minecraft! To prepare for the battle with the dragon, he needs a lot of golden apples, and for that, he requires a lot of gold. Therefore, Smilo goes to the mine.

The mine is a rectangular grid of size n×m, where each cell can be either gold ore, stone, or an empty cell. Smilo can blow up dynamite in any empty cell. When dynamite explodes in an empty cell with coordinates (x,y), all cells within a square of side 2k+1 centered at cell (x,y) become empty. If gold ore was located **strictly inside** this square (not on the boundary), it disappears. However, if the gold ore was on the boundary of this square, Smilo collects that gold.

Dynamite can only be detonated inside the mine, but the explosion square can extend beyond the mine's boundaries.

Determine the maximum amount of gold that Smilo can collect.

**Input**

Each test contains multiple test cases. The first line contains the number of test cases t (1≤t≤104). The description of the test cases follows.

The first line of each test case contains three integers n, m, and k (1≤n,m,k≤500) — the number of rows, columns, and the explosion parameter k, respectively.

Each of the following n lines contains m characters, each of which is equal to '.', '#', or 'g', where '.' — is an empty cell, '#' — is stone, 'g' — is gold. It is guaranteed that at least one of the cells is empty.

It is guaranteed that the sum n⋅m across all test cases does not exceed 2.5⋅105.

**Output**

For each test case, output a single integer — the maximum amount of gold that can be obtained.

**Example**

Input

```
3
2 3 1
#.#
g.g
2 3 2
#.#
g.g
3 4 2
.gg.
g..#
g##.
```

Output

```
2
0
4
```

-----

### 解题思路
​	该题是一个贪心题，只需要找到所有初始空位对应矩形范围内销毁的金块中的最小值。然后在初始爆炸的范围内进行平滑移动进行爆破即可无损耗的得到剩余所有金块，即$Count_{Gold} - min_{destoryedGlod}$，使用二维前缀和进行求解。

### 题解
```c++
#include<iostream>
#include<vector>
using namespace std;
int clamp(int num,int max){
	return (num < 1?0:(num > max?max:num));
}
int query(vector<vector<int>>& numbers,int x,int y,int k,int n,int m){
	int x1 = clamp(x + k,n); int y1 = clamp(y + k,m);
	int x2 = clamp(x - k - 1,n); int y2 = clamp(y - k - 1,m);
	int x3 = clamp(x + k,n); int y3 = clamp(y - k - 1,m);
	int x4 = clamp(x - k - 1,n); int y4 = clamp(y + k,m);
	return numbers[x1][y1] + numbers[x2][y2] - numbers[x3][y3] - numbers[x4][y4];
}
int main(){
	int T;
	cin >> T;
	while(T-->0){
		int n,m,k;
		cin >> n >> m >> k;
		vector<vector<int>> ps(n+1,vector<int>(m+1));
		vector<pair<int,int>> empty;
		for(int i = 1;i <= n;i++){
			for(int j = 1;j <= m;j++)
			{
				char temp;
				cin >> temp;
				if(temp == '.'){
					ps[i][j] = ps[i][j-1] + ps[i-1][j] - ps[i-1][j-1]; 
					empty.emplace_back(make_pair(i,j));
				}else if(temp == 'g'){
					ps[i][j] = 1 + ps[i][j-1] + ps[i-1][j] - ps[i-1][j-1];
				}else{
					ps[i][j] = ps[i][j-1] + ps[i-1][j] - ps[i-1][j-1]; 
				}
			}	
		}
		int min = 2000000;
		for(auto& [x,y]:empty){
			int temp = query(ps,x,y,k-1,n,m);
			if(temp < min){
				min = temp;
			}
		}
		cout << ps[n][m] - min << endl;
	}
}
```

