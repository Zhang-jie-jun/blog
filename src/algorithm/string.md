# 字符串算法

## 字符串算法简介
### 什么是字符串算法？
字符串算法是处理字符串数据的算法，包括字符串匹配、编辑距离、模式识别等。字符串算法广泛应用于文本处理、搜索引擎、生物信息学等领域。

### 常见字符串算法分类
- **字符串匹配**：KMP、Boyer-Moore、Rabin-Karp
- **字符串编辑**：编辑距离、最长公共子序列
- **字符串处理**：反转、替换、分割

---

## 1. KMP 算法

### 原理
通过预处理模式串构建部分匹配表（LPS数组），避免回溯。

### 实现
```go
func KMP(text, pattern string) []int {
    if len(pattern) == 0 || len(text) < len(pattern) {
        return []int{}
    }
    
    lps := buildLPS(pattern)
    result := []int{}
    
    i, j := 0, 0
    for i < len(text) {
        if text[i] == pattern[j] {
            i++
            j++
        }
        
        if j == len(pattern) {
            result = append(result, i-j)
            j = lps[j-1]
        } else if i < len(text) && text[i] != pattern[j] {
            if j != 0 {
                j = lps[j-1]
            } else {
                i++
            }
        }
    }
    
    return result
}

func buildLPS(pattern string) []int {
    lps := make([]int, len(pattern))
    length := 0
    
    for i := 1; i < len(pattern); {
        if pattern[i] == pattern[length] {
            length++
            lps[i] = length
            i++
        } else {
            if length != 0 {
                length = lps[length-1]
            } else {
                lps[i] = 0
                i++
            }
        }
    }
    
    return lps
}
```

### 复杂度
- **时间复杂度**：O(n + m)
- **空间复杂度**：O(m)

---

## 2. Boyer-Moore 算法

### 原理
从右向左比较，利用坏字符规则和好后缀规则跳过不必要的比较。

### 实现
```go
func BoyerMoore(text, pattern string) []int {
    if len(pattern) == 0 || len(text) < len(pattern) {
        return []int{}
    }
    
    badChar := buildBadCharTable(pattern)
    result := []int{}
    
    i := 0
    for i <= len(text)-len(pattern) {
        j := len(pattern) - 1
        
        for j >= 0 && text[i+j] == pattern[j] {
            j--
        }
        
        if j < 0 {
            result = append(result, i)
            i++
        } else {
            shift := j - badChar[text[i+j]]
            if shift < 1 {
                shift = 1
            }
            i += shift
        }
    }
    
    return result
}

func buildBadCharTable(pattern string) map[byte]int {
    table := make(map[byte]int)
    
    for i := 0; i < len(pattern); i++ {
        table[pattern[i]] = i
    }
    
    return table
}
```

### 复杂度
- **时间复杂度**：O(n/m)（平均），O(n*m)（最坏）
- **空间复杂度**：O(k)（k为字符集大小）

---

## 3. Rabin-Karp 算法

### 原理
利用哈希函数将字符串比较转化为整数比较。

### 实现
```go
func RabinKarp(text, pattern string) []int {
    if len(pattern) == 0 || len(text) < len(pattern) {
        return []int{}
    }
    
    base := 26
    mod := 1000000007
    
    result := []int{}
    
    // 计算模式串的哈希值
    patternHash := 0
    textHash := 0
    power := 1
    
    for i := 0; i < len(pattern); i++ {
        patternHash = (patternHash*base + int(pattern[i])) % mod
        textHash = (textHash*base + int(text[i])) % mod
        power = (power * base) % mod
    }
    
    for i := 0; i <= len(text)-len(pattern); i++ {
        if patternHash == textHash {
            if text[i:i+len(pattern)] == pattern {
                result = append(result, i)
            }
        }
        
        if i < len(text)-len(pattern) {
            textHash = (textHash*base - int(text[i])*power + int(text[i+len(pattern)])) % mod
            if textHash < 0 {
                textHash += mod
            }
        }
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(n + m)（平均），O(n*m)（最坏）
- **空间复杂度**：O(1)

---

## 4. 最长回文子串

### 原理
中心扩展法：以每个字符为中心向两边扩展。

### 实现
```go
func LongestPalindrome(s string) string {
    if len(s) == 0 {
        return ""
    }
    
    start, end := 0, 0
    
    for i := 0; i < len(s); i++ {
        len1 := expandAroundCenter(s, i, i)
        len2 := expandAroundCenter(s, i, i+1)
        
        maxLen := max(len1, len2)
        
        if maxLen > end-start {
            start = i - (maxLen-1)/2
            end = i + maxLen/2
        }
    }
    
    return s[start : end+1]
}

