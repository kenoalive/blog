---
title: LeetCode刷题笔记
date: 2024-05-14 00:00:00
image: cover.png
categories: ["前端"]
weight: 1 
---

### 两数之和
给定一个整数数组`nums`和一个整数目标值`target`，请你在该数组中找出 和为目标值`target`的那两个整数，并返回它们的数组下标。

#### 我的题解
暴力枚举，用嵌套循环就是干
```js
var twoSum = function(nums, target) {
    let result = []
    for(let i = 0;i<nums.length;i++){
        for(let j = i + 1;j<nums.length;j++) {
            if(nums[i] + nums[j] === target) {
                result = [i,j]
            }
        }
    }   

    return result
};
```
#### 更优雅的题解
评论里看到的题解，用`targetNum = target - curNum`直接拿到循环要获取的目标数据，然后用对象存储对应的索引和数据
```js
const twoSum = (nums, target) => {
  const prevNums = {};                    // 存储出现过的数字，和对应的索引               

  for (let i = 0; i < nums.length; i++) {       // 遍历元素   
    const curNum = nums[i];                     // 当前元素   
    const targetNum = target - curNum;          // 满足要求的目标元素   
    const targetNumIndex = prevNums[targetNum]; // 在prevNums中获取目标元素的索引
    if (targetNumIndex !== undefined) {         // 如果存在，直接返回 [目标元素的索引,当前索引]
      return [targetNumIndex, i];
    } else {                                    // 如果不存在，说明之前没出现过目标元素
      prevNums[curNum] = i;                     // 存入当前的元素和对应的索引
    }
  }
}
```

### 字母异位词分组
给你一个字符串数组，请你将字母异位词组合在一起
```
输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出: [["bat"],["nat","tan"],["ate","eat","tea"]]
```

#### 我的题解
思路是将所有单词的字符和频率统一转换成一个对象，用来判断是否为字母异位词

```js
const groupAnagrams = (strs) => {
    let newWords = []; // 输出的新单词
    let wordTypes = []; // 原单词的字符及其频率字符串数组，用来判断是否为字母异位词

    for (let i in strs) {
        const item = strs[i]
        const wordstr = getWordMaps(item);
        const index = wordTypes.indexOf(wordstr)
        if (index == -1) {
            wordTypes.push(wordstr);
            newWords.push([strs[i]])
        } else {
            newWords[index].push(strs[i])
        }
    }

    return newWords
};

const getWordMaps = (word) => {
    const map = {};
    for (item of word) {
        map[item] = (map[item] || 0) + 1;
    }

    const str = Object.entries(map)
        .sort((a, b) => a[0].localeCompare(b[0]))
        .toString();
    return str;
};
```

#### 官方题解 
> 由于互为字母异位词的两个字符串包含的字母相同，因此对两个字符串分别进行排序之后得到的字符串一定是相同的，故可以将排序之后的字符串作为哈希表的键。

直接使用Array.from() 和 sort() 来判断是否为字母异位词（看完觉得我太蠢了！😭

```js
var groupAnagrams = function(strs) {
    const map = new Map();
    for (let str of strs) {
        let array = Array.from(str);
        array.sort();
        let key = array.toString();
        let list = map.get(key) ? map.get(key) : new Array();
        list.push(str);
        map.set(key, list);
    }
    return Array.from(map.values());
};
```


### 移除元素
给你一个数组`nums`和一个值`val`，你需要原地移除所有数值等于`val`的元素，并返回移除后数组的新长度。
不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。

题解：由于不允许使用另外的数组空间，所以考虑使用双指针直接在原数组上修改，left、right为两个指针，right先出发，遇到符合条件的value，则复制给left，然后left++，循环完毕时，数组前left项就是已经移除后的新数组
```js
var removeElement = function(nums, val) {
    const n = nums.length;
    let left = 0;
    for (let right = 0; right < n; right++) {
        if (nums[right] !== val) {
            nums[left] = nums[right];
            left++;
        }
    }
    return left;
};
```

优化：
```js
var removeElement = function(nums, val) {
    let left = 0, right = nums.length;
    while (left < right) {
        if (nums[left] === val) {
            nums[left] = nums[right - 1];
            right--;
        } else {
            left++;
        }
    }
    return left;
};
```


### 删除有序数组中的重复项
#### 我的题解
直接用set去重，然后循环原数组，修改数组项
```js
var removeDuplicates = function (nums) {
    const newArr = Array.from(new Set(nums))

    for (let i = 0; i < newArr.length; i++) {
        nums[i] = newArr[i]
    }

    return newArr.length
};
```

#### 官方题解

还是用双指针：
如果数组 `nums` 的长度为0，则数组不包含任何元素，因此返回0。
当数组`nums`的长度大于0时，数组中至少包含一个元素，在删除重复元素之后也至少剩下一个元素，因此nums[0]保持原状即可，从下标1开始删除重复元素。

