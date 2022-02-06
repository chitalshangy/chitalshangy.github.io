---
layout:     post
title:      leetcode刷题记录
subtitle:   
date:       2021-12-24
author:     chital
header-img: img/post-bg-alibaba.jpg
catalog: true
tags:
    - 刷题
---

### 2021-12-24 
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/maximum-number-of-eaten-apples/)
  **分析：优先队列 + 贪心**
  ```
  public int eatenApples(int[] apples, int[] days) {
        Queue<int[]> tool = new PriorityQueue<int[]>((a, b) -> (a[0] - b[0]));
        int res = 0;
        int n = apples.length;
        for(int i = 0; i < n; ++i) {
            if(apples[i] != 0 && days[i] != 0) {
                tool.offer(new int[]{i + days[i], apples[i]});
            }
            /*
            **这里注意一下，第一次写的时候用的是 < 而不是 <=，导致出错，原因在于对题目的理解不够，i + days[i]这一天即是当前苹果腐烂的时间；
            */
            while(!tool.isEmpty() && tool.peek()[0] <= i) {
                tool.poll();
            }
            if(!tool.isEmpty()) {
                int[] tmp = tool.poll();
                res++;
                if(tmp[1] - 1 > 0) {
                    tool.offer(new int[]{tmp[0], tmp[1] - 1});
                }
            }
        }
        while(!tool.isEmpty()) {
            int[] tmp = tool.poll();
            int cur = Math.min(tmp[0] - n, tmp[1]);
            res += cur;
            n += cur;
        }
        return res;
    }
  ```

### 2021-12-25
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/even-odd-tree/)
  **分析：层序遍历；一开始写的不够精简；看了大佬的题解后才明白自己写得多么复杂；还是思路缺乏完整性的问题：一方面自己想的太多，确实全都想到了，但是是一个层叠的过程，就会导致在写题目过程中代码不断增加、复杂；反思一下，好好写草稿；**
  ```
  public boolean isEvenOddTree(TreeNode root) {
        if(root == null) {
            return true;
        }
        Queue<TreeNode> tool = new LinkedList<TreeNode>();
        tool.offer(root);
        boolean flag = true;
        while(!tool.isEmpty()) {
            int curSize = tool.size();
            int k = flag ? Integer.MIN_VALUE : Integer.MAX_VALUE;
            for(int i = 0; i < curSize; ++i) {
                TreeNode curNode = tool.poll();
                if(flag && (curNode.val <= k || curNode.val % 2 == 0)) {
                    return false;
                }
                if(!flag && (curNode.val >= k || curNode.val % 2 != 0)) {
                    return false;
                }
                k = curNode.val;
                if(curNode.left != null) {
                    tool.offer(curNode.left);
                }
                if(curNode.right != null) {
                    tool.offer(curNode.right);
                }
            }
            flag = !flag;
        }
        return true;
    }
  ```

### 2021-12-26
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/occurrences-after-bigram/)
  **分析：简单模拟**
  ```
  public String[] findOcurrences(String text, String first, String second) {
        String[] words = text.split(" ");
        List<String> list = new ArrayList<String>();
        for (int i = 2; i < words.length; i++) {
            if (words[i - 2].equals(first) && words[i - 1].equals(second)) {
                list.add(words[i]);
            }
        }
        int size = list.size();
        String[] ret = new String[size];
        for (int i = 0; i < size; i++) {
            ret[i] = list.get(i);
        }
        return ret;
    }
  ```

### 2021-12-27
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/friends-of-appropriate-ages/submissions/)
  **分析：排序+搜索，手写一个快排**
  ```
  public int numFriendRequests(int[] ages) {
        qSort(ages, 0, ages.length - 1);
        int res = 0, l = 0, r = 0;
        for(int i = 0; i < ages.length; ++i) {
            if(ages[i] > 14) {
                while(ages[l] <= 0.5 * ages[i] + 7) {
                    l++;
                }
                while(r + 1 < ages.length && ages[r + 1] <= ages[i]) {
                    r++;
                }
                res += r - l;
            }
        }
        return res;
    }

    void qSort(int[] ages, int l, int r) {
        if(l > r) {
            return ;
        }

        int i = l, j = r;
        int k = l;
        while(i < j) {
            while(i < j && ages[j] >= ages[k]) {
                j--;
            }
            while(i < j && ages[i] <= ages[k]) {
                i++;
            }
            swap(ages, i, j);
        }
        swap(ages, i, k);
        qSort(ages, l, i - 1);
        qSort(ages, i + 1, r);
    }

    void swap(int[] ages, int i, int j) {
        int tmp = ages[i];
        ages[i] = ages[j];
        ages[j] = tmp;
    }
  ```

