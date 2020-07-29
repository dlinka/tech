ACNode

```java
@Data
@NoArgsConstructor
public class ACNode {
    
    private Character value;
    private HashMap<Character, ACNode> child = new HashMap<>();
    private boolean isEnding = false;
    private ACNode fail;
    private int deep; //树的深度,用于计算匹配字符长度

    public ACNode(Character value, int deep) {
        this.value = value;
        this.deep = deep;
    }

    @Override
    public String toString() {
        return "ACNode{" +
                "value=" + value +
                ", childSize=" + child.size() +
                ", isEnding=" + isEnding +
                ", failValue=" + fail.getValue() +
                ", deep=" + deep +
                "}";
    }

}
```

ACBuilder

```java
public class ACBuilder {

    private ACNode ROOT = new ACNode();

    /**
     * 构建树
     */
    private void buildTree(String[] words) {
        for (String word : words) {
            char[] characters = word.toCharArray();
            ACNode node = this.ROOT;

            for (int i = 0; i < characters.length; i++) {
                Character key = characters[i];
                if (node.getChild().containsKey(key)) {
                    node = node.getChild().get(key);
                } else {
                    ACNode newNode = new ACNode(key, i + 1);
                    if (i == characters.length - 1) {
                        newNode.setEnding(true);
                    }
                    node.getChild().put(key, newNode);
                    node = newNode;
                }
            }
        }
    }

    /**
     * 构建失配指针
     */
    private void buildFail() {
        LinkedList<ACNode> queue = new LinkedList<>();
        queue.add(ROOT);
        while (!queue.isEmpty()) {
            ACNode parent = queue.poll();
            for (ACNode child : parent.getChild().values()) {
                if (parent == ROOT) {
                    child.setFail(ROOT);
                } else {
                    ACNode fail = parent.getFail();

                    while (fail != null) {
                        if (fail.getChild().get(child.getValue()) != null) {
                            child.setFail(fail.getChild().get(child.getValue()));
                            break;
                        } else {
                            fail = fail.getFail();
                            if (fail == null) {
                                child.setFail(ROOT);
                                break;
                            }
                        }
                    }
                }

                if (child.getChild().size() > 0) {
                    queue.add(child);
                }
            }
        }
    }

    public ACNode build(String[] words) {
        buildTree(words);
        buildFail();
        return ROOT;
    }

}
```

IllegalWordUtil

```java
public class IllegalWordUtil {

    /**
     * 查找所有匹配字符串
     */
    public static HashSet<String> match(ACNode root, String txt) {
        HashSet<String> result = new HashSet<>();

        char[] chars = txt.toCharArray();
        int index = 0;
        ACNode node = root;
        while (index < chars.length) {

            if (node.getChild().containsKey(chars[index])) {
                node = node.getChild().get(chars[index]);
                index++;
                if (node.isEnding()) {
                    result.add(txt.substring(index - node.getDeep(), index));
                }
            } else {
                node = node.getFail();
                if (node == null) {
                    node = root;
                    index++;
                }
            }
        }
        return result;
    }

    public static void main(String[] args) throws IOException {
        String[] words = {"姜子牙", "姜尚", "太公", "太傅", "姜太公", "姜太公钓鱼愿者上钩", "子牙"};
        ACNode node = new ACBuilder().build(words);
        HashSet<String> match = IllegalWordUtil.match(node, "啊阿姜子库啦啦愿者上钩子发太公");
        System.out.println("命中数量 - " + match.size());
        System.out.println("命中词词 - " + match);
    }

}
```

---

