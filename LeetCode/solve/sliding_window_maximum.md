# **LeetCode 239: Sliding Window Maximum (滑動視窗最大值)**

這份文件旨在詳細分析 LeetCode 第 239 題，從題目理解到最佳化解法的思路推導。

## **1\. 題目敘述 (Problem Description)**

題目：  
給你一個整數陣列 nums，有一個大小為 k 的滑動視窗（Sliding Window）從陣列的最左側移動到最右側。你只看得到在視窗內的 k 個數字。滑動視窗每次只向右移動一位。  
要求：  
返回每一個視窗位置的最大值。  
**範例 (Example)：**

輸入: nums \= \[1,3,-1,-3,5,3,6,7\], k \= 3  
輸出: \[3,3,5,5,6,7\]

解釋:  
視窗位置                最大值  
\---------------               \-----  
\[1  3  \-1\] \-3  5  3  6  7       3  
 1 \[3  \-1  \-3\] 5  3  6  7       3  
 1  3 \[-1  \-3  5\] 3  6  7       5  
 1  3  \-1 \[-3  5  3\] 6  7       5  
 1  3  \-1  \-3 \[5  3  6\] 7       6  
 1  3  \-1  \-3  5 \[3  6  7\]      7

**限制條件 (Constraints)：**

* $1 \\le k \\le nums.length \\le 10^5$  
* $-10^4 \\le nums\[i\] \\le 10^4$  
* 這意味著時間複雜度為 $O(N^2)$ 或 $O(NK)$ 的解法通常會超時 (Time Limit Exceeded)，我們需要尋找 $O(N)$ 或 $O(N \\log K)$ 的解法。

## **2\. 測驗目的 (Test Purpose)**

這道題目主要測試面試者以下能力：

1. **資料結構的選擇與應用：** 是否能跳出暴力的巢狀迴圈，選用適合維護「區間最值」的資料結構（如 Deque, Heap, Segment Tree）。  
2. **單調佇列 (Monotonic Queue) 的理解：** 這題是單調佇列的經典教科書級題目。  
   * **概念補充：** 單調佇列是一種特殊的佇列，其內部的元素始終保持「單調遞增」或「單調遞減」的順序。  
   * **在本題的作用：** 我們維護一個「單調遞減」的佇列，這保證了隊頭（Front）始終是當前視窗內的最大值。  
   * **核心價值：** 它解決了傳統佇列無法快速獲取最值的問題，同時利用「先進先出」的特性來管理視窗邊界，將查找最大值的時間複雜度從 $O(K)$ 降低到均攤 $O(1)$。  
   * **實作工具 (collections.deque)：** 在 Python 中，標準的 list 若要移除第一個元素 (pop(0))，時間複雜度為 $O(K)$，這會導致整體效率下降。因此我們使用 collections.deque (雙端佇列)，它支援在頭尾兩端都能以 $O(1)$ 的時間進行 append 和 pop 操作，完美契合單調佇列的需求。  
3. **邊界條件處理：** 視窗滑動時的元素進出順序（Index Handling）。  
4. **時間複雜度分析能力：** 理解為什麼線性時間 $O(N)$ 是可行的。

## **3\. 思路邏輯與推導 (Thought Process)**

### **第一階段：暴力解法 (Brute Force) \- 直覺但太慢**

最直觀的想法是每滑動一次，就遍歷視窗內的 $k$ 個元素找出最大值。

* **邏輯：** 針對每個視窗 nums\[i:i+k\] 執行 max()。  
* **缺點：** 總共有 $N-k+1$ 個視窗，每次找最大值花 $O(K)$。總時間複雜度為 $O(N \\cdot K)$。當 $N=10^5, k=N/2$ 時，運算量極大，必定超時。

### **第二階段：使用優先佇列 (Priority Queue / Max Heap)**

為了加速「找最大值」的過程，我們可以使用 Max Heap。

* **邏輯：** 維護一個 Heap，裡面存放 (-val, index) (存負值是因為 Python 預設是 Min Heap)。  
* **問題：** 當視窗往右滑，最左邊的元素滑出時，Heap 很難以 $O(1)$ 或 $O(\\log K)$ 的時間直接移除任意元素（通常只能移除 Top）。這會導致 Heap 裡累積很多已經不在視窗內的「過期」元素。  
* **修正：** 雖然不能直接刪，但我們可以採用「延遲刪除 (Lazy Removal)」策略——如果 Heap 的堆頂元素索引已經小於當前視窗的左邊界，就把它 pop 掉。  
* **複雜度：** $O(N \\log N)$。這是一個可行的解法，但還不是最佳。

### **第三階段：單調雙端佇列 (Monotonic Deque) \- 最佳解**