### 2021-12-28
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/concatenated-words/submissions/)
  **分析：字典树+dfs**
  ```
  Trie root;
    public List<String> findAllConcatenatedWordsInADict(String[] words) {
        root = new Trie();
        Arrays.sort(words, (a, b) -> a.length() - b.length());
        List<String> res = new ArrayList<String>();
        for(String word : words) {
            if(word.length() != 0) {
                if(check(word, 0)) {
                    res.add(word);
                }
                else {
                    insert(word);
                }
            } 
        }
        return res;
    }

    boolean check(String str, int begin) {
        if(str.length() == begin) {
            return true;
        }
        Trie curNode = root;
        for(int i = begin; i < str.length(); ++i) {
            if(curNode.child[str.charAt(i) - 'a'] == null) {
                return false;
            }
            curNode = curNode.child[str.charAt(i) - 'a'];
            if(curNode.isEnd) {
                if(check(str, i + 1)) {
                    return true;
                }
            }
        }
        return false;
    }

    void insert(String str) {
        Trie curNode = root;
        for(char c : str.toCharArray()) {
            if(curNode.child[c - 'a'] == null) {
                curNode.child[c - 'a'] = new Trie();
            }
            curNode = curNode.child[c - 'a'];
        }
        curNode.isEnd = true;
    }
    class Trie {
        Trie[] child;
        boolean isEnd;

        public Trie() {
            child = new Trie[26];
            isEnd = false;
        }
    }
  ```

### 2021-12-29
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/count-special-quadruplets/)
  **分析：模拟（这道题目直接看题解的）**
  ```
  public int countQuadruplets(int[] nums) {
        int n = nums.length, ans = 0;
        for (int a = 0; a < n; a++) {
            for (int b = a + 1; b < n; b++) {
                for (int c = b + 1; c < n; c++) {
                    for (int d = c + 1; d < n; d++) {
                        if (nums[a] + nums[b] + nums[c] == nums[d]) ans++;
                    }
                }
            }
        }
        return ans;
    }
  ```

### 2021-12-30
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/hand-of-straights/)
  **分析：模拟（抄的题解）**
  ```
  public boolean isNStraightHand(int[] hand, int groupSize) {
        int n = hand.length;
        if(n % groupSize != 0) {
            return false;
        }
        PriorityQueue<Integer> tool = new PriorityQueue<Integer>((a, b) -> (a - b));
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for(int i : hand) {
            map.put(i, map.getOrDefault(i, 0) + 1);
            tool.offer(i);
        }

        while(!tool.isEmpty()) {
            int cur = tool.poll();
            if(map.getOrDefault(cur, 0) != 0) {
                for(int i = 0; i < groupSize; ++i) {
                    int tmp = map.getOrDefault(cur + i, 0);
                    if(tmp == 0) {
                        return false;
                    }
                    map.put(cur + i, tmp - 1);
                }
            }
        }
        return true;
    }
  ```

### 2021-12-31
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/perfect-number/)
  **分析：模拟**
  ```
  public boolean checkPerfectNumber(int num) {
        if(num == 1) {
            return false;
        }
        int sum = 1;
        for(int i = 2; i * i <= num; ++i) {
            if(num % i == 0) {
                sum += i + num / i;
            }
        }
        return sum == num;
    }
  ```

### 2022-1-1
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/convert-1d-array-into-2d-array/)
  **分析：模拟，坐标映射：x = pos / n，y = pos % n；**
  ```
  public int[][] construct2DArray(int[] original, int m, int n) {
        if(original.length != m * n) {
            return new int[0][0];
        }
        int[][] res = new int[m][n];
        for(int i = 0; i < original.length; ++i) {
            res[i / n][i % n] = original[i];
        }
        return res;
    }
  ```

### 2022-1-2
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/elimination-game/)
  **分析：约瑟夫环问题，看了题解之后才理解，设f(i)为[1, i]中从左到右的解，f'(i)为[1, i]中从右到左的解，得出一个公式，然后根据约瑟夫德特性，得出另外一个公式，解出迭代公式即可；约瑟夫迭代映射关系：i = (j + k - 1) % n + 1（备忘，不适用于这道题目）**
  ```
  public int lastRemaining(int n) {
        if(n == 1) {
            return 1;
        }
        else {
            return 2 * (n / 2 + 1 - lastRemaining(n / 2));
        }
    }
  ```

### 2022-1-3
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/day-of-the-week/)
  **分析：抄题解的；分段计算**
  ```
  public String dayOfTheWeek(int day, int month, int year) {
        String[] week = {"Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"};
        int[] monthDays = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30};
        /* 输入年份之前的年份的天数贡献 */
        int days = 365 * (year - 1971) + (year - 1969) / 4;
        /* 输入年份中，输入月份之前的月份的天数贡献 */
        for (int i = 0; i < month - 1; ++i) {
            days += monthDays[i];
        }
        if ((year % 400 == 0 || (year % 4 == 0 && year % 100 != 0)) && month >= 3) {
            days += 1;
        }
        /* 输入月份中的天数贡献 */
        days += day;
        return week[(days + 3) % 7];
    }
  ```

