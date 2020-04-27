# 用前缀树过滤敏感词


<!--more-->

### 前缀树**

​	**名称：**Trie、字典树、查找树

​	**特点：**查找效率高，消耗内存大

​	**应用：**字符串检索、词频统计、字符串排序等

### **敏感词过滤器**

​	定义前缀树根据敏感词

​	初始化前缀树

​	编写过滤敏感词的方法

</br>

简单的进行查找和replace替换无疑效率是非常低的，我们使用一种叫前缀树（Trie、字典树、查找树）的数据结构来实现高效过滤敏感词，数据结构及过滤图示如下：

![过滤敏感词](https://i.loli.net/2020/01/27/W93QTOZsYUSN6IB.png)

**Step1 定义敏感词文件sensitive-words.txt，放在resources目录下**

```properties
VPN
翻墙
吸毒
开票
```

**Step2 敏感词过滤器**

```java
// 敏感词过滤器
@Component
public class SensitiveFilter {

    // 记录日志
    private static final Logger logger = LoggerFactory.getLogger(SensitiveFilter.class);

    // 替换符（检测到敏感词，替换为某符号，定义为常量）
    private static final String REPLACEMENT = "***";

    // 初始化前缀树
    private TrieNode rootNode = new TrieNode();


    // @PostConstruct表示这是一个初始化方法，当容器实例化这个Bean以后在调用它的构造器之后这个方法就会被自动调用
    @PostConstruct
    public void init() {
        // 在classes路径下读取敏感词文件（需要先编译），获得字节流
        try (
                InputStream is = this.getClass().getClassLoader().getResourceAsStream("sensitive-words.txt");
                // 转为字符流，在转为缓冲流(效率会更高)
                BufferedReader reader = new BufferedReader(new InputStreamReader(is));
        ) {
            String keyword;
            while ((keyword = reader.readLine()) != null) {
                // 添加到前缀树中，这一步比较复杂，封装为一个方法
                this.addKeyword(keyword);
            }
        } catch (IOException e) {
            logger.error("加载敏感词文件失败 : " + e.getMessage());
        }
    }

    // 将一个敏感词添加到前缀树中
    private void addKeyword(String keyword) {
        // 指针
        TrieNode tempNode = rootNode;
        for (int i = 0; i < keyword.length(); i++) {
            char c = keyword.charAt(i);
            TrieNode subNode = tempNode.getSubNode(c);
            if (subNode == null) {
                // 初始化子节点
                subNode = new TrieNode();
                tempNode.addSubNode(c, subNode);
            }

            // 让指针指向该子节点，进入下一轮循环
            tempNode = subNode;

            // 设置结束标识(到此为一个敏感词)
            if (i == keyword.length() - 1) {
                tempNode.setKeywordEnd(true);
            }
        }
    }


    /**
     * 过滤敏感词
     *
     * @param text 待过滤的文本
     * @return 过滤后的文本
     */
    public String filter(String text) {
        if (StringUtils.isBlank(text)) {
            return null;
        }

        // 指针1，指向前缀树
        TrieNode tempNode = rootNode;
        // 指针2，字符串指针
        int begin = 0;
        // 指针3，字符串指针
        int position = 0;
        // 记录生成的新字符串，使用可变长字符串
        StringBuilder sb = new StringBuilder();

        // 开始过滤
        while (position < text.length()) {
            char c = text.charAt(position);

            // 跳过符号，防止有的用户通过在敏感词中穿插各种符号来规避敏感词检测
            if (isSymbol(c)) {
                // 若指针1处于根节点，将此符号记入结果，让指针2向下走一步
                if (tempNode == rootNode) {
                    sb.append(c);
                    begin++;
                }
                // 无论符号在开头或中间，指针3都向下走一步
                position++;
                continue;
            }

            // 检查下级节点
            tempNode = tempNode.getSubNode(c);
            if (tempNode == null) {
                // 以begin为开头的字符串不是敏感词
                sb.append(text.charAt(begin));
                // 进入下一个位置
                position = ++begin;
                // 重新指向根节点
                tempNode = rootNode;
            } else if (tempNode.isKeywordEnd()) {
                // 发现了敏感词，将begin到position这段的字符串替换掉
                sb.append(REPLACEMENT);
                // 进入下一个位置
                begin = ++position;
                // 重新指向根节点
                tempNode = rootNode;
            } else {
                // 检查下一个字符
                position++;
            }
        }

        // 将最后一批字符计入结果
        sb.append(text.substring(begin));

        // 返回结果
        return sb.toString();
    }

    // 判断是否为符号
    private boolean isSymbol(Character c) {
        // 不是特殊符号，0x2E80到0x9FFF是东亚的文字范围
        return !CharUtils.isAsciiAlphanumeric(c) && (c < 0x2E80 || c > 0x9FFF);
    }


    // 前缀树
    private class TrieNode {

        // 关键词结束标识
        private boolean isKeywordEnd = false;

        // 子节点(key是下级字符，value是下级节点)
        private Map<Character, TrieNode> subNodes = new HashMap<>();

        public boolean isKeywordEnd() {
            return isKeywordEnd;
        }

        public void setKeywordEnd(boolean keywordEnd) {
            isKeywordEnd = keywordEnd;
        }

        // 添加子节点
        public void addSubNode(Character c, TrieNode node) {
            subNodes.put(c, node);
        }

        //  获取子节点
        public TrieNode getSubNode(Character c) {
            return subNodes.get(c);
        }

    }

}
```

**Step3 过滤测试**

```java
@SpringBootTest
@ContextConfiguration(classes = CommunityApplication.class)
public class SensitiveTests {

    @Autowired
    private SensitiveFilter sensitiveFilter;

    @Test
    public void testSensitiveFilter(){
        String text = "我是敏感词，你来过滤我啊，测试VPN，测试翻墙，测试吸毒，测试开票，哈哈哈！";
        System.out.println(sensitiveFilter.filter(text));

        text = "我是敏感词，你来过滤我啊，测试☆V☆P☆☆N☆，测试☆☆翻☆☆☆墙☆☆,☆测试☆吸☆☆毒☆☆☆，测试☆开☆☆票☆，哈哈哈！";
        System.out.println(sensitiveFilter.filter(text));
    }

}
```

输出结果：

```properties
我是敏感词，你来过滤我啊，测试***，测试***，测试***，测试***，哈哈哈！
我是敏感词，你来过滤我啊，测试☆***☆，测试☆☆***☆☆,☆测试☆***☆☆☆，测试☆***☆，哈哈哈！
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
