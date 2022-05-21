## 有一棵二叉树，请设计一个算法，按照层次打印这棵二叉树
给定二叉树的根结点root，请返回打印结果，结果按照每一层一个数组进行储存，所有数组的顺序按照层数从上往下，且每一层的数组内元素按照从左往右排列。保证结点数小于等于500。


### AC code
```c++
#include <iostream>
#include <vector>
#include <queue>


struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
    val(x), left(NULL), right(NULL) {}
};

class TreePrinter {
public:
    std::vector<std::vector<int> > printTree(TreeNode* root) {
        std::queue<TreeNode *> queue;
        std::vector<std::vector<int> > finalVec;

        if (root) {
            queue.push(root);
            queue.push(NULL);
            std::vector<int> layerVec;
            
            while (!queue.empty()) {
                TreeNode *frontNode = queue.front();
                queue.pop();
                
                if (frontNode) {
                    layerVec.push_back(frontNode->val);

                    if (frontNode->left != NULL) {
                        queue.push(frontNode->left);
                    }
                    if (frontNode->right != NULL) {
                        queue.push(frontNode->right);
                    }
                } else {
                    finalVec.push_back(layerVec);
                    layerVec.clear();
                    if (!queue.empty()) {
                        queue.push(NULL);
                    }
                }
            }
        }
        
        return finalVec;
    }
};

int main(int argc, const char * argv[]) {
    
    TreeNode *rootNode = new TreeNode(1);
    TreeNode *node1 = new TreeNode(2);
    rootNode->left = node1;
    TreeNode *node2 = new TreeNode(3);
    rootNode->right = node2;
    TreeNode *node3 = new TreeNode(4);
    node1->left = node3;
    TreeNode *node4 = new TreeNode(5);
    node1->right = node4;
    TreeNode *node5 = new TreeNode(6);
    node2->left = node5;
    TreeNode *node6 = new TreeNode(7);
    node2->right = node6;
    
    
    TreePrinter *printer = new TreePrinter();
    printer->printTree(rootNode);
        std::vector<std::vector<int>> finalVec = printer->printTree(rootNode);
        for (std::vector<std::vector<int> >::iterator vec = finalVec.begin(); vec != finalVec.end(); vec ++) {
            std::vector<int> tempVec = *vec;
            for (std::vector<int>::iterator instance = tempVec.begin(); instance != tempVec.end(); instance ++) {
                std::cout << *instance << std::ends;
            }
            std::cout << std::endl;
        }
    return 0;
}
```

## 总结
刚开始一点思路都没有，可能是太久没有摸算法了吧，而且也不知道要用上队列，更别说要考虑的一些细节了。总之这道题让我反思了很多，确实是不应该让自己的目光一直放在工程上了。

因为这道题十分经典，网上已经有了大量的题解，左神讲的意思是用一个队列 Q 去管理且用两个变量 last 和 nlash 去做记录，last 作为当前层级的最右节点，nlast 为当前下一层级的最右节点。刚开始先让 last 等于 rootNode ，然后 rootNode push() 入队，接着再让 rootNode pop() 出队，让 nlast 等于 rootNode->leftNode 入队，接着让 nlast 等于 rootNode->rightNode 且入队，此时发现 last 所代表的节点和 出队的 rootNode 相等，换行。继续用 last 等于 root->rightNode ，重复以上过程。

刚开始我觉得云里雾里的，虽然自己也确实没有思路，但是总感觉怪怪的。于是先来纠结一番如何进行按层遍历，也就是宽度优先遍历，当初自己前中后序遍历“嗖嗖”的就写出来了，现如今还得缓好久才知道应该怎么动手，😔。刚开始根本就不知道要用到队列，一直在纠结怎么递归。翻了翻网上的题解，一看到大家都在使用队列思路就开始活跃起来了。

经过一番修整，代码撸出来了，但是跟左神的做法不太一样，没有用到任何标记变量，当前层级如果要结束了直接在 push 进一个 NULL ，后边再去遍历的时候遇到 NULL 就标记着当前层级已经结束了，开始进行下一层级的宽度优先遍历。

再调整一下，代码也撸出来了，但是格式不对，研究了一下，发现是因为 push 进 finalVec 的时机不对，又调整了一下才得以 AC 。


## 二叉树的序列化
没有题意，意思就是要让我们把二叉树进行序列化，做二叉树的序列化左神讲到了一个重点，要进行区分节点的孩子，尤其对于 BFS 来说，需要标识清楚层级关系。

### BFS
```C++
bool serializationBinaryTree(std::vector<std::vector<int> > finalVec) {
        std::ofstream fileOut;
        fileOut.open("traversalOfBinaryTree.txt");
        
        if (!fileOut) {
            return false;
        }
        
        for (std::vector<std::vector<int> >::iterator vec = finalVec.begin(); vec != finalVec.end(); vec ++) {
            std::vector<int> tempVec = *vec;
            for (std::vector<int>::iterator instance = tempVec.begin(); instance != tempVec.end(); instance ++) {
                fileOut << *instance;
            }
            fileOut << std::endl;
        }
        
        fileOut.close();
        return true;
    }
```

### 总结
直接用上了上一题的输出的二维 vector 。

### 二叉树的序列化
二叉树的反序列化相对于序列化来说考虑的事情更多一些。同样，我也将实现二叉树的前中后序以及 BFS 反序列化。

### BFS 
```C++
TreeNode* deserializationBinaryTree() {
        std::ifstream fileIn;
        fileIn.open("traversalOfBinaryTree.txt");
        
        std::vector<std::vector<int>> finalVec;
        
        if (!fileIn) {
            return nullptr;
        }
        
        TreeNode *rootNode = nullptr, *currentNode = nullptr;
        std::queue<int> queue;
        std::queue<TreeNode *> nodeQueue;
        
        while(!fileIn.eof()) {
            std::string tmpString;
            std::getline(fileIn, tmpString);
            
            for (int i = 0; i < tmpString.length(); i ++) {
                queue.push(tmpString[i] - 48);
            }
            // int queue don't accept nullptr, becauese of it's real null pointer
            queue.push(-1);
        }
        
        
        while (!queue.empty()) {
            if (!rootNode) {
                rootNode = new TreeNode(queue.front());
                nodeQueue.push(rootNode);
                queue.pop();
                
                currentNode = rootNode;
            } else {
                if (queue.front() == -1) {
                    queue.pop();
                } else {
                    if (!currentNode->left) {
                        currentNode->left = new TreeNode(queue.front());
                        nodeQueue.push(currentNode->left);
                        queue.pop();
                    } else if (!currentNode->right) {
                        currentNode->right = new TreeNode(queue.front());
                        nodeQueue.push(currentNode->right);
                        queue.pop();
                    } else {
                        currentNode = nodeQueue.front();
                        nodeQueue.pop();
                    }
                }
            }
        }
        
        fileIn.close();

        return rootNode;
    }
```
### 总结
做 BFS 的反序列化纠结得有些久，刚开始也没想到用队列去存 getline 切割出来的内容，到最后想起了左神说的用什么方法进行序列化就用什么方法进行反序列化后，才点醒了我，撸完了第一波代码后，又卡住了，又掉进了死胡同，currentNode 无法和 parentNode 进行交互，也就说到了 rootNode 后，没法再进行循环建树了，差点又要放弃。在去刷牙的过程中，脑子活了一下，发现可以再用一个 nodeQueue 去 push queue pop 出来的当前层 node，遂搞定。