### 2022-1-4
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/cat-and-mouse/submissions/)
  **分析：动态规划博弈问题；分析题目，列出公式，对于数组 dp[i][j][k]，i 表示鼠所在的位置，j表示猫所在的位置，k 表示游戏进行的轮数；i = 0，说明鼠赢；i == j，猫赢；k >= 2 * n，说明平局；初始状态为 dp[1][2][0]，在此基础上进行深度优先搜索，对于当前状态 i 可到达的每一个状态进行搜索，最好赢，其次平，实在不行就只能输，在当前状态的结果态确定后，可达到当前状态的状态的结果态也已经确定，那么按照这个逻辑，最终即可得出答案。**
  ```
    int n;
    int [][][]dp;
    int [][]g;
    public int catMouseGame(int[][] graph) {
        n = graph.length;
        dp = new int[n][n][2 * n];
        g = graph;
        for(int i = 0; i < n; ++i) {
            for(int j = 0; j < n; ++j) {
                for(int k = 0; k < 2 * n; ++k) {
                    dp[i][j][k] = -1;
                }
            }
        }
        return fun(1, 2, 0);
    }

    int fun(int i, int j, int k) {
        if(k == 2 * n) {
            return 0;
        }
        int res = dp[i][j][k];
        if(i == 0) {
            res = 1;
        }
        else if(i == j) {
            res = 2;
        }
        else if(k >= 2 * n) {
            res = 0;
        }
        else if(res == -1){
            if(k % 2 == 0) {
                boolean win = false, draw = false;
                for(int mem : g[i]) {
                    int cur = fun(mem, j, k + 1);
                    if(cur == 1) {
                        win = true;
                    }
                    else if(cur == 0) {
                        draw = true;
                    }
                    if(win) {
                        break;
                    }
                }
                if(win) {
                    res = 1;
                }
                else if(draw) {
                    res = 0;
                }
                else {
                    res = 2;
                }
            }
            else {
                boolean win = false, draw = false;
                for(int mem : g[j]) {
                    if(mem != 0) {
                        int cur = fun(i, mem, k + 1);
                        if(cur == 2) {
                            win = true;
                        }
                        else if(cur == 0) {
                            draw = true;
                        }
                        if(win) {
                            break;
                        }
                    }
                }
                if(win) {
                    res = 2;
                }
                else if(draw) {
                    res = 0;
                }
                else {
                    res = 1;
                }
            }
        }
        dp[i][j][k] = res;
        return res;
    }
  ```

### 2022-1-5
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/replace-all-s-to-avoid-consecutive-repeating-characters/)
  **分析：虽然自己写了出来（废话），但是看了题解之后，发现自己思维还是不够完整，题解的判断方式很简洁，还是得多学习啊；**
  ```
  public String modifyString(String s) {
        int n = s.length();
        char[] arr = s.toCharArray();
        for (int i = 0; i < n; ++i) {
            if (arr[i] == '?') {
                for (char ch = 'a'; ch <= 'c'; ++ch) {
                    if ((i > 0 && arr[i - 1] == ch) || (i < n - 1 && arr[i + 1] == ch)) {
                        continue;
                    }
                    arr[i] = ch;
                    break;
                }
            }
        }
        return new String(arr);
    }
  ```

### 2022-1-6
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/simplify-path/)
  **分析：简单字符串应用；先使用正则表达式split分块，然后分情况压栈即可；**
  ```
  public String simplifyPath(String path) {
        String[] strArray = path.split("/+");
        StringBuffer res = new StringBuffer("/");
        Deque<String> tool = new LinkedList<String>();
        for(int i = 0; i < strArray.length; ++i) {
            if(strArray[i].equals(".")) {
                continue;
            }
            else if(strArray[i].equals("..")) {
                if(!tool.isEmpty()) {
                    tool.pollLast();
                }
            }
            else if(!strArray[i].equals("")){
                tool.offerLast(strArray[i]);
            }
        }
        while(!tool.isEmpty()) {
            res.append(tool.pollFirst() + "/");
            if(tool.isEmpty()) {
                res.deleteCharAt(res.length() - 1);
            }
        }
        return res.toString();
    }
  ```

### 2022-1-7
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/maximum-nesting-depth-of-the-parentheses/)
  **分析：简单模拟，因为题目已经限定了是有效括号字符串，因此很简单；**
  ```
  public int maxDepth(String s) {
        int res = 0;
        int cur = 0;
        for(int i = 0; i < s.length(); ++i) {
            if(s.charAt(i) == '(') {
                cur++;
            }
            else if(s.charAt(i) == ')') {
                res = Math.max(res, cur);
                cur--;
            }
        }
        return res;
    }
  ```

### 2022-1-8
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/gray-code/)
  **分析：数学题吧......看题解的规律**
  ```
  public List<Integer> grayCode(int n) {
        List<Integer> res = new ArrayList<Integer>();
        res.add(0);
        for(int i = 1; i <= n; ++i) {
            int curSize = res.size();
            for(int j = curSize - 1; j >= 0; --j) {
                res.add(res.get(j) | (1 << (i - 1)));
            }
        }
        return res;
    }
  ```

