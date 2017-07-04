---
title: 'SwordPointToOffer(40): 数组中只出现一次的数字'
date: 2017-02-07 14:28:32
categories:
- SwordPointToOffer
---

# 题目
一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字，要求时间复杂度是$O(n)$，空间复杂度是$O(1)$。

例如：输入数组 {2,4,3,6,3,2,5,5}，会输出4和6。

# 分析
这两个题目都在强调一个（或两个）数字只出现一次，其他的出现两次。这有什么意义呢？我们想到**异或运算**的一个性质：任何一个数字异或它自己都等于 0。也就是说， 如果我们从头到尾依次异或数组中的每一个数字，那么最终的结果刚好是那个只出现一次的数字，因为那些成对出现两次的数字全部在异或中抵消了。

想明白怎么解决这个简单问题之后，我们再回到原始的问题，看看能不能运用相同的思路。我们试着把原数组分成两个子数组，使得每个子数组包含一个只出现一次的数字，而其他数字都成对出现两次。如果能够这样拆分成两个数组， 我们就可以按照前面的办法分别找出两个只出现一次的数字了。

我们还是从头到尾依次异或数组中的每一个数字，那么最终得到的结果就是两个只出现一次的数字的异或结果。因为其他数字都出现了两次，在异或中全部抵消了。由于这两个数字肯定不一样，那么异或的结果肯定不为 0，也就是说在这个结果数字的二进制表示中至少就有一位为 1 。我们在结果数字中找到第一个为 1 的位的位置，记为第 n 位。现在**我们以第 n 位是不是 １ 为标准把原数组中的数字分成两个子数组，第一个子数组中每个数字的第 n 位都是 1，而第二个子数组中每个数字的第 n 位都是 0。**由于我们分组的标准是数字中的某一位是 1 还是 0 ， 那么出现了两次的数字肯定被分配到同一个子数组。因为两个相同的数字的任意一位都是相同的，我们不可能把两个相同的数字分配到两个子数组中去，**于是我们已经把原数组分成了两个子数组，每个子数组都包含一个只出现一次的数字**，而其他数字都出现了两次。我们已经知道如何在数组中找出唯一一个只出现一次数字， 因此到此为止所有的问题都已经解决了。

# 实现
```java
public class NumbersAppearOnce {

    public static void main(String[] args) {
        int[] data1 = new int[]{0, 2, 1, 2, 1, 3};
        int[] data2 = new int[]{3, 2, 1, 2, 1};

        NumbersAppearOnce test = new NumbersAppearOnce();
        test.handle(data2);
    }

    public void handle(int[] data) {
        if (data == null || data.length <= 0)
            return;

        int resultExclusiveOR = 0;
        for (int i : data)
            resultExclusiveOR ^= i;

        int indexBitOf1 = findFirstBitIs1(resultExclusiveOR);
        // array中不存在两个只出现一次的数或者只存在一个只出现一次的数字0
        if (indexBitOf1 < 0)
            throw new RuntimeException("There is not two num in array difference!");

        int n1 = 0;
        int n2 = 0;
        for (int i : data) {
            if (isBit1(i, indexBitOf1)) {
                n1 ^= i;
            } else {
                n2 ^= i;
            }
        }
        
        // 例如数组中只有一个数字出现一次，且这个数字是非0的。比如{3, 2, 1, 2, 1}，会输出3,0
        if (!checkResult(data, n1, n2))
            throw new RuntimeException("There is not two num in array difference!");
        
        System.out.println("n1=" + n1);
        System.out.println("n2=" + n2);
    }

    /**
     * 在整数二进制表示中找到最右边是1的位
     *
     * @param num
     * @return -1 表示没有bit=1
     */
    private int findFirstBitIs1(int num) {
        int indexBit = 0;
        while ((num & 1) == 0 && (indexBit < Integer.SIZE)) {
            num = num >> 1;
            ++indexBit;
        }

        if ((num & 1) == 0)
            indexBit = -1;

        return indexBit;
    }

    /**
     * 判断num的二进制表示中从右边数起的indexBit位是不是1
     * @param num
     * @param indexBit
     * @return
     */
    private boolean isBit1(int num, int indexBit) {
        num = num >> indexBit;
        return (num & 1) == 1;
    }

    private boolean checkResult(int[] data, int n1, int n2) {
        boolean b1 = false;
        boolean b2 = false;
        for (int i: data){
            if (i == n1)
                b1 = true;
            if (i == n2)
                b2 = true;
        }
        return b1 && b2;
    }
}
```