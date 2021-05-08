## 快速选择

### [数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

> 快速选择，最好的情况下O(n)

```java
    //第K大
    public  int findKthLargest(int[] nums, int k) {
        //找第K大，就是找 len - k 小
        k = nums.length - k;
        int l  = 0;
        int r = nums.length - 1;
        while (l < r){
            //每次返回排好序的位置
            int j = partition(nums,l,r);
            if (j == k){
                break;
            }
            if (j < k){
                l = j + 1;
            }
            if (j > k){
                r = j - 1;
            }
        }
        return nums[k];
    }

    private  int partition(int[] nums, int l, int r) {
        int x = nums[l];
        while (l < r){
            while (l < r && nums[r] >= x){
                r --;
            }
            if (l < r){
                nums[l++] = nums[r];
            }
            while (l < r && nums[l] < x){
                l ++;
            }
            if (l < r){
                nums[r--] = nums[l];
            }
        }
        nums[l] = x;
        return l;
    }
```

## 桶排序

## [前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

>设置若干个桶，每个桶存储出现频率相同的数。桶的下标表示数出现的频率，即第 i 个桶中存储的数出现的频率为 i。
>
>把数都放到桶之后，从后向前遍历桶，最先得到的 k 个数就是出现频率最多的的 k 个数。

```java
    //前K高频元素
    public static int[] topKFrequent(int[] nums, int k) {
        //设置若干个桶，每个桶存储出现频率相同的数。桶的下标表示数出现的频率，即第 i 个桶中存储的数出现的频率为 i。
        Map<Integer,Integer> frequencyForNum = new HashMap<>();
        for (int num : nums) {
            frequencyForNum.put(num,frequencyForNum.getOrDefault(num,0) + 1);
        }
//        以频率为下标来存储频率相同的元素元素
        List<List<Integer>> buckets = new ArrayList<>(nums.length + 1);
        for (int i = 0; i < nums.length + 1 ; i++) {
            buckets.add(new ArrayList<>());
        }
        for (Map.Entry<Integer, Integer> entry : frequencyForNum.entrySet()) {
            int cnt = entry.getValue();
            int num = entry.getKey();
            buckets.get(cnt).add(num);
        }
        int[] res = new int[k];
        int start = 0;
        for (int i = buckets.size() - 1; i >= 0; i--) {
            if (buckets.get(i).isEmpty()) {
                continue;
            }
            //将高频元素添加进来
            for (Integer integer : buckets.get(i)) {
                if (start < k){
                    res[start++] = integer;
                }else {
                   return res;
                }
            }
        }
        return res;
    }
```

### [根据字符出现频率排序](https://leetcode-cn.com/problems/sort-characters-by-frequency/)

```java
//字符串频率降序
    public static String frequencySort(String s) {
        char[] chars = s.toCharArray();
        Map<Character,Integer> charCnt = new HashMap<>();
        for (char c : chars) {
            charCnt.put(c,charCnt.getOrDefault(c,0) + 1);
        }
        //统计频率字母
        List<List<Character>> buckets = new ArrayList<>(chars.length + 1);
        for (int i = 0; i < chars.length + 1 ; i++) {
            buckets.add(new ArrayList<>());
        }
        for (Map.Entry<Character, Integer> entry : charCnt.entrySet()) {
            int cnt = entry.getValue();
            char c = entry.getKey();
            buckets.get(cnt).add(c);
        }
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = buckets.size() - 1; i >= 0 ; i--) {
            if (buckets.get(i).isEmpty()){
                continue;
            }
            for (Character character : buckets.get(i)) {
                for (int times = i; times >0 ; times--) {
                    stringBuilder.append(character);
                }
            }
        }
        return stringBuilder.toString();
    }
```

## 荷兰国旗问题

> 给定一个包含红色、白色和蓝色，一共 `n` 个元素的数组，**[原地](https://baike.baidu.com/item/原地算法)**对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

```java
 //三色排序
    public static void sortColors(int[] nums) {
        /**
        * 遍历的时候，
         * 假定[0,p0) 全部是数字0
         *那么可以设定[p0,i)全部是 数字 1
         *那么可以设定[i,p2]全部是 数字 2
         * 初始化：
         * 为了保证初始化的时候 [0, p0) 为空，设置 zero = 0，
         * 所以[i,p2]也是空集，p2初始化为 len 因为 len不可访问 所以每次操作之前p2--
        */
        //或者
        /**
        *  [0,p0]全部是数字 0
         * (p0,i)全部是数字 1
         * [i,p2]全部是数字2
         * 初始化
         * p0 = -1;  那么每次操作之前 p0 ++
         * p2 = len
        */
        int p0 = -1;
        int p2 = nums.length;
        int i = 0;
        while (i < p2){
            if (nums[i] == 0){
                p0++;
                swap(nums,p0,i);
                i++;
            }else if (nums[i] == 1){
                i++;
            }else if (nums[i] == 2){
                p2--;
                swap(nums,p2,i);
            }
        }
    }

    private static void swap(int[] nums, int i, int j) {
        int t = nums[i];
        nums[i] = nums[j];
        nums[j] = t;
    }
```