### 2022-1-9
* 每日一题
  **分析：简单模拟题**
  ```
  public char slowestKey(int[] releaseTimes, String keysPressed) {
        char resChar = keysPressed.charAt(0);
        int resVal = releaseTimes[0];
        for(int i = 1; i < releaseTimes.length; ++i) {
            int curTime = releaseTimes[i] - releaseTimes[i - 1];
            if(curTime > resVal) {
                resVal = curTime;
                resChar = keysPressed.charAt(i);
            }
            else if(curTime == resVal) {
                if(keysPressed.charAt(i) > resChar) {
                    resChar = keysPressed.charAt(i);
                }
            }
        }
        return resChar;
    }
  ```

### 2021-1-10
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/additive-number/submissions/)
  **分析：暴力搜索dfs + 回溯；所谓回溯就是在下一层dfs判断为false的时候返回这一层，把当前层的数据从存储结构中拿出**
  ```
  List<List<Integer>> list;
    int n;
    public boolean isAdditiveNumber(String num) {
        n = num.length();
        list = new ArrayList<List<Integer>>();
        return dfs(num, 0);
    }

    boolean dfs(String num, int begin) {
        int curSize = list.size();
        if(begin == n) {
            return curSize >= 3;
        }
        int end = num.charAt(begin) == '0' ? begin + 1 : n;
        List<Integer> curValue = new ArrayList<Integer>();
        for(int i = begin; i < end; ++i) {
            curValue.add(0, num.charAt(i) - '0');
            if(curSize < 2 || check(list.get(curSize - 1), list.get(curSize - 2), curValue)) {
                list.add(curValue);
                if(dfs(num, i + 1)) {
                    return true;
                }
                list.remove(curSize);
            }
        }
        return false;
    }
    
    boolean check(List<Integer> list1, List<Integer> list2, List<Integer> list3) {
        int tmp = 0;
        List<Integer> res = new ArrayList<Integer>();
        for(int i = 0; i < list1.size() || i < list2.size(); ++i) {
            int curVal = 0;
            if(i < list1.size()) {
                curVal += list1.get(i);
            }
            if(i < list2.size()) {
                curVal += list2.get(i);
            }
            curVal += tmp;
            res.add(curVal % 10);
            tmp = curVal / 10;
        }
        if(tmp != 0) {
            res.add(tmp);
        }
        if(res.size() != list3.size()) {
            return false;
        }
        for(int i = 0; i < res.size(); ++i) {
            if(res.get(i) != list3.get(i)) {
                return false;
            }
        }
        return true;
    }
  ```

* 快速排序
  [题目链接](https://leetcode-cn.com/problems/sort-an-array/)
  **分析：递归快速排序思想（如果想要写其他排序，也可以通过这道题目）**
  ```
  int[] num;
    public int[] sortArray(int[] nums) {
        num = nums;
        qSort(0, num.length - 1);
        return num;
    }
    void qSort(int l, int r) {
        if(l > r) {
            return ;
        }
        int i = l, j = r;
        int k = l;
        while(i < j) {
            while(i < j && num[j] >= num[k]) {
                j--;
            }
            while(i < j && num[i] <= num[k]) {
                i++;
            }
            swap(i, j);
        }
        swap(i, k);
        qSort(l, i - 1);
        qSort(i + 1, r);
    }

    void swap(int i, int j) {
        int tmp = num[i];
        num[i] = num[j];
        num[j] = tmp;
    }
  ```

### 2022-1-11
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/escape-a-large-maze/)
  **分析：暴力搜索基础上进行数据判断；自己的代码有问题，看的题解**
  ```
  int EDGE = (int)1e6, MAX = (int)1e5;
    long BASE = 131L;
    Set<Long> set = new HashSet<>();
    int[][] dir = new int[][]{{1,0},{-1,0},{0,1},{0,-1}};
    public boolean isEscapePossible(int[][] blocked, int[] s, int[] t) {
        for (int[] p : blocked) set.add(p[0] * BASE + p[1]);
        int n = blocked.length;
        MAX = n * (n - 1) / 2; // 可直接使用 1e5
        return check(s, t) && check(t, s);
    }
    boolean check(int[] a, int[] b) {
        Set<Long> vis = new HashSet<>();
        Deque<int[]> d = new ArrayDeque<>();
        d.addLast(a);
        vis.add(a[0] * BASE + a[1]);
        while (!d.isEmpty() && vis.size() <= MAX) {
            int[] poll = d.pollFirst();
            int x = poll[0], y = poll[1];
            if (x == b[0] && y == b[1]) return true;
            for (int[] di : dir) {
                int nx = x + di[0], ny = y + di[1];
                if (nx < 0 || nx >= EDGE || ny < 0 || ny >= EDGE) continue;
                long hash = nx * BASE + ny;
                if (set.contains(hash)) continue;
                if (vis.contains(hash)) continue;
                d.addLast(new int[]{nx, ny});
                vis.add(hash);
            }
        }
        return vis.size() > MAX;
    }
  ```

