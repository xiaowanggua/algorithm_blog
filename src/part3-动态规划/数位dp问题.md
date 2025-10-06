# 数位 DP 入门指北（记忆化搜索版）

> 数位 DP 主要解决一类形如  
> **“满足 xxx 条件的数是好数，求 ≤ n 的好数有多少个”**  
> 的问题。  
> xxx 条件通常是「数字在 x 进制表示下具有某种性质」：  
> 不存在重复数字 / 数位和在给定区间 / 不能出现 0 ……


## 1. 核心思路

1. 用**记忆化 DFS** 逐位枚举，**从高位往低位**扫；  
2. 必带两个状态：  
   - `same`：前面是否**与原数 n 完全一致**；  
   - `zeros`：前面是否**全是前导 0**；  
3. 根据题意再挂一个 `curr` 记录“与题目相关的数位性质”（二进制压位、数位和、前一位数字 …）。  
4. 记忆化键值 = `(pos, curr)`，注意 `same==true` 或 `zeros==true` 时状态不可复用。
5. **不要**想着刷表了
6. 部分问“`[a，b]`区间之间的好数有多少”的题目，拆成两次小于等于，然后做差即可。

## 2. 例题 1：LeetCode 902 最大为 N 的数字组合
给定一个按 **非递减顺序** 排列的数字数组 `digits` 。你可以用任意次数 `digits[i]` 来写的数字。例如，如果 `digits = ['1','3','5']`，我们可以写数字，如 `'13'`, `'551'`, 和 `'1351315'`。

返回 **可以生成的小于或等于给定整数 `n` 的正整数的个数** 。


示例 1：

输入：`digits = ["1","3","5","7"]`, `n = 100`
输出：`20`
解释：
可写出的 20 个数字是：
1, 3, 5, 7, 11, 13, 15, 17, 31, 33, 35, 37, 51, 53, 55, 57, 71, 73, 75, 77.
示例 2：

输入：`digits = ["1","4","9"]`, `n = 1000000000`
输出：`29523`
解释：
我们可以写 3 个一位数字，9 个两位数字，27 个三位数字，
81 个四位数字，243 个五位数字，729 个六位数字，
2187 个七位数字，6561 个八位数字和 19683 个九位数字。
总共，可以使用`D`中的数字写出 29523 个整数。
示例 3:

输入：`digits = ["7"]`, `n = 8`
输出：`1`

提示：

`1 <= digits.length <= 9`
`digits[i].length == 1`
`digits[i] 是从 '1' 到 '9' 的数`
`digits 中的所有值都 不同 `
`digits 按 非递减顺序 排列`
`1 <= n <= 10e9`

### 通用思路
本题几乎可以说是最简单的情况，只需要考虑前导0和是否于n完全一致，核心思路为：
- 维护变量same（表示前位是否与n完全一致）
- 维护变量zeros（表示前位是否全0）

接下去详细讲解这两个变量的价值
- 假定当前位置i的前方的数位，与n完全一致，那么当前位的选择就不能高于n的第i位（例如n是12345，当前位的前方是123，选择第四位的时候就要分成两种情况讨论）：
  - 选择0123的时候，就**存在一位与n不同**，same标记置false，进行进一步dfs；
  - 选择4的时候，前位就**与n完全一致**，标记保留true，进行进一步的dfs
- 假定当前位置i的前方的数位，全是0，那么当前的数位仅需考虑（例如已经选择000）
  - 当前位**继续维持0**，继续dfs（即选择考虑0000x，<10的情况）
  - 亦或者**选择一个非0的数**，不再维持前位全0（ 即选择考虑000xx，<100的情况）


如果两个变量全false，那遍历数当前位置可以填入的值即可（0-9）

上面是解决数位dp的一般思路，绝大多数数位dp可以依照维护这两个变量进行思考

大多数题目还需要维护一个“考虑当前这个数是否满足xxx条件用的变量” curr


