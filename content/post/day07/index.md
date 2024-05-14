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

### 官方题解

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
