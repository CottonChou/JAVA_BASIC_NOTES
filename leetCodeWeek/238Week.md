#### [ K 进制表示下的各位数字总和](https://leetcode-cn.com/problems/sum-of-digits-in-base-k/)

>给你一个整数 n（10 进制）和一个基数 k ，请你将 n 从 10 进制表示转换为 k 进制表示，计算并返回转换后各位数字的 总和 。
>
>转换后，各位数字应当视作是 10 进制数字，且它们的总和也应当按 10 进制表示返回。

```
输入：n = 34, k = 6
输出：9
解释：34 (10 进制) 在 6 进制下表示为 54 。5 + 4 = 9 。
```

```java
public int sumBase(int n, int k) {
        if(n < k){
            return n;
        }
        int sum = 0;
        while(n >= k){
            sum += n%k;
            n = n/k;
        }
        if(n != 0){
            sum += n;
        }
        return sum;
}
```

#### [最高频元素的频数](https://leetcode-cn.com/problems/frequency-of-the-most-frequent-element/)

>元素的 频数 是该元素在一个数组中出现的次数。
>
>元素的 频数 是该元素在一个数组中出现的次数。
>
>给你一个整数数组 nums 和一个整数 k 。在一步操作中，你可以选择 nums 的一个下标，并将该下标对应元素的值增加 1 。
>
>执行最多 k 次操作后，返回数组中最高频元素的 最大可能频数 。
>

```json
输入：nums = [1,4,8,13], k = 5
输出：2
解释：存在多种最优解决方案：

- 对第一个元素执行 3 次递增操作，此时 nums = [4,4,8,13] 。4 是数组中最高频元素，频数是 2 。
- 对第二个元素执行 4 次递增操作，此时 nums = [1,8,8,13] 。8 是数组中最高频元素，频数是 2 。
- 对第三个元素执行 5 次递增操作，此时 nums = [1,4,13,13] 。13 是数组中最高频元素，频数是 2 。
```

```java
//思路就是滑动窗口枚举每一位，前提是先排序。每次新增一个大数，需要加1的次数等于前面的数字个数*差值。前面的一定是反转好的数字是对齐的。
// 1-> 1
// 1 2 -> 2 2 
// 2 2 3 -> 3 3 3

//因此加1 的次数等于 num[i] - num[i-1] * (i - left)
public int maxFrequency(int[] nums, int k) {
        int cnt = 0;
        int max = 0;
        Arrays.sort(nums);
        int left = 0;
        int right = 0;
        int len = nums.length;
        while(right < len){
            while(right < len && cnt <= k){
                //这一句可以放在循环外，但是放在循环内我觉得看起来更加清晰
                max = Math.max(max,right - left + 1);
                right++;
                if(right == len){
                    break;
                }
                // cnt计算的是 假设每个数都要加1 = 这个数的次数 
                cnt = cnt + (nums[right] - nums[right-1]) * (right - left);
            }
            while(left < right && cnt > k){
                cnt = cnt - (nums[right] - nums[left]);
                left++;
            }
        }
        return max;
    }
```

#### [所有元音按顺序排布的最长子字符串](https://leetcode-cn.com/problems/longest-substring-of-all-vowels-in-order/)

>当一个字符串满足如下条件时，我们称它是 美丽的 ：
>
>所有 5 个英文元音字母（'a' ，'e' ，'i' ，'o' ，'u'）都必须 至少 出现一次。
>这些元音字母的顺序都必须按照 字典序 升序排布（也就是说所有的 'a' 都在 'e' 前面，所有的 'e' 都在 'i' 前面，以此类推）
>比方说，字符串 "aeiou" 和 "aaaaaaeiiiioou" 都是 美丽的 ，但是 "uaeio" ，"aeoiu" 和 "aaaeeeooo" 不是美丽的 。
>
>给你一个只包含英文元音字母的字符串 word ，请你返回 word 中 最长美丽子字符串的长度 。如果不存在这样的子字符串，请返回 0 。
>
>子字符串 是字符串中一个连续的字符序列。

```java
//不常规的解法，题目只有元音字母，元音是升序的
public int longestBeautifulSubstring(String word) {
        int ans = 0, type = 1, len = 1;
        for(int i = 1; i < word.length(); i++){
            if(word.charAt(i) >= word.charAt(i-1)) len++; //更新字符串长度
            if(word.charAt(i) > word.charAt(i-1)) type++; //更新字符种类
            if(word.charAt(i) < word.charAt(i-1)) { type = 1; len = 1;} //字符串不美丽，从当前字符重新开始
            if(type == 5) ans = Math.max(ans, len);  //更新最大字符串
        } 
        return ans;
    }
```

