# 组合模式

组合模式又叫作部分整体模式，主要用于实现部分和整体操作的一致性。组合模式常根据树形结构来表示部分及整体之间的关系，使得用户对单个对象和组合对象的操作具有一致性。

组合模式通过特定的数据结构简化了部分和整体之间的关系，使得客户端可以像处理单个元素一样来处理整体的数据集，而无须关心单个元素和整体数据之间的内部复杂结构。

组合模式以类似树形结构的方式实现整体和部分之间关系的组合。

如类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/组合模式类图.png)

1. #### 定义TreeNode

```java
public class TreeNode {

    private String name;
    private TreeNode parent;
    private Vector<TreeNode> children = new Vector<>();

    public TreeNode(String name) {
        this.name = name;
    }

    public TreeNode getParent() {
        return parent;
    }

    public void setParent(TreeNode parent) {
        this.parent = parent;
    }

    public void add(TreeNode node) {
        children.add(node);
    }

    public void remove(TreeNode node) {
        children.remove(node);
    }

    public Enumeration<TreeNode> getChildren() {
        return children.elements();
    }
}
```

2. #### 测试类

```java
public class TreeNodeTest {

    public static void main(String[] args) {
        TreeNode nodeA = new TreeNode("A");
        TreeNode nodeB = new TreeNode("B");
        nodeA.add(nodeB);
        System.out.println(JSON.toJSONString(nodeA));
    }
}
```

