# 字符串 && 数组

[TOC]

## 字符串

### [415. 字符串相加](https://leetcode-cn.com/problems/add-strings/)

> 给定两个字符串形式的非负整数 num1 和num2 ，计算它们的和。
>
> 提示：
>
> num1 和num2 的长度都小于 5100
> num1 和num2 都只包含数字 0-9
> num1 和num2 都不包含任何前导零
> 你不能使用任何內建 BigInteger 库， 也不能直接将输入的字符串转换为整数形式

* 模拟

  * 思路

    * 本题只要对两个大整数模拟 **竖式加法**的过程即可，竖式加法就是纸算两个整数相加的方法，即相同数位对齐，从低到高位逐位相加，若当前位和超过10，则进位

      <img src="https://assets.leetcode-cn.com/solution-static/415/1.png" alt="fig1" style="zoom: 25%;" />

    * 具体实现也不复杂，定义两个指针`i, j`分别指向`num1, num2`的末尾，即最低位，同时定义一个变量`add`表示当前位是否有进位，然后从末尾到开头逐位相加即可。若两个数字位数不同，则对 **位数较短的数字进行补零操作**

  * 代码

    ```c++
    // https://leetcode-cn.com/problems/add-strings/solution/zi-fu-chuan-xiang-jia-by-leetcode-solution/
    // 模拟
    class Solution {
    public:
        string addStrings(string num1, string num2) {
            int i = num1.size() - 1, j = num2.size() - 1, add = 0;
            string res = "";
            while (i >= 0 || j >= 0 || add != 0) {
                int x = i >= 0 ? num1[i] - '0' : 0;
                int y = j >= 0 ? num2[j] - '0' : 0;
                int s = x + y + add;
                res.push_back('0' + s % 10);
                add = s / 10;
                i -= 1;
                j -= 1;
            }
            reverse(res.begin(), res.end());
            return res;
        }
    };
    ```

  * 复杂度

    * 时间复杂度：$O(max(len1, len2))$
    * 空间复杂度：$O(1)$



### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

> 给你一个字符串 `s`，找到 `s` 中最长的回文子串。
>
> **示例 1：**
>
> ```
> 输入：s = "babad"
> 输出："bab"
> 解释："aba" 同样是符合题意的答案。
> ```
>
> **示例 2：**
>
> ```
> 输入：s = "cbbd"
> 输出："bb"
> ```

* 动态规划：