### 2022-1-12
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/increasing-triplet-subsequence/)
  **分析：双指针，之前写过，但忘记了；递增的三元子序列，这个逻辑应该是没有问题的；**
  ```
  public boolean increasingTriplet(int[] nums) {
        int l = Integer.MAX_VALUE, m = Integer.MAX_VALUE;
        if(nums.length < 3) {
            return false;
        }

        for(int tmp : nums) {
            if(tmp <= l) {
                l = tmp;
            }
            else if(tmp <= m) {
                m = tmp;
            }
            else {
                return true;
            }
        }
        return false;
    }
  ```

### 2022-1-13
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/largest-number-at-least-twice-of-others/submissions/)
  **分析：简单模拟题，看了题解之后，自己还是不够简单；**
  ```
  public int dominantIndex(int[] nums) {
        if(nums.length == 1) {
            return 0;
        }
        int res = -1, max = Integer.MIN_VALUE;
        int k = nums[0];
        for(int i = 0; i < nums.length; ++i) {
            int tmp = max;
            if(nums[i] > max) {
                max = nums[i];
                res = i;
            }
            if(max != tmp) {
                k = tmp;
            }
            else if(nums[i] > k) {
                k = nums[i];
            }
        }
        if(max >= k * 2) {
            return res;
        }
        else {
            return -1;
        }
    }
  ```

### 2022-1-14
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/find-k-pairs-with-smallest-sums/)
  **分析：自己使用优先队列+个数优化的方式进行；大佬使用多路归并；**
  ```
  public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        PriorityQueue<int[]> queue = new PriorityQueue<int[]>((a, b) -> (b[0] + b[1] - a[0] - a[1]));
        for(int i = 0; i < nums1.length && i < k; ++i) {
            for(int j = 0; j < nums2.length && j < k; ++j) {
                queue.offer(new int[]{nums1[i], nums2[j]});
                if(queue.size() > k) {
                    queue.poll();
                }
            }
        }
        List<List<Integer>> res = new ArrayList<List<Integer>>();
        while(!queue.isEmpty()) {
            int[] tmp = queue.poll();
            List<Integer> mem = new ArrayList<Integer>();
            mem.add(tmp[0]);
            mem.add(tmp[1]);
            res.add(mem);
        }
        return res;
    }
  ```
  ```
  boolean flag = true;
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        List<List<Integer>> ans = new ArrayList<>();
        int n = nums1.length, m = nums2.length;
        if (n > m && !(flag = false)) return kSmallestPairs(nums2, nums1, k);
        PriorityQueue<int[]> q = new PriorityQueue<>((a,b)->(nums1[a[0]]+nums2[a[1]])-(nums1[b[0]]+nums2[b[1]]));
        for (int i = 0; i < Math.min(n, k); i++) q.add(new int[]{i, 0});
        while (ans.size() < k && !q.isEmpty()) {
            int[] poll = q.poll();
            int a = poll[0], b = poll[1];
            ans.add(new ArrayList<>(){{
                add(flag ? nums1[a] : nums2[b]);
                add(flag ? nums2[b] : nums1[a]);
            }});
            if (b + 1 < m) q.add(new int[]{a, b + 1});
        }
        return ans;
    }
  ```

### 2022-1-15
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/calculate-money-in-leetcode-bank/submissions/)
  **分析：简单模拟**
  ```
  public int totalMoney(int n) {
        int x = n / 7;
        int y = n % 7;
        int a1 = 7 * (1 + 7) / 2, an = 7 * (x + x + 6) / 2;
        int s1 = x * (a1 + an) / 2;
        int s2 = y * (x + 1 + x + y) / 2;
        return s1 + s2;
    }
  ```

### 2022-1-16
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/linked-list-random-node/submissions/)
  **分析：使用池塘抽样法，解决随机流问题；**
  ```
  static ListNode head;
    static Random r;
    public Solution(ListNode head) {
        this.head = head;
        r = new Random();
    }
    
    public int getRandom() {
        int res = 0, idx = 1;
        ListNode curNode = this.head;
        while(curNode != null) {
            if(r.nextInt(idx) == 0) {
                res = curNode.val;
            }
            curNode = curNode.next;
            idx++;
        }
        return res;
    }
  ```

* 剑指offer
  [题目链接](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)
  **分析：滑动窗口+单调递减双端队列；主要是窗口初始化；**
  ```
  public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        if(n == 0) {
            return new int[]{};
        }
        Deque<Integer> queue = new LinkedList<Integer>();
        int l = 0, r = 0;
        int[] res = new int[n - k + 1];
        int idx = 0;
        while(r < n) {
            while(!queue.isEmpty() && queue.peekLast() < nums[r]) {
                queue.pollLast();
            }
            queue.addLast(nums[r]);
            if(r - l + 1 == k) {
                res[idx++] = queue.peekFirst();
                if(queue.peekFirst() == nums[l]) {
                    queue.pollFirst();
                }
                l++;
            }
            r++;
        }
        return res;
    }
  ```