這是本題的核心思路。我們需要一個容器，既能快速知道最大值，又能快速處理滑動視窗的「先進先出」。

關鍵觀察 (Crucial Observation)：  
假設視窗中有兩個元素 nums\[i\] 和 nums\[j\]，且 i \< j (i 在 j 左邊)。  
如果 nums\[i\] \< nums\[j\]，那麼 nums\[i\] 永遠不可能 成為視窗的最大值了。  
為什麼？

1. 當 nums\[j\] 還在視窗內時，因為 nums\[j\] 比 nums\[i\] 大，最大值輪不到 nums\[i\]。  
2. 當 nums\[j\] 滑出視窗時，nums\[i\] 早就已經滑出去了（因為 i 在 j 左邊）。

結論：  
如果一個新進來的數字比前面的數字大，前面的數字就是「廢物」，可以直接丟掉。這暗示我們可以維護一個 數值遞減 的佇列。  
(補充) 關於單調性的維護機制與「單調遞增」的對比：  
你可能會好奇：「那如果要保持 單調遞增 (Monotonically Increasing) 呢？或是這些單調性到底是怎麼『保持』的？」  
維護單調性的核心邏輯在於 **「汰弱留強」**，但誰是「強」取決於你的目標是找最大值還是最小值：

* **本題目標：找最大值 (Max)** $\\rightarrow$ **維護單調遞減 (Decreasing)**  
  * **邏輯：** 我們只在乎「大」的數字。  
  * **操作：** 當新元素 nums\[i\] 進來時，如果不比隊尾的元素小（nums\[i\] \>= tail），代表隊尾那個較小的數字已經沒用了（因為 nums\[i\] 又大又新），所以把隊尾 **踢掉 (Pop)**。  
  * **結果：** 佇列內保留的都是相對較大的數，且由大到小排列。  
* **反向思考：找最小值 (Min)** $\\rightarrow$ **維護單調遞增 (Increasing)**  
  * **邏輯：** 如果題目變成「滑動視窗最小值」，我們就只在乎「小」的數字。  
  * **操作：** 當新元素 nums\[i\] 進來時，如果不比隊尾的元素大（nums\[i\] \<= tail），代表隊尾那個較大的數字已經沒用了（因為 nums\[i\] 又小又新），所以把隊尾 **踢掉 (Pop)**。  
  * **結果：** 佇列內保留的都是相對較小的數，且由小到大排列。

**操作步驟 (針對本題的單調遞減)：**

1. **保持單調性 (Push)：** 當新元素 nums\[i\] 進來時，從佇列尾端 (Back) 檢查，如果尾端的數字比 nums\[i\] 小，就把它移除（因為它不再有機會成為最大值）。重複此動作直到尾端數字大於 nums\[i\] 或佇列為空，然後將 nums\[i\] 的 **索引** 加入尾端。  
2. **移除過期元素 (Pop)：** 檢查佇列頭端 (Front) 的索引。如果該索引已經滑出視窗範圍（即 index \<= i \- k），則將其移除。  
3. **記錄答案 (Result)：** 佇列的頭端 (Front) 對應的數值永遠是當前視窗的最大值。從視窗形成後（i \>= k \- 1）開始記錄。

## **4\. 複雜度分析 (Complexity Analysis)**

* **時間複雜度 (Time Complexity):** $O(N)$  
  * 雖然程式碼中有 while 迴圈，但請注意：每個元素最多只會被 append 進 deque 一次，也最多只會被 pop 出來一次。  
  * 因此，所有操作的總次數是線性的，平均分攤到每個元素上是 $O(1)$。總體時間為 $O(N)$。  
* **空間複雜度 (Space Complexity):** $O(K)$  
  * 我們需要一個 deque 來存儲索引。  
  * 在最壞情況下（例如陣列是嚴格遞減的 \[5, 4, 3, 2, 1\]），deque 中會存儲視窗內的所有元素，因此空間最大為 $O(K)$。  
  * 輸出陣列佔用 $O(N \- K \+ 1)$，通常視為回傳值不計入演算法輔助空間，若計入則為 $O(N)$。

```
from collections import deque

class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        result = list()

        # ----- This monotonic queue only store the index ----- #
        Queue = deque()
        for i in range(len(nums)):

            # ----- Remove the out of range index ----- #
            if Queue:
                if Queue[0] < i-k+1:
                    Queue.popleft()

            # ----- Sustain the monotonicity by comparing with the last element ----- #
            while Queue and nums[Queue[-1]] < nums[i]:
                Queue.pop()
            
            # ----- Add the index into this queue ----- #
            Queue.append(i)

            # ----- First element formated after index > win_size - 1 ----- #
            if i >= k-1:
                result += [nums[Queue[0]]]
        
        return result
```
