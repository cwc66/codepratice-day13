# codepratice-day13
代码随想录第13次

二叉树第三次练习

递归、回溯、迭代内容。
重点理解回溯

[平衡二叉树](https://leetcode.cn/problems/balanced-binary-tree/)
给定一个二叉树，判断它是否是平衡二叉树。
平衡二叉树是指该树的节点的左右子树的高度相差不超过1.

二叉树节点的深度：
    从根节点到该节点的最长简单路径边的条数。

二叉树节点的高度：
    从该节点到叶子节点的最长简单路径边的条数。


求深度从上到下查，需要前序遍历（中左右），高度只能从下到上查，只能后序遍历（左右中）。

求二叉树最大深度时，由于求的是二叉树的根节点的高度，根节点的高度就是这棵树的最大深度。可以用后序遍历。

递归三部曲
1.明确递归函数的参数和返回值
参数 传入节点
返回值 以当前传入节点为根节点的树的高度

2.明确终止条件
递归过程中遇到空节点为终止，返回0，表示当前节点为根节点的树，高度为0.

3.明确单层递归的逻辑
判断以当前传入节点为根节点的二叉树是否是平衡二叉树： 左子树高度与右子树高度的差。

分别求左右子树高度，如果插值小于等于1，返回当前二叉树高度，否则返回-1表示已经不是平衡二叉树

```CPP
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int getheight(TreeNode* node)
    {
        if(node==nullptr)
        {
            return 0;
        }
        int leftheight=getheight(node->left);
        if(leftheight==-1) return -1;
        int rightheight=getheight(node->right);
        if(rightheight==-1) return -1;
        //求节点的高度，某节点的高度是左右子树高度最大值+1
        return abs(leftheight-rightheight)>1 ? -1:1+max(leftheight,rightheight);
    }
    bool isBalanced(TreeNode* root) {
        return getheight(root)==-1? false:true;
    }
};
```

迭代法：
通过求传入节点为根节点的最大深度来求的高度。用栈模拟后序遍历。
也相当于模板，模板改写.      定义一个函数，专门求高度
```CPP
class Solution{
private:
    int getDepth(TreeNode* cur)
    {
        stack<TreeNode*> st;
        if(cur!=nullptr) st.push(cur);
        int depth=0;
        int result=0;
        while(!st.empty())
        {
            TreeNode* node=st.top();
            if(node!=nullptr)
            {
                st.pop();
                st.push(node);
                st.push(nullptr);
                depth++;
                if(node->right) st.push(node->right);
                if(node->left) st.push(node->left);
            }
            else
            {
                st.pop();
                node=st.top();
                st.pop();
                depth--;
            }
            result=result>depth?result:depth;
        }
    }

public://模板 迭代
    bool isBalanced(TreeNode* root)
    {
        stack<TreeNode*> st;
        if(root==nullptr) return true;
        st.push(root);
        while(!st.empty())
        {
            TreeNode* node=st.top();
            st.pop();
            if(abs(getDepth(node->left)-getDepth(node->right))>1)
            {
                return false;
            }
            if(node->right) st.push(node->right);
            if(node->left) st.push(node->left);
        }
        return true;
    }
}

```

[二叉树的所有路径](https://leetcode.cn/problems/binary-tree-paths/description/)
给你一个二叉树的根节点root,按任意顺序，返回所有从根节点到叶子节点的路径。

前序遍历，方便让父节点指向孩子节点，找到对应的路径。设计回溯，因为要把路径记录下来，需要回溯来回退一个路径，进入另外的路径。

1.递归函数参数和返回值
根节点，记录每一条路径path，存放结果集result,递归不需要返回值。

2.确定递归终止条件
一半是节点为空，终止。但是这里要找叶子节点，开始结束的处理逻辑。
当cur不为空，但其左右孩子为空（条件

这里用vector类的path记录路径，需要把path转化为string，再放到result里。（方便做回溯

3.确定单层递归逻辑
path.push_back(cur->val);
判断cur是否为空，否 不进行递归，在递归后进行回溯(pop_back出一个元素)

```CPP
// 版本一
class Solution {
private:

    void traversal(TreeNode* cur, vector<int>& path, vector<string>& result) {
        path.push_back(cur->val); // 中，中为什么写在这里，因为最后一个节点也要加入到path中 
        // 这才到了叶子节点
        if (cur->left == NULL && cur->right == NULL) {
            string sPath;
            for (int i = 0; i < path.size() - 1; i++) {
                sPath += to_string(path[i]);
                sPath += "->";
            }
            sPath += to_string(path[path.size() - 1]);
            result.push_back(sPath);
            return;
        }
        if (cur->left) { // 左 
            traversal(cur->left, path, result);
            path.pop_back(); // 回溯
        }
        if (cur->right) { // 右
            traversal(cur->right, path, result);
            path.pop_back(); // 回溯
        }
    }

public:
    vector<string> binaryTreePaths(TreeNode* root) {
        vector<string> result;
        vector<int> path;
        if (root == NULL) return result;
        traversal(root, path, result);
        return result;
    }
};
```
精简版：
用string path，复制复制，没使用引用，否则无法回溯。
回溯对应traversal(cur->left,path+"->",result);中path+"->"，每次调用完，path没有加上"->"。
```CPP
class Solution {
private:

    void traversal(TreeNode* cur, string path, vector<string>& result) {
        path += to_string(cur->val); // 中
        if (cur->left == NULL && cur->right == NULL) {
            result.push_back(path);
            return;
        }
        if (cur->left) traversal(cur->left, path + "->", result); // 左
        if (cur->right) traversal(cur->right, path + "->", result); // 右
    }

public:
    vector<string> binaryTreePaths(TreeNode* root) {
        vector<string> result;
        string path;
        if (root == NULL) return result;
        traversal(root, path, result);
        return result;

    }
};
```

能够展现回溯过程对应代码：
递归结束，上一层path值不受影响（没有引用）相当于回溯，一开始用vector因为如果用pop弹出string类型元素，数值对应的弹出字符个数不确定。

```CPP
//版本二
class Solution {
private:
    void traversal(TreeNode* cur, string path, vector<string>& result) {
        path += to_string(cur->val); // 中，中为什么写在这里，因为最后一个节点也要加入到path中
        if (cur->left == NULL && cur->right == NULL) {
            result.push_back(path);
            return;
        }
        if (cur->left) {
            path += "->";
            traversal(cur->left, path, result); // 左
            path.pop_back(); // 回溯 '>'
            path.pop_back(); // 回溯 '-'
        }
        if (cur->right) {
            path += "->";
            traversal(cur->right, path, result); // 右
            path.pop_back(); // 回溯'>'
            path.pop_back(); // 回溯 '-'
        }
    }

public:
    vector<string> binaryTreePaths(TreeNode* root) {
        vector<string> result;
        string path;
        if (root == NULL) return result;
        traversal(root, path, result);
        return result;

    }
};
```

[左叶子之和](https://leetcode.cn/problems/sum-of-left-leaves/)
给定二叉树的根节点 root ，返回所有左叶子之和。

判断左叶子，不是二叉树左侧节点。

节点A的左孩子不为空，且左孩子的左右孩子都为空（说明是叶子节点），那么A节点的左孩子为左叶子节点

判断当前节点是不是左叶子是无法判断的，必须要通过节点的父节点来判断其左孩子是不是左叶子。

1.确定递归函数的参数和返回值
判断一个树的左叶子节点之和，那么一定要传入树的根节点，递归函数的返回值为数值之和，所以为int

2.确定终止条件
空节点 左叶子值一定位0
遍历节点为叶子节点，左叶子也为0

3.确定单层递归的逻辑
当遇到左叶子节点的时候，记录数值，然后通过递归求取左子树左叶子之和，和 右子树左叶子之和，相加便是整个树的左叶子之和。


```CPP
class Solution {
public:
    int sumOfLeftLeaves(TreeNode* root) {
        if (root == NULL) return 0;
        if (root->left == NULL && root->right== NULL) return 0;

        int leftValue = sumOfLeftLeaves(root->left);    // 左
        if (root->left && !root->left->left && !root->left->right) { // 左子树就是一个左叶子的情况
            leftValue = root->left->val;
        }
        int rightValue = sumOfLeftLeaves(root->right);  // 右

        int sum = leftValue + rightValue;               // 中
        return sum;
    }
};


```
精简代码(主要先学第一版：
```CPP
class Solution {
public:
    int sumOfLeftLeaves(TreeNode* root) {
        if (root == NULL) return 0;
        int leftValue = 0;
        if (root->left != NULL && root->left->left == NULL && root->left->right == NULL) {
            leftValue = root->left->val;
        }
        return leftValue + sumOfLeftLeaves(root->left) + sumOfLeftLeaves(root->right);
    }
};
```

迭代法
模板
```CPP
class Solution {
public:
    int sumOfLeftLeaves(TreeNode* root) {
        stack<TreeNode*> st;
        if (root == NULL) return 0;
        st.push(root);
        int result = 0;
        while (!st.empty()) {
            TreeNode* node = st.top();
            st.pop();
            if (node->left != NULL && node->left->left == NULL && node->left->right == NULL) {
                result += node->left->val;
            }
            if (node->right) st.push(node->right);
            if (node->left) st.push(node->left);
        }
        return result;
    }
};

```
通常用节点的左右孩子判断当前节点属性。
这道题要用节点的父节点来判断当前节点属性。

[完全二叉树的节点个数](https://leetcode.cn/problems/count-complete-tree-nodes/)
给你一棵 完全二叉树 的根节点 root ，求出该树的节点个数。

完全二叉树的定义如下：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 h 层（从第 0 层开始），则该层包含 1~ 2^h 个节点。

普通二叉树逻辑
后序（左右中）
递归

1.确定递归函数的参数和返回值：
参数就是传入树的根节点，返回就返回以该节点为根节点二叉树的节点数量，所以返回值为int类型。

2.确定终止条件：
如果为空节点的话，就返回0，表示节点数为0。

3.确定单层递归的逻辑：
先求它的左子树的节点数量，再求右子树的节点数量，最后取总和再加一 （加1是因为算上当前中间节点）就是目前节点为根节点的节点数量。

```CPP
// 版本一
class Solution {
private:
    int getNodesNum(TreeNode* cur) {
        if (cur == NULL) return 0;
        int leftNum = getNodesNum(cur->left);      // 左
        int rightNum = getNodesNum(cur->right);    // 右
        int treeNum = leftNum + rightNum + 1;      // 中
        return treeNum;
    }
public:
    int countNodes(TreeNode* root) {
        return getNodesNum(root);
    }
};
```
精简版
```CPP
// 版本二
class Solution {
public:
    int countNodes(TreeNode* root) {
        if (root == NULL) return 0;
        return 1 + countNodes(root->left) + countNodes(root->right);
    }
};
```

迭代 模板
```CPP
class Solution {
public:
    int countNodes(TreeNode* root) {
        queue<TreeNode*> que;
        if (root != NULL) que.push(root);
        int result = 0;
        while (!que.empty()) {
            int size = que.size();
            for (int i = 0; i < size; i++) {
                TreeNode* node = que.front();
                que.pop();
                result++;   // 记录节点数量
                if (node->left) que.push(node->left);
                if (node->right) que.push(node->right);
            }
        }
        return result;
    }
};
```

完全二叉树性质 解法

完全二叉树只有两种情况
1.满二叉树
2.最后一层叶子节点没有满

1-> 直接用2^树的深度 -1计算。
2-> 分别递归左右孩子，递归到某一深度，一定有孩子为满二叉树，以1计算。
判断满二叉树：递归向左遍历的深度 等于 递归向右遍历的深度。
如果不是满二叉树，继续递归

```CPP
class Solution {
public:
    int countNodes(TreeNode* root) {
        if (root == nullptr) return 0;
        TreeNode* left = root->left;
        TreeNode* right = root->right;
        int leftDepth = 0, rightDepth = 0; // 这里初始为0是有目的的，为了下面求指数方便
        while (left) {  // 求左子树深度
            left = left->left;
            leftDepth++;
        }
        while (right) { // 求右子树深度
            right = right->right;
            rightDepth++;
        }
        if (leftDepth == rightDepth) {
            return (2 << leftDepth) - 1; // 注意(2<<1) 相当于2^2，所以leftDepth初始为0
        }
        return countNodes(root->left) + countNodes(root->right) + 1;
    }
};
```
时间复杂度O(N*logN),因为N个节点，每个节点需要递归左右孩子，每次计算深度需要logN。

