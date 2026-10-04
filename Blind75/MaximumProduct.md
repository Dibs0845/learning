***Maximum Product Subarray***

Given an integer array nums, find a subarray that has the largest product, and return the product.

The test cases are generated so that the answer will fit in a 32-bit integer.

***Note*** that the product of an array with a single element is the value of that element.

 

​​​​​​***​Example 1:***

Input: nums = [2,3,-2,4]
Output: 6
Explanation: [2,3] has the largest product 6.

***Example 2:***

Input: nums = [-2,0,-1]
Output: 0
Explanation: The result cannot be 2, because [-2,-1] is not a subarray.

```
public int maxProduct(int[] nums) {
    if(nums.length == 0) {
        return 0;
    }
    int max = nums[0];
    int min = nums[0];
    int result = max;
    for(int i = 1; i < nums.length; i++){
        int cur = nums[i];
        int temp = Math.max(cur, Math.max(max*cur, min*cur));
        min = Math.min(cur, Math.min(min*cur, max*cur));
        max = temp;
        result = Math.max(result, max);
    }
    return result;
}

```