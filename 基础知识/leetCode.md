## TwoSum

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

```
Example:
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

### AC Code

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

### AC Code

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

## 两数相加
这道题难度是 medium ，AC 后我觉得完全没有昨天的 Easy 好做，开始怀疑 LeetCode 是不是搞错了哈哈。做的过程没有感觉到有多困难，但是最后输出 finalNode 的时候只丢出来了最后一个节点，突然想起来这是因为一直都在让 finalNode = finalNode.next ，然后开始陷入纠结中，稍微找到了点思路，肯定要用一个中间 node 去记录当前链，然后把每次生成的新节点添加到中间 node 上去，最后再把中间 node 每次都赋值给 finalNode 。但是这样也有问题，维护这几个 node 的成本太大，差点没绕晕我。

最后还是看了参考答案，整体框架跟我写的都是一致的，只不过在我最纠结的地方，参考答案居然用的是一个新 node 等于了 finalNode，最后直接 return finalNode ， finalNode 根本就不参与这其中的计算，突然觉得这个方法好棒！因为 finalNode 始终都是新 node 的头，不管后续新 node 怎么去变换都不会改变，因为链表头已经被 finalNode 抓住了！

### AC Code

```swift 
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public var val: Int
 *     public var next: ListNode?
 *     public init(_ val: Int) {
 *         self.val = val
 *         self.next = nil
 *     }
 * }
 */
class Solution {
    func addTwoNumbers(_ l1: ListNode?, _ l2: ListNode?) -> ListNode? {
        // 判断是否为空
        if l1 == nil && l2 != nil {
            return l2
        } else if l1 != nil && l2 == nil {
            return l1
        } else if (l1 == nil && l2 == nil) {
            return nil
        } else {
            var finalNode = ListNode(0)
            
            var tempNode = l1
            var otherNode = l2
            var currentNode = finalNode
            
            while true {
                // 判断当前是否为空
                if tempNode != nil {
                    currentNode.val += (tempNode?.val)!
                }
                if otherNode != nil {
                    currentNode.val += (otherNode?.val)!
                }
                
                tempNode = tempNode?.next
                otherNode = otherNode?.next
                
                if currentNode.val - 10 >= 0 {
                    currentNode.val = currentNode.val - 10
                    currentNode.next = ListNode(1)
                    currentNode = currentNode.next!
                } else {
                    if tempNode == nil && otherNode == nil {
                        break
                    }
                    currentNode.next = ListNode(0)
                    currentNode = currentNode.next!
                }
            }
            
            return finalNode
        }
    }
}
```

## 两个排序数组的中位数
惊呆了，用自己最初的想法居然直接一把 AC 掉了困难题目，真不知道这困难是不是放错了哈哈哈，总之很开心就是了。刚开始想的巨多，一直在纠结怎么把两个有序的数组用一个较好的方法直接合并，然后又考虑到了题目是个有序数组，接着想到了用二分balabala，总之就是题还没开始写，我就已经想得乱七八糟，最后差点被自己吓屎去翻参考答案了，这又给了我一个提醒，做题之前确实是要好好的构思题目怎么来，但是要注意不要想太多，因为其实很多东西都是水到渠成的hhhh

```swift
func findMedianSortedArrays(_ nums1: [Int], _ nums2: [Int]) -> Double {
        var finalArray: Array<Int> = []
        var i = 0, j = 0
        
        // 合并两个有序数组
        while (i < nums1.count && j < nums2.count)  {
            if (nums1[i] < nums2[j]) {
                finalArray.append(nums1[i])
                i += 1
            } else {
                finalArray.append(nums2[j])
                j += 1
            }
        }
        
        // 添加剩余内容
        while true {
            if i < nums1.count {
                finalArray.append(nums1[i])
                i += 1
            }
            if j < nums2.count {
                finalArray.append(nums2[j])
                j += 1
            }
            if i >= nums1.count && j >= nums2.count {
                break
            }
        }
        
        // 返回中位数
        if finalArray.count % 2 != 0 {
            return Double(finalArray[finalArray.count / 2])
        } else {
            let v = (finalArray[finalArray.count / 2] + finalArray[finalArray.count / 2 - 1])
            if v % 2 != 0 {
                return Double(v / 2) + 0.5
            }
            return Double(v / 2)
        }
    }
```