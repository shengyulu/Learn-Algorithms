# 字典树trie

## trie基本

`字典树`，英文名`Trie树`，Trie一词来自retrieve，发音为/tri:/ “tree”，也有人读为/traɪ/ “try”，
又称`单词查找树` 或 `前缀树`，Trie树，是一种树形结构（多叉树）。

trie，又称为前缀树或字典树，是一种有序树，用于保存关联数组。

1. 除根节点不包含字符，每个节点都包含一个字符
2. 从根节点到某一个节点，路径上经过的字符连接起来，为该节点对应的字符串
3. 每个节点的所有子节点包含的字符都不相同（保证每个节点对应的字符串都不一样）

比如：

```
                    / \    
                   / | \
                  t  a  i                
                /  \     \
               o    e     n
                   /|\    /
                  a d n  n                
```

上面的Trie树，可以表示字符串集合{“a”, “to”, “tea”, “ted”, “ten”, “i”, “in”, “inn”} 。

trie树把每个关键字保存在一条路径上，而不是一个节点中  
两个有公共前缀的关键字，在Trie树中前缀部分的路径相同，所以Trie树又叫做前缀树（Prefix Tree）。  



### trie 优缺点

它的优点是： 

1. 插入和查询的效率很高，都是O(m),其中 m 是待插入/查询的字符串的长度
2. Trie树可以对关键字按字典序排序  
3. 利用字符串的公共前缀来最大限度地减少无谓的字符串比较,提高查询效率

缺点：

1. trie 树比较费内存空间，在处理大数据时会内存吃紧
2. 当hash函数较好时，Hash查询效率比 trie 更优

[知乎这里](http://www.zhihu.com/question/27168319)有个问题：`10万个串找给定的串是否存在`, 对trie和hash两种方案给出了讨论。 


[DATrie](https://github.com/kmike/datrie) 是使用python实现的双数组trie树， 双数组可以减少内存的使用量  。有关 double-array trie，可以参考[这篇论文](http://linux.thai.net/~thep/datrie/datrie.html)
  

### trie应用

典型应用是：前缀查询,字符串查询，排序  
  
* 用于统计，排序和保存大量的字符串（但不仅限于字符串）  
* 经常被搜索引擎系统用于文本词频统计  
* 排序大量字符串   
* 用于索引结构  
* 敏感词过滤  


### 实际应用问题  

1. 给你100000个长度不超过10的单词。对于每一个单词，我们要判断他出没出现过，如果出现了，求第一次出现在第几个位置  
分析思路一：trie树 ，找到这个字符串查询操作就可以了，如何知道出现的第一个位置呢？我们可以在trie树中加一个字段来记录当前字符串第一次出现的位置。  

2. 已知n个由小写字母构成的平均长度为10的单词,判断其中是否存在某个串为另一个串的前缀子串 
  
3. 给出N 个单词组成的熟词表，以及一篇全用小写英文书写的文章，请你按最早出现的顺序写出所有不在熟词表中的生词。  
分析：trie树查询单词的应用。先建立N个熟词的前缀树，然后按文章的单词一次查询。  
  
4. 给出一个词典，其中的单词为不良单词。单词均为小写字母。再给出一段文本，文本的每一行也由小写字母构成。判断文本中是否含有任何不良单词。例如，若rob是不良单词，那么文本problem含有不良单词。
分析：先用不良单词建立trie树，然后过滤文本(每个单词都在trie树上查询，查询的复杂度O(1),效率非常高)，这正是`敏感词过滤系统(或垃圾评论系统)`的原理。  
  
5. 给你N 个互不相同的仅由一个单词构成的英文名，让你将它们按字典序从小到大排序输出
分析：这是trie树排序的典型应用，建立N个单词的trie树，然后线序遍历整个树，就可以达到效果。  




## trie树存储结构和基本操作

最简单实现 ---- 26个字母表 a-z (没有考虑数字，大小写，其他字符如=-*/)
子树用数组存储，浪费空间；如果系统中存在大量字符串，且这些字符串基本没有公共前缀，trie树将消耗大量内存  
如果用链表存储，查询时需要遍历链表，查询效率有所降低



```
define ALPHABET_NUM 26
typedef struct trie_node{
   char value;
   bool isKey;/*是否代表一个关键字*/
   int count; /*可用于词频统计，表示关键字出现的次数*/
   struct Node *subTries[ALPHABET];
}*Trie

Trie Trie_create();
int Trie_insert(Trie trie,char *word); // 插入一个单词
int Trie_search(Trie trie,char *word);// 查找一个单词
int Trie_delete(Trie trie,char *word);// 删除一个单词

Trie Trie_create(){
    trie_node* pNode = new trie_node();
    pNode->count = 0;
    for(int i=0; i<ALPHABET_SIZE; ++i)
        pNode->children[i] = NULL;
    return pNode;
}

void trie_insert(trie root, char* key)
{
    trie_node* node = root;
    char* p = key;
    while(*p)
    {
        if(node->children[*p-'a'] == NULL)
        {
            node->children[*p-'a'] = create_trie_node();
        }
        node = node->children[*p-'a'];
        ++p;
    }
    node->count += 1;
}

/**
 * 查询：不存在返回0，存在返回出现的次数
 */ 
int trie_search(trie root, char* key)
{
    trie_node* node = root;
    char* p = key;
    while(*p && node!=NULL)
    {
        node = node->children[*p-'a'];
        ++p;
    }
    
    if(node == NULL)
        return 0;
    else
        return node->count;
}

```

trie树的增加和删除都比较麻烦，但索引本身就是写少读多，是否考虑添加删除的复杂度上升，依靠具体场景决定。  



