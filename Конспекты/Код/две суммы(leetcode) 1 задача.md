```python

#Зимин Я.В.

class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        
        first_opperand = 0
        second_opperand = 0

        for i in range(len(nums)): # Первый цикл перебирает первый элемент пары
            first_op = nums[i] # Берётся значение первого элемента
            for j in range(i+1, len(nums)): # Второй цикл начинается с следующего индекса после текущего
                second_op = nums[j] # Берётся второй элемент пары
                if first_op + second_op == target: # Проверяем сумму текущих значений
                    return [i, j] # Если сумма равна цели возвращаем оба индекса
```
![[{B214E3D9-D478-4ABE-A491-E982984486C4}.png]]