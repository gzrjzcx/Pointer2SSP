![img](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LrrEWl8BTXDnZp0laOP%2F-LrrGj7zqA-hkThkk982%2F-LrrJeuGkf3NbvBjC7u_%2F%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-10-23%20%E4%B8%8B%E5%8D%882.33.34.png?alt=media&token=b85edd09-ea71-431d-9d59-0fbc259b67a2)

Enter a caption for this image (optional)









‌

首先，回文指的是正序倒序都相同的字符序列。在此我们尝试用三种方法解决此题：DP，ExpandAroundCenter和Manacher。







‌

# DP







‌

DP是解决此题的一种常规思路。根据回文的定义可知，一个字符串 SijSij      S_{ij}<math><semantics><mrow><msub><mi>S</mi><mrow><mi>i</mi><mi>j</mi></mrow></msub></mrow><annotation encoding="application/x-tex">S_{ij}</annotation></semantics></math>Sij​ 是回文的条件为：







Sij={true=substring Si+1,j−1 is palindrome && Si==Sjfalse=otherwiseSij={true=substring Si+1,j−1 is palindrome && Si==Sjfalse=otherwise







‌

可以看出这是一个bool类型的状态转移矩阵。例如，如果bab 是回文子串，那么ababa 肯定也是回文子串，因为 S1==a==S5S1==a==S5      S_1 = =a == S_5<math><semantics><mrow><msub><mi>S</mi><mn>1</mn></msub><mo>=</mo><mo>=</mo><mi>a</mi><mo>=</mo><mo>=</mo><msub><mi>S</mi><mn>5</mn></msub></mrow><annotation encoding="application/x-tex">S_1 = =a == S_5</annotation></semantics></math>S1​==a==S5​ 且bab 是回文子串。为了记录子串是否为回文，我们利用一个数组dp[n][n] 来记录子串是否会回文。整个DP寻找思路可以看成先把子串长度为1或2的初始回文子串找到，然后从子串长度为3开始利用状态转移方程寻找。时间复杂度和空间复杂度均为 O(n2)O(n2)      O(n^2)<math><semantics><mrow><mi>O</mi><mo>(</mo><msup><mi>n</mi><mn>2</mn></msup><mo>)</mo></mrow><annotation encoding="application/x-tex">O(n^2)</annotation></semantics></math>O(n2) ，代码如下：









DP

C

Add file

C 

exit: ⌘↩

```
char* longestPalindrome(char* s) {
    int max_count = 1, len = strlen(s), start = 0;
    if(len < 2)
        return s;
    int dp[len][len]; //记录子串是否为回文
    memset(dp, 0, sizeof(dp));
    
    //先初始化dp,单个字符和一对相等的字符都是回文。相当于先把子串长度为1和2的回文找到。
    for(int i=0; i<len; i++)
    {
        dp[i][i] = 1;
        if(s[i+1] && s[i] == s[i+1])
        {
            dp[i][i+1] = 1;
            max_count = 2;
            start = i;
        }
    }
    
    //从长度为3的子串开始寻找
    for(int sublen=3; sublen<=len; sublen++)
        for(int i=0; i<=len-sublen; i++)
        {
            int j = i+sublen-1;
            if(dp[i+1][j-1] && s[i] == s[j])
            {
                dp[i][j] = 1;
                if(max_count < sublen)
                {
                    max_count = sublen;
                    start = i;
                }
            }
        }
    char *res = (char *)malloc(max_count+1);
    memcpy(res, s+start, max_count);
    res[max_count] = 0;
    return res;
}
               
```







‌

# ExpandAroundCenter







‌

实际上，我们并不需要用一个辅助数组dp来记录子串是否为回文，而是可以从最短的子串开始寻找。其核心思想是找到一个回文子串的中心点，然后不断向两边扩展直到两边字符不相同。需要注意的是该中心点可能是一个字符，也可能是两个字符，例如aba 的中心点是a, abba 的中心点则是bb。如此则把空间复杂度降为 O(n)O(n)      O(n)<math><semantics><mrow><mi>O</mi><mo>(</mo><mi>n</mi><mo>)</mo></mrow><annotation encoding="application/x-tex">O(n)</annotation></semantics></math>O(n) ，代码如下：









ExpandAroundCenter

C

Add file

C 

exit: ⌘↩

```
void expandAroundCenter(char *s, int left, int right, int len, int *start, int *max_count)
{
    while(left>=0 && right<=len && s[left] == s[right])
    {
        left--;
        right++;
    }
    if(*max_count < right-left-1)
    {
        *max_count = right-left-1;
        *start = left+1;
    }
}


char* longestPalindrome(char* s) {
    int len = strlen(s), start = 0, max_count =0;
    if(len < 2)
        return s;


    for(int i=0; i<len; i++)
    {
        expandAroundCenter(s, i, i, len, &start, &max_count);
        expandAroundCenter(s, i, i+1, len, &start, &max_count);
    }
    
    char *res = (char *)malloc(max_count+1);
    memcpy(res, s+start, max_count);
    res[max_count] = 0;
    return res;
}
               
```







‌

# Manacher's Algorithm







‌

Manacher算法实际上可以看做是ExpandAroundCenter和DP方法的结合与优化：它基于ExpandAroundCenter的思想，先找到每个回文子串的中心点；同时利用DP的思想，**借助当前判断的子串的对称串（肯定被搜索过）来判断当前子串是否为回文子串**。利用对称子串的信息是Manacher算法的关键，其中可以分为如下两种情况：







![img](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LrrEWl8BTXDnZp0laOP%2F-LrxqLh8b5cZoh1OSYLX%2F-Lry41w5KuMb1sN1hokM%2Fmanacher_algorithm.png?alt=media&token=571c9550-735d-4d2a-9a5c-d6ec340e60b0)

Enter a caption for this image (optional)







![img](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LrrEWl8BTXDnZp0laOP%2F-LrxqLh8b5cZoh1OSYLX%2F-Lry41w6z8PCQ0Ex4jYV%2Fmanacher_algorithm2.png?alt=media&token=45899d09-e695-4b42-b1f1-83dd840db1fd)

Enter a caption for this image (optional)







‌

在整个循环中，我们需要不断更新两个变量：







‌

- `pivot` ：当前检测到的最大的回文子串的中心点。
- `max_right` ：当前检测到的最大的回文子串的右边界点。





 





 





 





 