* top100
  [题目链接](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)
  **分析：我用的是两个堆；题解使用二分；**
  ```
  private PriorityQueue<Integer> max;
    private PriorityQueue<Integer> min;
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        max = new PriorityQueue<Integer>((a, b) -> (b - a));
        min = new PriorityQueue<Integer>();
        int l1 = nums1.length;
        int l2 = nums2.length;
        for(int i = 0; i < l1 || i < l2; ++i) {
            if(i < l1) {
                insert(nums1[i]);
            }
            if(i < l2) {
                insert(nums2[i]);
            }
            if(max.size() - min.size() > 1) {
                min.offer(max.poll());
            }
            else if(min.size() - max.size() > 1) {
                max.offer(min.poll());
            }
        }
        if((l1 + l2) % 2 == 0) {
            double res = min.peek() + max.peek();
            return res / 2;
        }
        else {
            return max.size() > min.size() ? max.peek() : min.peek();
        }
    }

    void insert(int l) {
        if(max.isEmpty()) {
            max.offer(l);
        }
        else if(min.isEmpty()) {
            if(l > max.peek()) {
                min.offer(l);
            }
            else {
                max.offer(l);
            }
        }
        else {
            if(l < max.peek()) {
                max.offer(l);
            }
            else {
                min.offer(l);
            }
        }
    }
  ```

### 2022-1-17
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/count-vowels-permutation/submissions/)
  **分析：dp，自己想到了dp，但是还是直接看了大佬题解；**
  ```
  static int mod = (int)1e9 + 7;
    public int countVowelPermutation(int n) {
        long[][] f = new long[n][5];
        Arrays.fill(f[0], 1);
        for(int i = 0; i < n - 1; ++i) {
            f[i + 1][0] += f[i][1];
            f[i + 1][0] += f[i][4];
            f[i + 1][0] += f[i][2];
            f[i + 1][1] += f[i][0];
            f[i + 1][1] += f[i][2];
            f[i + 1][2] += f[i][1];
            f[i + 1][2] += f[i][3];
            f[i + 1][3] += f[i][2];
            f[i + 1][4] += f[i][3];
            f[i + 1][4] += f[i][2];
            for (int j = 0; j < 5; ++j) {
                f[i + 1][j] %= mod;
            }
        }
        long res = 0;
        for(int i = 0; i < 5; ++i) {
            res += f[n - 1][i];
        }
        return (int)(res % mod);
    }
  ```

### 2022-1-18
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/minimum-time-difference/)
  **分析：模拟；需要注意，这道题目自己之前看的题解；根据鸽巢原理，一共有24 * 60种不同的时间；**
  ```
  public int findMinDifference(List<String> timePoints) {
        List<Integer> nums = new ArrayList<Integer>();
        for (String t: timePoints) {
            int h = Integer.parseInt(t.substring(0, 2));
            int m = Integer.parseInt(t.substring(3, 5));
            int x = h * 60 + m;
            nums.add(x);
            nums.add(x + 1440);
        }
        Collections.sort(nums);
        int n = nums.size();
        int res = Integer.MAX_VALUE;
        for (int i = 1; i < n; i ++) {
            int x = nums.get(i - 1);
            int y = nums.get(i);
            res = Math.min(res, y - x);
        }
        return res;
    }
  ```

### 2022-1-19
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/contains-duplicate-ii/)
  **分析：移动窗口 + 判断**
  ```
  public boolean containsNearbyDuplicate(int[] nums, int k) {
        int n = nums.length;
        Set<Integer> set = new HashSet<Integer>();
        for(int i = 0; i < n; ++i) {
            if(i > k) {
                set.remove(nums[i - k - 1]);
            }
            if(set.contains(nums[i])) {
                return true;
            }
            set.add(nums[i]);
        }
        return false;
    }
  ```

### 2022-1-20
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/stone-game-ix/submissions/)
  **分析：抄的题解**
  ```
  public boolean stoneGameIX(int[] stones) {
        int cnt0 = 0, cnt1 = 0, cnt2 = 0;
        for (int val : stones) {
            int type = val % 3;
            if (type == 0) {
                ++cnt0;
            } else if (type == 1) {
                ++cnt1;
            } else {
                ++cnt2;
            }
        }
        if (cnt0 % 2 == 0) {
            return cnt1 >= 1 && cnt2 >= 1;
        }
        return cnt1 - cnt2 > 2 || cnt2 - cnt1 > 2;
    }
  ```