```js
var removeDuplicates = function(nums) {
    const n = nums.length;
    if (n === 0) {
        return 0;
    }
    let fast = 1, slow = 1;
    while (fast < n) {
        if (nums[fast] !== nums[fast - 1]) {
            nums[slow] = nums[fast];
            ++slow;
        }
        ++fast;
    }
    return slow;
};
```


### 多数元素

给定一个大小为 n 的数组 nums ，返回其中的多数元素。多数元素是指在数组中出现次数 大于 ⌊ n/2 ⌋ 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

#### 我的题解
使用Map统计每个元素出现的次数，如果times 大于 n/2，则输出

```js
var majorityElement = function (nums) {
    const halfLen = nums.length / 2;
    const numMap = new Map()
    let target = 0;

    for (item of nums) {
        let times = numMap.has(item) ? numMap.get(item) + 1 : 1
        numMap.set(item, times)
        if (times > halfLen) {
            return item
        }
    }
};
```

#### 优化题解
因为大于一半, 所以排序后的 中间那个数必是
```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var majorityElement = function(nums) {
  nums.sort((a,b) => a - b)
  return nums[Math.floor(nums.length / 2)]
};
```
用对象记录：
```js

var majorityElement = function(nums) {
   let half = nums.length / 2
   let obj = {}
   for(let num of nums){
      obj[num] = (obj[num] || 0) + 1
      if(obj[num] > half) return num
   }
};
```

### 最后一个单词的长度
给你一个字符串`s`，由若干单词组成，单词前后用一些空格字符隔开。返回字符串中最后一个单词的长度。
单词是指仅由字母组成、不包含任何空格字符的最大子字符串

#### 我的题解
现成的js api，调就完事
```js
var lengthOfLastWord = function(s) {
    return s.trim().split(' ').pop().length;
};
```

#### 官方题解

```js
var lengthOfLastWord = function(s) {
    let end = s.length - 1;
    while(end >= 0 && s[end] == ' ') end--;
    if(end < 0) return 0;
    let start = end;
    while(start >= 0 && s[start] != ' ') start--;
    return end - start;
};
```



### 最长公共前缀
编写一个函数来查找字符串数组中的最长公共前缀。如果不存在公共前缀，返回空字符串 ""。

#### 我的题解
初始化prefix为数组第一项`strs[0]`，从`strs[1]`开始循环对比 `prefix` 和字符串

```js
var longestCommonPrefix = function (strs) {
    let prefix = ""
    if (!strs.length) return ""
    prefix = strs[0]

    for (let i = 1; i < strs.length; i++) {
        if(strs[i] === "") return ""

        for (let j = 0; j < prefix.length; j++) {
            if (strs[i][j] !== prefix[j]) {
                prefix = prefix.substring(0, j)
            }
        }
    }
    return prefix;
};
```

### 有时间限制的缓存
编写一个类，它允许获取和设置键-值对，并且每个键都有一个过期时间 。
该类有三个公共方法：
`set(key, value, duration)` ：接收参数为整型键 key 、整型值 value 和以毫秒为单位的持续时间 duration 。一旦 duration 到期后，这个键就无法访问。如果相同的未过期键已经存在，该方法将返回 true ，否则返回 false 。如果该键已经存在，则它的值和持续时间都应该被覆盖。

`get(key)`：如果存在一个未过期的键，它应该返回这个键相关的值。否则返回 -1 。

`count()`：返回未过期键的总数。

#### 我的题解
使用Map和setTimeout，每次set的时候，先判断key是否存在，如果存在则清除原来的timeout，然后重新设置新的timeout

```js
var TimeLimitedCache = function () {
    this.map = new Map();
    this.times = {}
};

/** 
 * @param {number} key
 * @param {number} value
 * @param {number} duration time until expiration in ms
 * @return {boolean} if un-expired key already existed
 */
TimeLimitedCache.prototype.set = function (key, value, duration) {
    const hasCache = this.map.has(key)
    if (hasCache) clearTimeout(this.times[key]);

    this.map.set(key, value) // 设置键和值
    this.times[key] = setTimeout(() => { // 设置time
        this.map.delete(key)
    }, duration)

    return hasCache;
};

/** 
 * @param {number} key
 * @return {number} value associated with key
 */
TimeLimitedCache.prototype.get = function (key) {
    return this.map.has(key) ? this.map.get(key) : -1
};

/** 
 * @return {number} count of non-expired keys
 */
TimeLimitedCache.prototype.count = function () {
    return this.map.size
};
```

### 扁平化嵌套数组

#### 我的题解
使用递归
```js
var flat = function (arr, n) {
    if (!n) return arr;
    let res = arr

    for (let i = 0; i < n; i++) {
        res = flat(res)
    }

    function flat(arrs) {
        return arrs.reduce((pre, cur) => {
            if (Array.isArray(cur)) {
                return pre.concat(cur)
            } else {
                pre.push(cur)
                return pre;
            }
        }, [])
    }

    return res
};
```