func expandAroundCenter(s string, left, right int) int {
    for left >= 0 && right < len(s) && s[left] == s[right] {
        left--
        right++
    }
    
    return right - left - 1
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 复杂度
- **时间复杂度**：O(n²)
- **空间复杂度**：O(1)

---

## 5. 字符串反转

### 实现
```go
func ReverseString(s []byte) {
    left, right := 0, len(s)-1
    
    for left < right {
        s[left], s[right] = s[right], s[left]
        left++
        right--
    }
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)

---

## 6. 最长公共前缀

### 原理
逐个字符比较所有字符串。

### 实现
```go
func LongestCommonPrefix(strs []string) string {
    if len(strs) == 0 {
        return ""
    }
    
    prefix := strs[0]
    
    for i := 1; i < len(strs); i++ {
        for len(strs[i]) < len(prefix) || strs[i][:len(prefix)] != prefix {
            prefix = prefix[:len(prefix)-1]
            if prefix == "" {
                return ""
            }
        }
    }
    
    return prefix
}
```

### 复杂度
- **时间复杂度**：O(n*m)
- **空间复杂度**：O(1)

---

## 7. 字符串转换整数 (atoi)

### 原理
处理符号、数字转换、溢出判断。

### 实现
```go
func MyAtoi(s string) int {
    if len(s) == 0 {
        return 0
    }
    
    sign := 1
    result := 0
    i := 0
    
    // 跳过空格
    for i < len(s) && s[i] == ' ' {
        i++
    }
    
    // 处理符号
    if i < len(s) && (s[i] == '+' || s[i] == '-') {
        if s[i] == '-' {
            sign = -1
        }
        i++
    }
    
    // 转换数字
    for i < len(s) && s[i] >= '0' && s[i] <= '9' {
        digit := int(s[i] - '0')
        
        // 溢出检查
        if result > (int(^uint(0)>>1)-digit)/10 {
            if sign == 1 {
                return int(^uint(0) >> 1)
            }
            return int(^uint(0) >> 1) * -1 - 1
        }
        
        result = result*10 + digit
        i++
    }
    
    return result * sign
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)

---

## 字符串算法对比

| 算法 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|----------|-----------|----------|
| KMP | O(n+m) | O(m) | 单模式串匹配 |
| Boyer-Moore | O(n/m) | O(k) | 长文本搜索 |
| Rabin-Karp | O(n+m) | O(1) | 多模式串匹配 |
| 最长回文 | O(n²) | O(1) | 回文检测 |
| 字符串反转 | O(n) | O(1) | 简单反转 |
| 最长公共前缀 | O(nm) | O(1) | 前缀匹配 |
| atoi | O(n) | O(1) | 字符串转整数 |

## 常见面试题

### 1. KMP 算法中的 LPS 数组是什么？

**答案：**
LPS（Longest Prefix Suffix）数组存储模式串中每个位置的最长相等前后缀长度，用于跳过不必要的比较。

### 2. Boyer-Moore 算法为什么高效？

**答案：**
Boyer-Moore 算法从右向左比较，利用坏字符规则和好后缀规则可以跳过多个字符，平均情况下比其他算法更快。

### 3. Rabin-Karp 算法的哈希冲突如何处理？

**答案：**
当哈希值相同时，需要进行字符串比较来确认是否真正匹配，避免哈希冲突导致的错误。

### 4. 如何判断字符串是否为回文？

**答案：**
使用双指针法，从两端向中间比较字符是否相等。

### 5. 如何处理字符串转整数的溢出？

**答案：**
在每次计算前检查是否会溢出，使用 INT_MAX 和 INT_MIN 作为边界。

<center>...未完待续...</center>
---  
***  

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<div id="gitalk-container"></div>
<script>
  var gitalk = new Gitalk({
    "clientID": "44d7c96f948be236a8c9",
    "clientSecret": "fb9fb3178db6640131c4e3eb69f9449e42bba661",
    "repo": "blog",
    "owner": "Zhang-jie-jun",
    "admin": ["Zhang-jie-jun"],
    "id": location.pathname,      
    "distractionFreeMode": false  
  });
  gitalk.render("gitalk-container");
</script>