### 2022-1-21
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/jump-game-iv/)
  **分析：广度优先搜索；先把所有值相同的index处理成一个集合，然后再套用模板即可；需要注意，为了时间考虑，用Set作为已访问index的存储结构；**
  ```
  public int minJumps(int[] arr) {
        int n = arr.length;

        Map<Integer, List<Integer>> map = new HashMap<Integer, List<Integer>>();
        for(int i = 0; i < n; ++i) {
            map.putIfAbsent(arr[i], new ArrayList<Integer>());
            map.get(arr[i]).add(i);
        }

        Queue<int[]> queue = new LinkedList<int[]>();
        Set<Integer> visitedIndex = new HashSet<Integer>();
        
        queue.offer(new int[]{0, 0});
        visitedIndex.add(0);
        while(!queue.isEmpty()) {
            int[] cur = queue.poll();
            if(cur[0] == n - 1) {
                return cur[1];
            }
            if(map.containsKey(arr[cur[0]])) {
                for(int i : map.get(arr[cur[0]])) {
                    if(!visitedIndex.contains(i)) {
                        queue.offer(new int[]{i, cur[1] + 1});
                        visitedIndex.add(i);
                    }
                }
                map.remove(arr[cur[0]]);
            }

            if(cur[0] - 1 >= 0 && !visitedIndex.contains(cur[0] - 1)) {
                queue.offer(new int[]{cur[0] - 1, cur[1] + 1});
                visitedIndex.add(cur[0] - 1);
            }
            if(cur[0] + 1 < n && !visitedIndex.contains(cur[0] + 1)) {
                queue.offer(new int[]{cur[0] + 1, cur[1] + 1});
                visitedIndex.add(cur[0] + 1);
            }
        }
        return -1;
    }
  ```

### 2022-1-23
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/stock-price-fluctuation/submissions/)
  **分析：map存储时间戳和价格的映射，treemap存储价格和对应价格个数的映射；**
  ```
  int cur;
    Map<Integer, Integer> map;
    TreeMap<Integer, Integer> tr;
    public StockPrice() {
        cur = 0;
        map = new HashMap<Integer, Integer>();
        tr = new TreeMap<Integer, Integer>();
    }
    
    public void update(int timestamp, int price) {
        cur = Math.max(cur, timestamp);
        if(map.containsKey(timestamp)) {
            int oldPri = map.get(timestamp);
            int timeSu = tr.get(oldPri);
            if(timeSu == 1) {
                tr.remove(oldPri);
            }
            else {
                tr.put(oldPri, timeSu - 1);
            }
        }
        map.put(timestamp, price);
        tr.put(price, tr.getOrDefault(price, 0) + 1);
    }
    
    public int current() {
        return map.get(cur);
    }
    
    public int maximum() {
        return tr.lastKey();
    }
    
    public int minimum() {
        return tr.firstKey();
    }
  ```

### 2022-1-24
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/second-minimum-time-to-reach-destination/)
  **分析：抄的题解**
  ```
  static int N = 10010, M = 4 * N, INF = 0x3f3f3f3f, idx = 0;
    static int[] he = new int[N], e = new int[M], ne = new int[M];
    static int[] dist1 = new int[N], dist2 = new int[N];
    void add(int a, int b) {
        e[idx] = b;
        ne[idx] = he[a];
        he[a] = idx;
        idx++;
    }
    public int secondMinimum(int n, int[][] edges, int time, int change) {
        Arrays.fill(dist1, INF);
        Arrays.fill(dist2, INF);
        Arrays.fill(he, -1);
        idx = 0;
        for (int[] e : edges) {
            int u = e[0], v = e[1];
            add(u, v); add(v, u);
        }
        PriorityQueue<int[]> q = new PriorityQueue<>((a,b)->a[1]-b[1]);
        q.add(new int[]{1, 0});
        dist1[1] = 0;
        while (!q.isEmpty()) {
            int[] poll = q.poll();
            int u = poll[0], step = poll[1];
            for (int i = he[u]; i != -1; i = ne[i]) {
                int j = e[i];
                int a = step / change, b = step % change;
                int wait = a % 2 == 0 ? 0 : change - b;
                int dist = step + time + wait;
                if (dist1[j] > dist) {
                    dist2[j] = dist1[j];
                    dist1[j] = dist;
                    q.add(new int[]{j, dist1[j]});
                    q.add(new int[]{j, dist2[j]});
                } else if (dist1[j] < dist && dist < dist2[j]) {
                    dist2[j] = dist;
                    q.add(new int[]{j, dist2[j]});
                }
            }
        }
        return dist2[n];
    }
  ```

### 2022-1-26
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/detect-squares/submissions/)
  **分析：**
  ```
  Map<Integer, Map<Integer, Integer>> row2Col = new HashMap<>();
    
    public void add(int[] point) {
        int x = point[0], y = point[1];
        Map<Integer, Integer> col2Cnt = row2Col.getOrDefault(x, new HashMap<>());
        col2Cnt.put(y, col2Cnt.getOrDefault(y, 0) + 1);
        row2Col.put(x, col2Cnt);
    }
    
    public int count(int[] point) {
        int x = point[0], y = point[1];
        int ans = 0;
        Map<Integer, Integer> col2Cnt = row2Col.getOrDefault(x, new HashMap<>());
        for (int ny : col2Cnt.keySet()) {
            if (ny == y) continue;
            int c1 = col2Cnt.get(ny);
            int len = y - ny;
            int[] nums = new int[]{x + len, x - len};
            for (int nx : nums) {
                Map<Integer, Integer> temp = row2Col.getOrDefault(nx, new HashMap<>());
                int c2 = temp.getOrDefault(y, 0), c3 = temp.getOrDefault(ny, 0);
                ans += c1 * c2 * c3;
            }
        }
        return ans;
    }
  ```