### 本题代码
```c++
class Solution {
public:
    int atMostNGivenDigitSet(vector<string>& digits, int n) {
        int m = digits.size();
        vector<int> nums;
        for (auto& s : digits) {
            nums.push_back(s[0] - '0');
        }
        std::ranges::sort(nums);
        string s = std::to_string(n);
        int t = s.size();
        vector<int> memo(t, -1);
        function<int(int, bool, bool)> dfs = [&](int i, bool same,
                                                 bool zeros) -> int {
            // i表示当前位置
            // same表示前面是否完全与n相同
            // zeros表示当前正在考虑前缀0

            if (i >= t) {
                // 由于考虑的是从1到n，因此全0是不可行的
                return int(!zeros);
            }
            if (!same && !zeros && memo[i] != -1) {
                return memo[i];
            }
            int res = 0;
            if (zeros) {
                // 考虑前缀是否全0
                res += dfs(i + 1,false, true);
                // 当前位保留0（例如当前考虑千位，传递到考虑1-999的情况）
                if (same) {
                    // 高位与n完全一致
                    for (int k : nums) {
                        if (k <= s[i] - '0' && k > 0) {
                            // 如果当前位比n大，那就寄了，因此需要判断
                            // 注意，此处不能选择0，否则和上面的保留0矛盾
                            res += dfs(i + 1, k == s[i] - '0', false);
                            // 根据当前位的选择考虑递归传入的same标识
                        }
                    }
                } else {
                    // 高位存在比n小的位，当前位随便怎么选都不会高于n，遍历可选量
                    for (int k : nums) {
                        if (k > 0) {
                            // 注意，此处不能选择0，否则和上面的保留0矛盾
                            res += dfs(i + 1, false, false);
                        }
                    }
                }
            } else {
                // 前方存在非0的元素，因此即使是0也可以安全的选择
                if (same) {
                    // 高位与n完全一致
                    for (int k : nums) {
                        if (k <= s[i] - '0') {
                            // 如果当前位比n大，那就寄了，因此需要判断
                            res += dfs(i + 1, k == s[i] - '0', false);
                            // 根据当前位的选择考虑递归传入的same标识
                        }
                    }
                } else {
                    // 高位存在比n小的位，当前位随便怎么选都不会高于n，遍历
                    for (int k : nums) {
                        res += dfs(i + 1, false, false);
                    }
                    memo[i] = res;
                }
            }
            return res;
        };
        return dfs(0, true, true);
    }
};
```

再给出一道例题，用相似的风格多写几题就能很快上手
## 3. 例题 2：LeetCode 2376 统计特殊整数（数位互不相同）
如果一个正整数每一个数位都是 **互不相同** 的，我们称它是 特殊整数 。

给你一个 **正 整数** `n` ，请你返回区间` [1, n]` 之间特殊整数的数目。

 

示例 1：

输入：`n = 20`
输出：`19`
解释：1 到 20 之间所有整数除了 11 以外都是特殊整数。所以总共有 19 个特殊整数。
示例 2：

输入：`n = 5`
输出：`5`
解释：1 到 5 所有整数都是特殊整数。
示例 3：

输入：`n = 135`
输出：`110`
解释：从 1 到 135 总共有 110 个整数是特殊整数。
不特殊的部分数字为：22 ，114 和 131 。
 

提示：

`1 <= n <= 2 * 10e9`
### 思路讲解
本题的思路很简单，在上一题的基础上增加一个变量curr，
使用二进制掩码来记录在每个数是否存在（例如12就记为0011）

### 本题代码
```c++
class Solution {
public:
    int countSpecialNumbers(int n) {
        string s = std::to_string(n);
        int m = s.size();
        vector<unordered_map<int,int>> memo(m);
        function<int(int, int, bool, bool)> dfs = [&](int i, int curr,
                    bool same, bool zeros) {
            // i表示数位
            // curr用二进制掩码记录被用过的数
            // same表示当前数是否与n一模一样
            // zeros表示当前数的前面是不是全是前导0
            if (i >= m) {
                return int(!zeros);
            }
            if(!same && !zeros && memo[i].count(curr)){
                return memo[i][curr];
            }
            int res = 0;
            if (zeros) {
                //前导0不参与是否被用过的记录
                res += dfs(i + 1, curr, false, true);
                if (!same) {
                    for (int j = 1; j < 10; j++) {
                        if (!((curr >> j) & 1)) {
                            // 这个位没有被用过
                            res += dfs(i + 1, curr + (1 << j), false, false);
                        }
                    }
                } else {
                    int up = s[i] - '0';
                    for (int j = 1; j <= up; j++) {
                        if (!((curr >> j) & 1)) {
                            // 这个位没有被用过
                            res += dfs(i + 1, curr + (1 << j), j == up, false);
                        }
                    }
                }
            } else {
                if (!same) {
                    for (int j = 0; j < 10; j++) {
                        if (!((curr >> j) & 1)) {
                            // 这个位没有被用过
                            res += dfs(i + 1, curr + (1 << j), false, false);
                        }
                    }
                    memo[i][curr] = res;
                } else {
                    int up = s[i] - '0';
                    for (int j = 0; j <= up; j++) {
                        if (!((curr >> j) & 1)) {
                            // 这个位没有被用过
                            res += dfs(i + 1, curr + (1 << j), j == up, false);
                        }
                    }
                }
            }
            return res;
        };
        return dfs(0, 0, true, true);
    }
};
```

## 4. 小结模板

| 需要维护的状态 | 含义 |
|---|---|
| `pos` | 当前处理到第几位（从高到低） |
| `same` | 前面是否紧贴 n 的对应前缀 |
| `zeros` | 前面是否全是前导 0 |
| `curr` | 题目所需“数位性质”压缩（可能很多，静下心来慢慢写） |

把上述四个变量扔进 DFS，**只在 `same==false && zeros==false` 时记忆化**，就能解决 70 % 的数位 DP 计数题。

多写几道，手感自然就来。祝刷题愉快！