* 中心扩散法

  * 思路:回文串一定是对称的，所以我们可以每次循环选择一个中心，进行左右扩展，判断左右字符是否相等即可。

  ![image.png](https://pic.leetcode-cn.com/1b9bfe346a4a9a5718b08149be11236a6db61b3922265d34f22632d4687aa0a8-image.png)

  由于存在奇数的字符串和偶数的字符串，所以我们需要从一个字符开始扩展，或者从两个字符之间开始扩展，所以总共有 n+n-1 个中心。

  * 代码

  ```c++
  class Solution {
  public:
      string longestPalindrome(string s) {
          int max_left = 0, max_right = 0;
          for (int i = 0; i < s.size(); ++i) {
              std::pair<int, int> odd = getPalindromeRange(s, i, i);
              std::pair<int, int> even = getPalindromeRange(s, i, i + 1);
              if (odd.second - odd.first > max_right - max_left) {
                  max_left = odd.first;
                  max_right = odd.second;
              }
              if (even.second - even.first > max_right - max_left) {
                  max_left = even.first;
                  max_right = even.second;
              }
          }
          return s.substr(max_left, max_right - max_left + 1);
      }
  
      pair<int, int> getPalindromeRange(string &s, int left, int right) {
          while (left >= 0 && right < s.size() && s[left] == s[right]) {
              --left;
              ++right;
          }
          return {left + 1, right - 1};
      }
  };
  ```

  



### [3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

> 给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。
>
> 示例 1:
>
> 输入: s = "abcabcbb"
> 输出: 3 
> 解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
> 示例 2:
>
> 输入: s = "bbbbb"
> 输出: 1
> 解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
> 示例 3:
>
> 输入: s = "pwwkew"
> 输出: 3
> 解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
>      请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
> 示例 4:
>
> 输入: s = ""
> 输出: 0

* 滑动窗口

  * 思路

    * 这道题主要用到思路是：滑动窗口
    * 什么是滑动窗口？其实就是一个队列,比如例题中的 abcabcbb，进入这个队列（窗口）为 abc 满足题目要求，当再进入 a，队列变成了 abca，这时候不满足要求。所以，我们要移动这个队列！
    * 如何移动？我们只要把队列的左边的元素移出就行了，直到满足题目要求！一直维持这样的队列，找出队列出现最长的长度时候，求出解！

  * 代码

    ```c++
    // https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/solution/hua-dong-chuang-kou-by-powcai/
    // 滑动窗口法
    class Solution {
    public:
        int lengthOfLongestSubstring(string s) {
            int left = 0, right = 0;
            int res = 0;
            unordered_set<char> lookup;
            for (right = 0; right < s.size(); ++right) {
                while (lookup.find(s[right]) != lookup.end()) {
                    lookup.erase(s[left]);
                    ++left;
                }
                res = max(res, right - left + 1);
                lookup.insert(s[right]);
            }
            return res;
        }
    };
    ```

  * 复杂度

    * 时间复杂度：O(n)
    * 空间复杂度：O(n)

### [8. 字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi/)

> 请你来实现一个 myAtoi(string s) 函数，使其能将字符串转换成一个 32 位有符号整数（类似 C/C++ 中的 atoi 函数）。
>
> 函数 myAtoi(string s) 的算法如下：
>
> 读入字符串并丢弃无用的前导空格
> 检查下一个字符（假设还未到字符末尾）为正还是负号，读取该字符（如果有）。 确定最终结果是负数还是正数。 如果两者都不存在，则假定结果为正。
> 读入下一个字符，直到到达下一个非数字字符或到达输入的结尾。字符串的其余部分将被忽略。
> 将前面步骤读入的这些数字转换为整数（即，"123" -> 123， "0032" -> 32）。如果没有读入数字，则整数为 0 。必要时更改符号（从步骤 2 开始）。
> 如果整数数超过 32 位有符号整数范围 [−231,  231 − 1] ，需要截断这个整数，使其保持在这个范围内。具体来说，小于 −231 的整数应该被固定为 −231 ，大于 231 − 1 的整数应该被固定为 231 − 1 。
> 返回整数作为最终结果。
> 注意：
>
> 本题中的空白字符只包括空格字符 ' ' 。
> 除前导空格或数字后的其余字符串外，请勿忽略 任何其他字符。
>
>
> 示例 1：
>
> 输入：s = "42"
> 输出：42
> 解释：加粗的字符串为已经读入的字符，插入符号是当前读取的字符。
> 第 1 步："42"（当前没有读入字符，因为没有前导空格）
>          ^
> 第 2 步："42"（当前没有读入字符，因为这里不存在 '-' 或者 '+'）
>          ^
> 第 3 步："42"（读入 "42"）
>            ^
> 解析得到整数 42 。
> 由于 "42" 在范围 [-231, 231 - 1] 内，最终结果为 42 。
> 示例 2：
>
> 输入：s = "   -42"
> 输出：-42
> 解释：
> 第 1 步："   -42"（读入前导空格，但忽视掉）
>             ^
> 第 2 步："   -42"（读入 '-' 字符，所以结果应该是负数）
>              ^
> 第 3 步："   -42"（读入 "42"）
>                ^
> 解析得到整数 -42 。
> 由于 "-42" 在范围 [-231, 231 - 1] 内，最终结果为 -42 。
> 示例 3：
>
> 输入：s = "4193 with words"
> 输出：4193
> 解释：
> 第 1 步："4193 with words"（当前没有读入字符，因为没有前导空格）
>          ^
> 第 2 步："4193 with words"（当前没有读入字符，因为这里不存在 '-' 或者 '+'）
>          ^
> 第 3 步："4193 with words"（读入 "4193"；由于下一个字符不是一个数字，所以读入停止）
>              ^
> 解析得到整数 4193 。
> 由于 "4193" 在范围 [-231, 231 - 1] 内，最终结果为 4193 。
> 示例 4：
>
> 输入：s = "words and 987"
> 输出：0
> 解释：
> 第 1 步："words and 987"（当前没有读入字符，因为没有前导空格）
>          ^
> 第 2 步："words and 987"（当前没有读入字符，因为这里不存在 '-' 或者 '+'）
>          ^
> 第 3 步："words and 987"（由于当前字符 'w' 不是一个数字，所以读入停止）
>          ^
> 解析得到整数 0 ，因为没有读入任何数字。
> 由于 0 在范围 [-231, 231 - 1] 内，最终结果为 0 。
> 示例 5：
>
> 输入：s = "-91283472332"
> 输出：-2147483648
> 解释：
> 第 1 步："-91283472332"（当前没有读入字符，因为没有前导空格）
>          ^
> 第 2 步："-91283472332"（读入 '-' 字符，所以结果应该是负数）
>           ^
> 第 3 步："-91283472332"（读入 "91283472332"）
>                      ^
> 解析得到整数 -91283472332 。
> 由于 -91283472332 小于范围 [-231, 231 - 1] 的下界，最终结果被截断为 -231 = -2147483648 。



### [151. 翻转字符串里的单词](https://leetcode-cn.com/problems/reverse-words-in-a-string/)

> 给你一个字符串 s ，逐个翻转字符串中的所有 单词 。
>
> 单词 是由非空格字符组成的字符串。s 中使用至少一个空格将字符串中的 单词 分隔开。
>
> 请你返回一个翻转 s 中单词顺序并用单个空格相连的字符串。
>
> 说明：
>
> 输入字符串 s 可以在前面、后面或者单词间包含多余的空格。
> 翻转后单词间应当仅用一个空格分隔。
> 翻转后的字符串中不应包含额外的空格。
>
>
> 示例 1：
>
> 输入：s = "the sky is blue"
> 输出："blue is sky the"
> 示例 2：
>
> 输入：s = "  hello world  "
> 输出："world hello"
> 解释：输入字符串可以在前面或者后面包含多余的空格，但是翻转后的字符不能包括。
> 示例 3：
>
> 输入：s = "a good   example"
> 输出："example good a"
> 解释：如果两个单词间有多余的空格，将翻转后单词间的空格减少到只含一个。
> 示例 4：
>
> 输入：s = "  Bob    Loves  Alice   "
> 输出："Alice Loves Bob"
> 示例 5：
>
> 输入：s = "Alice does not even like bob"
> 输出："bob like even not does Alice"

* 思路：模拟遍历

* 代码

  ```c++
  // https://leetcode-cn.com/problems/reverse-words-in-a-string/solution/cyou-mei-jian-dan-dai-ma-by-xiaohu9527-imlc/908779
  // 模拟
  class Solution {
  public:
      string reverseWords(string s) {
          int i = 0, len = s.size();
          string res = "";
          while (i < len) {
              int c = 0;
              // i 找到单词起点
              while (i < len && s[i] == ' ') {
                  ++i;
              }
              // i,c同时走，i走到单词末尾，c表示单词长度,i - c表示单词起点左边
              while (i < len && s[i] != ' ') {
                  ++i;
                  ++c;
              }
              if (c) {
                  // i - c 表示单词起点，c表示单词长度
                  res = s.substr(i - c, c) + " " + res;
              }
          }
          return res.substr(0, res.size() - 1);
      }
  };
  ```

* 复杂度
