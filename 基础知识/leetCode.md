# TwoSum

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

```
Example:
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

AC CODE:

```swift
class Solution {
    func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
        var final: [Int] = [0, 0]
        var index: Int = 0
        for _ in nums {
            var index_i:Int = index+1
            for _ in index+1..<nums.count {
                if target == nums[index] + nums[index_i] {
                    final = [index, index_i]
                }
                index_i = index_i + 1
            }
            index = index + 1
        }
        return final
    }
}
```

题意非常简单，在LeetCode上也是Easy，可能是太久没被OJ虐过了，以为它说的Easy和我自己想的Easy是一个层面。🙂。最开始我的想法是用target遍历nums，对每一个item做差值temp，然后再nums中直接contains(temp)，这就是我认为的Easy。😔。

然后事实上我却忘了最简单的重复对象没考虑上，当出现([3, 2, 4], 6)的测试集wa就直打脸，最后加上判重方法后，再submit，居然这回给我wa的是因为含有负数的测试集没过。

这时才想起来，原来Easy的套路这么多。此时，我的做法变为了把nums先abs后为Nums，Nums再重复之前的工作，此时负数的测试集过了，但是尼玛居然还出了比如负数和正数凑一块的测试集([-3, 2, 3], 0)，这回print出来的final就成了(0, 0)，因为abs后导致了做了差值的temp去找了第一次出现的3，而不是第二次出现的3。

最后受不了了，仔细的再梳理了一遍思路，很多时候啊一些被别人标上Easy的事情，没经过自己手也私以为是Easy，殊不知那是人家磕磕碰碰后的“得道”Easy，最后一拍脑瓜子才傻愣愣的发现完全不需要abs，直接用target见遍历后的item拿到temp后，直接contains即可，但是！！！submit居然woc的又time out！！！我只记得当初自己踏坑的时候每遇到time out就GG。

此时隐约的觉得会不会是while的问题导致的time out，但实际上while和for的时间复杂度都是O(n)哇，但是我没查到关于contains函数的复杂度到底是多少，我觉得Swift的Array实际上跟C++中的Vector差不多，虽然肯定是不会直接干find，但至少我认为应该是有参考过find的，而且还是觉得哪里不太对劲，但是又说不出来，我觉得不管是find还是contains会不会就是个hash_map呢？如果就是hash_map，那就游离在O(1)和O(n)之间了。所以那最差也就是O(n^2)，我就没搞懂这是为啥老给我time out。

没办法，死磕也不是个办法，这出门不顺一下子就遇到了坎，看了solution，原来就是把contains换成了两个for，修改成了Swift版本后，这尼玛就AC了。

我是没搞懂为啥，只能日后再探了或者希望有识之士能给我提个醒🙏。

====== 2018-8-13 更新 ======
哈哈，最近又开始做 LeetCode 主要是想用剩下的时间给明年春招的优势更大一些（我是真的不想秋招），所以又开始二刷 2Sum 哈哈。不过第二次再去思考这道题的时候又给了我很多不一样的思考，直接看代码吧。

```Swift
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
        var index = 0
        var final:[Int] = [0 ,0]
        for num in nums {
            let tempNum = target - num
            let tempNums = nums[index+1..<nums.count]
            
            // contains 为一次遍历
            if tempNums.contains(tempNum) {
                // 这里的 index 多了一次遍历
                let tempIndex:Int = tempNums.index(of: tempNum)!
                final = [index, tempIndex]
           }
            index += 1
        }
    return final
}
```

这是第一遍提交的超时代码，给我报超时了以后其实我就知道个差不多了，首先有个 for 已经是 O(n) 了，用到了 `num[index+1..<nums.count]` 这已经是 O(n^2) 了，然后还有个 `contains` 这就 O(n^3) 了，后边还有个 `index` ，emmm，确实该超时了。

接下来又优化了一下，代码如下：

```Swift
var index = 0
var final:[Int] = [0 ,0]
for num in nums {
    var n_index = index + 1
    for n in nums[n_index..<nums.count] {
        if n + num == target {
            final = [index, n_index]
        }
        n_index += 1
    }
    index += 1
}
return final
```

提交后还是超时，后边想了一下，时间复杂度还是 O(n^2) ，因为 `nums[n_index..<nums.count]` 还在。此时我没法了，看了之前写的代码，用的是 for(index) 套了 for(index - 1) ，这样复杂度是 O(logn)。随后在网上又看到了一个解法利用上了 Dictionary ，直接就是 O(n)，我自己认为应该是最优解了哈哈，代码如下：

```Swift
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    var final = [Int]()
    var dict = [Int: Int]()
    for index in 0..<nums.count {
        guard let lastIndex = dict[target - nums[index]] else {
            dict[nums[index]] = index
            continue
        }
        final.append(index)
        final.append(lastIndex)
    }
    return final
}
```