### 2022-1-27
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/number-of-valid-words-in-a-sentence/)
  **分析：简单模拟**
  ```
  public int countValidWords(String sentence) {
        int n = sentence.length();
        int l = 0, r = 0;
        int ret = 0;
        while (true) {
            while (l < n && sentence.charAt(l) == ' ') {
                l++;
            }
            if (l >= n) {
                break;
            }
            r = l + 1;
            while (r < n && sentence.charAt(r) != ' ') {
                r++;
            }
            if (isValid(sentence.substring(l, r))) { // 判断根据空格分解出来的 token 是否有效
                ret++;
            }
            l = r + 1;
        }
        return ret;
    }

    public boolean isValid(String word) {
        int n = word.length();
        boolean hasHyphens = false;
        for (int i = 0; i < n; i++) {
            if (Character.isDigit(word.charAt(i))) {
                return false;
            } else if (word.charAt(i) == '-') {
                if (hasHyphens == true || i == 0 || i == n - 1 || !Character.isLetter(word.charAt(i - 1)) || !Character.isLetter(word.charAt(i + 1))) {
                    return false;
                }
                hasHyphens = true;
            } else if (word.charAt(i) == '!' || word.charAt(i) == '.' || word.charAt(i) == ',') {
                if (i != n - 1) {
                    return false;
                }
            }
        }
        return true;
    }
  ```

### 2021-1-28
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/the-number-of-weak-characters-in-the-game/)
  **题目分析：抄答案**
  ```
  public int numberOfWeakCharacters(int[][] properties) {
        Arrays.sort(properties, (o1, o2) -> {
            return o1[0] == o2[0] ? (o1[1] - o2[1]) : (o2[0] - o1[0]);
        });
        int maxDef = 0;
        int ans = 0;
        for (int[] p : properties) {
            if (p[1] < maxDef) {
                ans++;
            } else {
                maxDef = p[1];
            }
        }
        return ans;
    }
  ```

### 2022-1-29
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/map-of-highest-peak/submissions/)
  **分析：抄的爆搜**
  ```
  int[][] dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

    public int[][] highestPeak(int[][] isWater) {
        int m = isWater.length, n = isWater[0].length;
        int[][] ans = new int[m][n];
        for (int i = 0; i < m; ++i) {
            Arrays.fill(ans[i], -1); // -1 表示该格子尚未被访问过
        }
        Queue<int[]> queue = new ArrayDeque<int[]>();
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (isWater[i][j] == 1) {
                    ans[i][j] = 0;
                    queue.offer(new int[]{i, j}); // 将所有水域入队
                }
            }
        }
        while (!queue.isEmpty()) {
            int[] p = queue.poll();
            for (int[] dir : dirs) {
                int x = p[0] + dir[0], y = p[1] + dir[1];
                if (0 <= x && x < m && 0 <= y && y < n && ans[x][y] == -1) {
                    ans[x][y] = ans[p[0]][p[1]] + 1;
                    queue.offer(new int[]{x, y});
                }
            }
        }
        return ans;
    }
  ```

### 2022-2-3
* 每日一题
  [题目链接](https://leetcode-cn.com/problems/find-the-minimum-number-of-fibonacci-numbers-whose-sum-is-k/)
  **分析：贪心选取每次小于k的值，减去后继续选取；**
  ```
  public int findMinFibonacciNumbers(int k) {
        List<Integer> fib = new ArrayList<Integer>();
        int a = 1, b = 1;
        fib.add(1);
        while(a + b <= k) {
            int c = a + b;
            fib.add(c);
            a = b;
            b = c;
        }
        int res = 0;
        for(int i = fib.size() - 1; i >= 0 && k > 0; --i) {
            if(k >= fib.get(i)) {
                k -= fib.get(i);
                res++;
            }
        }
        return res;
    }
  ```

### 2022-2-5
* 每日刷题
  [题目链接](https://leetcode-cn.com/problems/path-with-maximum-gold/)
  **分析：搜索+回溯**
  ```
  int[][] to = new int[][]{{0, 1}, {0, -1}, {-1, 0}, {1, 0}};
    int[][] grid;
    int res;
    int n;
    int m;
    public int getMaximumGold(int[][] grid) {
        this.n = grid.length;
        this.m = grid[0].length;
        this.grid = grid;
        this.res = 0;
        for(int i = 0; i < n; ++i) {
            for(int j = 0; j < m; ++j) {
                if(grid[i][j] != 0) {
                    dfs(i, j, 0);
                }
            }
        }
        return res;
    }

    void dfs(int x, int y, int cur) {
        int tmp = grid[x][y];
        cur += tmp;
        res = Math.max(res, cur);
        grid[x][y] = 0;
        for(int i = 0; i < to.length; ++i) {
            int xn = x + to[i][0];
            int yn = y + to[i][1];
            if(xn >= 0 && xn < n && yn >= 0 && yn < m && grid[xn][yn] > 0) {
                dfs(xn, yn, cur);
            }
        }
        grid[x][y] = tmp;
    }
  ```